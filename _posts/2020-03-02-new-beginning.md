---
layout: post
title: "Start of big journey!"
date: 2020-03-02 14:40:00
categories: "Game"
---

Hello !! I've got something quite exciting to share... Over the past few months, I've been playing around with a ton of different things when it comes to game development, and thought I might start a project that may help others to learn and overcome certain challenges I had faced during this time.

The main problem I had, was finding resources on how making the networking part of a multiplayer real time game would work. So, I was starting to have this idea of writing articles on how designing an engine for this sort of thing might work.

I will also be documenting how the rest of the engine development goes as well, including the graphics engine, audio, ui, etc...

Something to note, is that this is not going to be a generic game engine that can be used to make any game, like Unity. This is going to be just a standalone game, for which I will program the underlying engine.

I won't get into detail as to what the game will be about for now, but the main thing that is to note is that it will be a realtime multiplayer first person strategy game.

# Following along

I will try to post one of these everytime I make quite a noticeable advancement in development. The code is completely open source and free to use / modify / contribute to. 

The game currently doesn't have a name, but is under the repository vkPhysics ([link to Github](https://github.com/llGuy/vkPhysics)).

At the moment of creating this project, the engine currently contains a couple modules (C++). Each module will have a header file which will contain all publicly accessible methods / structs that other modules may use.

For example, the renderer module has a public `renderer.hpp` file which contains all functions that other modules can have access to. 

For each module, there may be numerous underlying source files, each with a prefix (first letter of .hpp file) and underscore. 

For example, in the renderer module, there is the lighting submodule (`r_lighting.cpp`), the mesh submodule (`r_mesh.cpp`), etc...

# Current state of the project

For now, given that I just started, there isn't very much. Just the classic sphere PBR demo. 

I'm currently working on the renderer (because it is quite essential to any game really), which as an underlying graphics API, uses Vulkan. I will move on to more core engine stuff (networking, entities, ...) once sufficient renderer components will be added (lighting, atmosphere, ...), which I will be documenting with articles as well.

![photo](/assets/vkPhysics_pbr.png)
