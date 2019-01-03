---
layout: post
title: "Parallax and Normal Mapping"
date: 2019-1-1 8:00:00
categories: "GameEngine"
---

These two effects are not too complicated and can really make the 3D scene look much much better.
For a simple 2D texture, it can turn something like this :

![photo](/assets/normal.PNG)

To :

![photo](/assets/parallax.PNG)

The effect simply consists of displacing the texture coordinates of the rendered texture depending on the angle at which the camera views the surface.

The best explanation I found from here : https://learnopengl.com/Advanced-Lighting/Parallax-Mapping
