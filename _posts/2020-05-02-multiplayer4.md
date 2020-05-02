---
layout: post
title: "Multiplayer Architecture - Part 4: Terraforming"
date: 2020-05-01 9:20:00
categories: "Game"
---

In the previous article, we discussed the implementation-side of player movement, prediction errors, and entity interpolation. Terraforming, is a much, much more complex issue, that I spent a very long time just hacking at the computer trying to get perfectly right. I've tried this whole time to find a simpler way of synchronising terraforming between multiple clients, but am still dumbfounded at how complex it ended up being.

Firstly, here is a video of terraforming:

<iframe width="560" height="315" src="https://www.youtube.com/embed/cKCpu-NAcjY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Before we get to the multiplayer side of terraforming, let's discuss how this works.

## Quick recap of chunk system

The world is actually a voxel world. It's made up of a 3D grid of points (voxels) which hold a 1-byte unsigned value going from 0 to 255. These points are organised in chunks (16x16x16). To render the mesh that you see in the video, I use the marching cubes algorithm to generate the mesh. Here's a great video by [Sebastian Lague](https://www.youtube.com/watch?v=M3iI2l0ltbE) explaining that algorithm (obviously, implementation is also in the vkPhysics repo, for more implementation details, in `game/w_chunks.cpp`). When terraforming, a ray is traced and if there is a collision with a voxel whose value is high enough for a mesh to appear, a sphere around that voxel will get modified, causing these mounds / craters to appear.

If this was a Minecraft-like game, in which players could just modify one voxel at a time, the synchronisation between players, would be much easier, because there is only one voxel to take care of. In this game, just a terraforming sphere radius of about 2 in-game meters (what it is set at in the video), could cause around 80 voxels to get modified, in the intervals between commands getting sent to the server. This means that when it comes to error checking, up to 80 voxels need to be corrected predicted...

## Let's first do this without worrying about corrections

The first thing we need to take care of is recording the modifications the client has done in the time between sending commands to the server. When we send commands to the server, we will not only the buttons pressed, the predicted positions / directions, but also the predicted values of *just* the voxels that have been modified.

Then the server receives this packet, executes the "player actions" that were sent (just the buttons pressed), which would modify the server's local terrain state. This terrain state will get dispatched to the other clients as "terrain deltas": just the voxel changes.

The other clients, in a nutshell, will interpolate the voxels that the server said had been modified, between their original local values client-side, and the final values that the server has sent.

Now, let's take these one by one.

## Sending to the server *just* the modified voxels

Each chunk, not only contains a 3D array grid (16x16x16) of `unsigned char`, containing the current values of the voxels which translates to the current mesh that is on the screen, but also contains another 3D array grid (16x16x16) of `unsigned char`. These are all set to 255 by default (the maximum value of a voxel is 254).

When a voxel (coordinate x, y, z in local chunk coordinates - 0 to 15) gets modified, the value at index (x y z) of this other 3D array (called modification pool in the code) gets set to the modified voxel's initial value.

Because we want this initial value to be the initial value before any modifications have been made between the times the client sends commands to the server, we can check if the voxel has already been modified if the value at index (x y z) of the other 3D array is 255. If it is, the voxel at (x y z) hasn't been modified. If it hasn't been modified, we also push into an array of signed shorts (called modification_stack), the index into this other 3D array which points to the newly modified voxel.

> Note: This is the exact same process on the server-side, except, instead of being in between commands getting sent to the server, it's the game state snapshot dispatches to the clients

### Sending commands

An index to every chunk containing modifications since the previous time we sent commands to the server gets stored in an array of modified chunks.

When sending commands to the server, the client will traverse this array, and pack all the modified voxels of each chunk with its voxel coordinate, initial value and final value. The modified voxels / chunks get cleared from the array and that's it! (it really isn't)

## Server-side handling of these modifications

The server then receives not only the predicted values of each voxel individually, but also the commands (button presses). Until the next game state dispatch, the server will also be simulating the world and executing these commands, thereby also computing the game state (which is the correct game state).

