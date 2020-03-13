---
layout: post
title: "Deferred Renderer"
date: 2019-1-1 9:00:00
categories: "GameEngine"
excerpt_separator: <!--more-->
---

Deferred rendering, is a technique in which the program postpones (hence deferring) the lighting calculations after all the geometry has been rendered so that the lighting calculations happen only on the pixels that are rendered. All geometry behind the player, or culled by the view frustum will not have the lighting process, saving a lot of time.

<!--more-->

It is also much better for multiple light sources as the calculations are reduced to just what is on the screen.

When rendering the scene, there are 3 or 4 outputs, part of the G-Buffer : the positions, the normals, the albedo (the color) and possibly extra information (roughness, metalness for PBR which I will write about later on).

### Normal

![photo](/assets/normal_r.PNG)

### Position

![photo](/assets/position.PNG)

Afterwards lighting calculations happen in the lighting shader, part of the post processing pipeline.

The only problem I had with writing the deferred renderer was that because I was storing the positions of the pixels in view space, I needed more precision in its GBuffer texture. I just happened to put GL_RGBA not GL_RGBA16F. This caused a bit of problems and time but wasn't too bad. 
