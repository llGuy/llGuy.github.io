---
layout: post
title: "Shadows"
date: 2018-11-10 13:40:00
categories: "GameEngine"
---

Shadows, something that needs lots of things to work, in order to work...

### How to implement shadows

Shadows are simply put, parts of an object that does not come in contact with light because another object is blocking the light.

The way you implement it is by rendering the depth information of the scene except you render the scene from the light's view. Whatever the light "cannot see" will be dimmer because of the shadow. This new depth image is called the shadow map.

![photo](/assets/shadowmap.PNG)
(the shadow map)

### The problems that arose...

The main problem I had when working with this is `framebuffers`. I had never used framebuffers before so using it here was a huge pain.

It was very hard for me to know whether or not a framebuffer was working because it wasn't rendered to the screen.

Then, once I created a very simple GUI system to see what was rendered to the FBO, often, nothing was rendering to that GUI. It  was just black.

Thanks OpenGL.

I was therefore kind of stuck as the problem could have been ANYWHERE - there were so many things that need to go right to create the FBO : the texture creation, renderbuffer ...

Not only that, but once the shadows worked, they only worked in a certain part of the world. All objects out of this zone wouldn't cast shadows :(

I read a ton of tutorials and watched a ton of videos on this subject...

[The first video I watched](https://www.youtube.com/watch?v=o6zDfDkOFIc) was by the ThinMatrix which I really like. His tutorials are very clear and simple. I understood the concept of shadows through this tutorial, however, the implementation was a bit less straightforward to follow. He wasn't writing the code to explain the implementation in the video. He was just going over the code he had written and explaining it a bit as otherwise the video would have been too long which is understandable. Therefore, I downloaded the code he provided and tried to follow it. That really didn't go well. I couldn't really follow what he was doing... The way he implemented most of his camera code (which was essential to shadows) were not at all what I did.

So, I decided to scratch that tutorial and had a look at [this tutorial](http://www.opengl-tutorial.org/intermediate-tutorials/tutorial-16-shadow-mapping/). I finally managed to get simple shadows into my world after a bit of work!

But there was one last problem... the world in my project was big and the shadows only cast in a small zone.

The final tutorial which cleared everything up for me about this was [this video](https://www.youtube.com/watch?v=lUo7s-i9Gy4). He explained the maths behind casting shadows in a big world extremely clearly and I managed to implement proper shadows! Finally.

![photo](/assets/shadows.PNG)
