---
layout: post
title: "Screen Space Relfections"
date: 2019-1-1 10:00:00
categories: "GameEngine"
---

# Screen Space Reflections

This was quite hard to implement mostly because there isn't many documentation on how to properly implement SSR. The best I found was this : http://imanolfotia.com/blog/update/2017/03/11/ScreenSpaceReflections.html

However, I honestly found it quite confusing as some things don't really line up (I even asked a question just to be sure are something specific).

The first thing I decided to do was to implement through this tutorial. However it didn't work at all. This was one of the few problems I encountered while trying to implement this.

![photo](/assets/SSR1.PNG)

Before I get to what the problems were exactly, I need to explain how this works.

# Explanation

SSR is interesting as it manages to simulate relatively realistic reflections with just what is on the screen eliminating other methods such as planar, cubemap as it is quite a bit cheaper as a full rendering of the scene isn't necessary. The way it works is using the position GBuffer, for each pixel on the screen

![photo](/assets/SSR2.png)

you ray trace in the direction of the reflected ray (GH normalized) until the ray is behind something according to the posiition GBuffer. When that happens, the ray tracing stops. To get extra precision, we can do a binary search to get the most precise cooordinate possible.

The concept isn't that complicated. However, there are many many many problems that arise if we just implement it like this. First of all,

![photo](/assets/SSR3.png)

the basic checking to see whether or not to reflect is that the point B is behind the point A. However, there is no threshold as to how far away the point B should be. If the point P, for example, is right underneath the point A, it shouldn't reflect right? however, it does because it thinks that the point B is behind A. That is why it looks like the ball is being reflected on every point that is underneath it in screen space.

Ok that doesn't seem too bad, just put a threshold. If B is further away than 1.2, don't reflect the point.

![photo](/assets/SSR6.PNG)

Wow that looks much better actually

![photo](/assets/SSR7.PNG)

But what are those artifacts all about?!

Well this was a hard problem to tackle. It seemed that enabling the threshold and disabling it both cause problems. What to do.

![photo](/assets/SSR4.png)

The problem is being caused by what is seen in the first image with the box. The ray distances are set depending on the distance of the point that is trying to reflect something and the camera. As you can see, the distance between the resulting point B and the point A that should actually be reflected depends very much on where the point P is. There are zones in which the point P is positioned in a such a way that the respective point B will be further away than 1.2 from A. That is why there are multiple "bar-like" artifact at certain areas.

What if we make the threshold bigger to 10?

![photo](/assets/SSR8.PNG)

Same problem as before : the points that aren't supposed to reflect are reflecting.

# Solution : use the dot product

![photo](/assets/SSR5.png)

As you can see, the point P is under the box (and shouldn't be reflecting), angle between the vectors hit P2 to theoretical B2 and P2 to the actual correct reflected point A2 is very big whereas the angle between the vectors P1 to B1 and P1 to A1 are much smaller. This is because the point B1 for incorrect reflections will be much further away than for correct reflections.

If we then check for the cos(angle) of P to A and P to B, we can set that as the threshold for the angle.

I wrote about this on gamedev : https://www.gamedev.net/forums/topic/700212-problem-with-screen-space-reflections-in-opengl/?tab=comments#comment-5397702 
It is likely to be better explained there if you want to understand more about this.

When adding this threshold, the result is this :

![photo](/assets/SSR_FIX.PNG)

Of course, I had to make a water effect :)

![photo](/assets/SSR_WATER.PNG)

