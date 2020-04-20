---
layout: post
title: "Multiplayer Architecture - Part 3: Player movement Implementation"
date: 2020-04-18 12:40:00
categories: "Game"
---

In the previous article, we discussed the theory behind client side prediction, error corrections and interpolation. The theory may not sound so bad, but the implementation is particularly taxing and difficult. The most difficult part is handling the prediction errors and making sure that when the client corrects them, the server and clients are all in sync.

## Synchronising, without taking care of prediction errors

It may be simpler for now, to not worry about prediction errors, as the complexity just grows exponentially when that happens (although, we will get into it towards the end of the article). Let's just focus on sending the commands to the server and getting the server to dispatch the states of the other clients so that we can interpolate between the states.

### Sending input to server

We must note, that the client sends commands to the server every 20 milliseconds.

> Note: We send things at 20 or 25ms intervals to not overwhelm the server with packets.

Meanwhile, with every frame, the game will capture input and the state of the player will therefore change (with movement, looking around with mouse, etc...). Therefore, in between these 20 milliseconds, there will be multiple frames, and we must save all the input that altered the player's state so that we can send ALL of the input actions to the server with that 20ms mark comes and we actually send the input. 

This is farely trivial:

- Every time input is read, we push all the input of that frame to an array of `player actions` (a structure containing the key strokes and mouse movements, along with the `delta time` because this may change depending on framerate)
- Once it comes time to send input to server, we simple send all the player actions that were pushed, to the server and we clear the array of `player actions`

Once the server receives the input, it will push these structures once again to a list of inputs for that specific client. When it comes for the server to update the game state, it will simply crunch through these and the state of the game will have been updated.

### Interpolating between different player states of the other remote clients

When it comes for the server to dispatch the game state to all the clients, it will send the current positions / directions of each client.

When the client receives this (in function `s_process_game_state_snapshot`), it will push this player state "snapshot" in to an array. It will then over multiple frames, interpolate between 2 of these player snapshots at a time.

> Note: We make sure to have at least 3 player states so that if the client program finishes interpolating between 2 different states, there is always a third one to interpolate to. This does however mean that there is a little extra delay between what the client sees and what actually happened.

### Error correction time

Error correction, in both player movement and the terrain's modifications are definitely the most complicated parts of the engine for the moment (especially the terrain). Here we will just focus on the player error corrections for now.

#### Reminder: What exactly is an error correction?

When stuff gets sent over the network, especially with UDP packets, it is possible that packets get lost. In this case, the server would have not received the information it needed to compute the state that the client believes to be reality. Therefore, the server will compute state that is different from that of the client, and since the server is authoritative, the server must tell the client to make a correction.

#### Implementing the correction handling

The server needs a way to know whether the state between the server and client is different. In order to do this, the client will send the "predicted" state every time it sends input. The server saves the last instance of the predicted state of a client before a game state snapshot needs to happen (every 25 ms where the server sends packets to all clients syncing them up). It then compares the "predicted" state with the actual state of that the server has calculated. If they are the same, perfect! If they are not, we got a problem. What to do now?

Well, in the packet that the server sends to the clients, there is a flags: "needs to do correction". If this flag is set (which the server will set if there was a prediction error), then the client needs to override its state with the "correct" state that the server sent over. This will mean that the new "predicted" states that the client would have calculated (from pressing keys and so on) between the time that it sent the packet containing the error and time the packet saying that there was an error arrived, will be completely overriden. This will cause a slight jump in what the client sees. This is something that can often be observed in realtime fast paced shooter games, if your Wi-Fi isn't so good!

This will work, but there will be one really nasty bug that might hit you once in a while. That bug being these moments where the client just keeps on doing these corrections until you stop moving. This is probably something that is more tied with the way that I designed this engine. When the client receives this packet telling the client to do a correction, the client would have maybe already recorded some input and pushed it onto the stack of player actions, ready to be sent in the next 20 ms mark to send the input to the server. What you're going to have to do, is if you receive this correction packet, you need to clear this array of player inputs because it will affect what the server will calculate (as it will have taken into account input that was not recorded after the correction, it will go through extra input steps and mess up the state, causing it to ask the client to make corrections again, in a loop - VERY BAD). That was really the worst bug from the system, which is now fixed.

The other thing we need to do, is if the server thinks that the client made a prediction error, set a flag somewhere saying: "DO NOT INTERPRET ANY PLAYER INPUT COMMANDS FROM THIS CLIENT UNTIL THE CLIENT HAS ACKNOWLEDGED THE PREDICTION ERROR AND EXPLICITLY TOLD US THAT THEY DID A CORRECTION". The reason for this is that the client will have probably sent an input packet before it would have even received the packet from the server saying it needs a correction: it would be an invalid packet, and the server shouldn't interpret it. Once the client has acknowledged the predcition error (this is a flag that is set in the packet containing the input commands of the client), the server will unset this flag and continue to interpret and welcome all packets from this clients.

That is pretty much player movement done. The next behemoth to take on is terrain modifications, and synchronising that between clients. That will probably deserve a couple articles...
