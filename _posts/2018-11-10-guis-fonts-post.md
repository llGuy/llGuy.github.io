---
layout: post
title: "Guis and Fonts"
date: 2018-11-10 14:20:00
categories: "GameEngine"
---

### GUIs

GUIs were already not a very simple thing to implement. A lot of tutorials that I looked at for GUIs said that implementing them was relatively easy - "it's just rendering a 2-D textured quad" ... NO IT'S NOT JUST THAT

Let's assume that you are implementing GUIs in the normal naive way in which you just pass the 2D (OpenGL) coordinates of your GUI's quad. Ok, what happens if you resize your window? Your quad is completely messed up.

Alright, alright, let's do it another way. Let's use pixel coordinates!

Ok so you are going to specify the coordinates of your GUI in pixels and you want it to become centered in the screen. What happens if you resize? The size of you GUI doesn't change, your GUI isn't centered blah blah...

So how do you do it? Here's [an article](https://www.gamedev.net/articles/programming/general-and-gameplay-programming/creating-a-very-simple-gui-system-for-small-games-part-i-r3652/) on gamedev explaining how to do it. What's going to happen is that you are going to use a mix of the first solution and the second...

### How about fonts?

Fonts are also not the simplest thing to implement unfortunately despite being EVERYWHERE in games nowadays.

For this I used [this video](https://www.youtube.com/watch?v=mnIQEQoHHCU) by the ThinMatrix as it's probably the best one.

Implementing fonts consists of creating some kind of texture atlas for all your characters, parsing an extra font file with all the information of each character of that font in the atlas, and rendering textured quads for each character. Each UV of each vertex of the quads needs to be mapped to the correct coordinates of the texture atlas. It's kind of a pain to do but once you do it, it adds a whole new dimension of debugging :)

![photo](/assets/fonts.PNG)
