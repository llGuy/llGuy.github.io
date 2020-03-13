---
layout: post
title: "Reflective Surfaces"
date: 2018-11-10 14:00:00
categories: "GameEngine"
excerpt_separator: <!--more-->
---

Unfortunately, this post will not be too interesting. Although water looks super awesome in games, it was much easier to implement than those damn shadows (which look easier to implement).

<!--more-->

### How it works

To implement reflective surfaces, you just need to render the scene from underneath and above the surface and use [projective texturing](https://www.youtube.com/watch?v=GADTasvDOX4) to blend the two images together onto that surface.

### Reflective surfaces (and refractive surfaces) - Water

Implementing reflective surfaces (or just water) was very simple which was something I wasn't expecting when implementing it.

This was mostly due to the fact that I had already worked with shadows which used framebuffers (the new concept for me).

For this, I used [this video](https://www.youtube.com/watch?v=HusvGeEDU_U)

![photo](/assets/water.PNG)
