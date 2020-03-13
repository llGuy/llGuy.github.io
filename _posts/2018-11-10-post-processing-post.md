---
layout: post
title: "Post Processing"
date: 2018-11-10 14:30:00
categories: "GameEngine"
excerpt_separator: <!--more-->
---

Post processing is super cool! as it's not very hard to implement and makes your game look so cool.

<!--more-->

### DoF

Depth of field is effect in which when you look at an object, the objects around that central point get blurry.

It wasn't hard to implement and I didn't even need a tutorial for it (however I don't know if this is actually how you do it).

All you do is get the depth information of your scene, render a blurred version of the scene ([Gaussian blur](https://www.youtube.com/watch?v=uZlwbWqQKpc)), and pass it through a shader where each pixel's depth is compared to the center pixel's depth. Each pixel's depth will get blurred depending on the difference between its depth and the center pixel's depth.

![photo](/assets/dof.PNG)