Then, when it comes to dispatching the voxels to the clients, the server will simply (because the server also does the whole thing of saving the initial values of each modified voxel, etc...) send the initial and final values of each voxel to the clients for them to interpolate!!!

# ALL the problems with this

There are *so* many different issues with this system when it actually comes to implementing: they come in 2 main categories:

- Handling voxel prediction errors
- Making sure the client correctly interpolates between the voxels that the server said were modified by people

Let's take these one by one, starting by errors, because it will come hand in hand with interpolation (somehow).

## Prediction errors

One may think that to correct the voxels of the client, either we can simply send the *correct* values of the voxels that it thought to have modified, or we could simply send over the entire correct chunk. Then the client simply sets the values of what the server says were incorrect to the correct values right? Right?

> Yes, but no

The problem is explained in the following diagram:

![photo](/assets/terraforming_diagram.png)

> Each terraforming action is represented by a star. The yellow represents the timeline of the client, the red: the global time, the purple: the timeline of the server.

When the server receives the first terraforming action (white), let's say that there was some error that caused the client to incorrectly predict the final voxel values.

The server says there's an error and prepares a packet to send to the client telling it to correct these voxels.

> What's the problem?

The problem is that as you can see, by the time the client receives the packet saying that it needed to correct the voxels it sent (with the white terraforming action) which was a while ago, the client already did some further modifications! This means that even if the client simply corrected the voxels that the server said to correct, it would still have a ton of incorrect voxels!

> Solution?

I wish the solution were simpler, unfortunately, it is not.

What I decided to do, was to keep a sort of history of *all* modifications that the client sends to the server. Every time it comes for the client to send commands, it will save in a list of modifications the voxel modifications that it sent in one sort of package. The next time it sends commands, save that to another package. To each package, associate it with the tick at which the client sent the packet - this will be important in a second.

When there is a prediction error, we need to make sure that all modifications done between the time it takes to send the commands and to receive the response from the server, get *undone*.

To do this, the server will tell the client which tick contained the incorrect voxel modifications (which the server knows because the client sent it via the commands packet). Then the client, when receiving the packet, will traverse through the history of modified voxels from the most recent to that tick, and undo them one by one - by setting the voxel values to the initial values contained in the history package.

Here is a video demonstrating the *reverting* of voxels (I simulate lag by temporarily not sending commands to the server which will inevitably cause an error correction):

<iframe width="560" height="315" src="https://www.youtube.com/embed/J2-1pGsp0LY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Once reverting is done, we can proceed to setting the correct values of the voxels that the server sent us.

We must also make sure to clear the history packages list from all outdated packages (whose tick value is less than the tick of the packet that the server sent to the client).

And that is pretty much error correction done!

Here is a quick video demonstrating this:

<iframe width="560" height="315" src="https://www.youtube.com/embed/lTQNLdVSnWM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

> Everytime the player jitters, it's because the server is telling it to do a correction (in real gameplay, it would never be this bad, I am deliberately causing lag in this clip).

## Voxel interpolation

This is the final part of terraforming which is tedious to implement.

Let's suppose that client A is not doing any modifications but client B is hacking away causing havoc and doing all sorts of crazy terraforming. Client A would simply interpolate between the voxels which the server said were modified to synchronise what A sees. But, because the server simply sends all the voxels which were modified, it will also send to client B what *it* modified. Now the problem comes in the fact that: 

> How the hell will client B be able to tell *not* to interpolate between the voxels it had modified previously?!

I won't explain the whole algorithm because this article is already getting long.

What I ended up doing was combining all the most recent modifications (from the diagram, if we are trying to avoid interpolating between the voxels modified in the white star, the recent modifications would be the blue, green and red stars) into one big modification package and one by one checking if the voxel that the server sent had been modified. If it hadn't, we just don't set that voxel to be interpolated in the next few frames.

That's pretty much it, the most tedious part comes to combining the modifications, and comparing them but all the implementation is in the repo if you want to check it out (it will be in `source/game/n_core.cpp`).

Here is the final cathartic video showcasing the whole thing between 2 clients!

<iframe width="560" height="315" src="https://www.youtube.com/embed/xF2L9rf2tfo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
