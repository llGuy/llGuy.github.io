---
layout: post
title: "Tribute to game development: what making video games has taught me about programming"
date: 2020-07-07 12:00:00
categories: "Game"
---

I must admit, I really love game development. It always comes to me as a shock whenever I think that the making of something that was so often frowned upon by the adults that surrounded me as a kid, is something that I truly feel indebted to for its educational value especially considering that I practically didn't know anything about programming before starting.

![photo](/assets/mario.jpeg)

As someone who is very interested in computer programming, I cannot believe how many subfields I've had to explore with the ultimate goal of making the [game](https://github.com/llGuy/vkPhysics) I'm currently working on.

I therefore just wanted to pay tribute to traditional game development by sharing the various things I explored and learned to various degrees of depth as well as explain in depth how they were useful to me along the way. Hopefully, for people who feel overwhelmed by the vast amount of things that exist under the "computer programming" or "computer science" umbrella, I might convince some of you to try out game development for yourself to learn a wide variety of things in a naturally structured way.

So, following the path I took when it comes to the topics I came across in my journey in game development, let's start with the basics.

## C/C++, algorithms, optimisation, design

The development of games is extremely performance-oriented. Games have been, over the years, visually pushing more and more towards animation quality. To put it into perspective, here is what video games are pushing towards alongside what animation is pushing towards:

![photo](/assets/ue5.jpeg)

> Unreal Engine 5 demo shot

![photo](/assets/toy-story-4.jpeg)

> Toy Story 4 shot

And rendering a shot from a Pixar film can take up to a couple tens of hours. Rendering that shot of UE5, needs to take less than a sixtieth of a second... Performance is a big deal.

This ultimately means that access to low-level infrastructure is crucial when it comes to performant game code - hence, C/C++ programming. Because of this, low-level programming will become very familiar, and anyone spending a decent amount of time doing it, will eventually gain a very solid and practical understanding of how lots of things work in the computer - whether it be memory, the CPU (architecture, SSE, etc..), or even the operating system. This has ultimately come in handy when for example I had to think about how to organise entities, managing memory etc...

![photo](/assets/cpp.png)

Furthermore, when dealing with a large amount of objects (entities, or anything else that needs updating each frame), exploring topics like algorithms and various optimisation techniques will be very important as in-game population grows. Getting familiar with not only big O notation, but also memory layout becomes essential for optimising code. Cache misses or poorly designed traversal algorithms can become performance bottlenecks very quickly with a large amount of objects.

Another thing that is worth mentioning in this category, is code architecture. How do you architect your code? The way you decide to architect your game code (or any code for that matter) can greatly impact not only performance but maintainability. I came across various people on Youtube talking about how object oriented programming, or procedural programming, or whatever other type of programming is the best and pondered on which was best suited for my specific problems. It quickly became very useful to know when to use which.

However, with just a programming language and the knowledge of various programming paradigms or algorithms not much can be done and here we get into the meat of game programming - graphics.

# Graphics programming (2D and 3D)

![photo](/assets/teapot.jpeg)

This is where performance is usually the most crucial. Rendering is almost always what takes the most time each frame so optimising rendering code could prevent many performance bottlenecks.

Honestly, the amount of things you will learn through graphics programming is really outstanding. The first time I ever came across a "matrix" or a "vector" was through the beginner OpenGL 3D tutorials. But it doesn't just stop there, game math can be so complex - even involving these things called quaternions which are like black magic 4 dimensional imaginary numbers that are indispensable in 3D skeletal animation. But the great thing is, that instead of rote learning like at school where teachers give you exercises to repeat and train your ability to compute vector operations by hand, in graphics programming, you get to use these concepts and actually do extremely cool stuff with it - move characters, make them jump, shoot bullets, weird rotational wizardry, etc... and you remember how to use these concepts.

When it comes to the more computational side of graphics, learning about GPU architecture and various APIs becomes necessary. With the push for more low-level, low CPU overhead graphics APIs like DirectX 12 or Vulkan will teach anyone who spends time learning it not only how the GPU actually works, but makes them think about every graphics-related operation to make sure maximum performance is achieved.

And of course, it would be silly not to mention all the graphics techniques that exploring this field will make you encounter - deferred rendering, ambient occlusion, screen space reflections... (all of which I at some point had to implement).

Something I didn't expect to delve into in such depth was physics. Getting your scenes to look relatively realistic will require diving into of how light works, and possibly even things like how the atmosphere works (if realistic atmospheric scattering is something that your game needs).

![photo](/assets/atmospheric.jpeg)

> I finally know why the atmosphere is blue!!

But with the rise of FPS shooters and online games, trying out multiplayer is definitely worth it - although, it is a whole other world.

## Networking

Networking, and specifically synchronising players in a real-time multiplayer game is surprisingly complex. It will introduce you to a whole other type of programming. I'm serious. When I told myself, that I wanted to expand my game to be multiplayer, I thought that it wouldn't be very difficult and I could easily port my code to become multiplayer. I was very wrong - it's a totally different way of thinking.

When writing net code, it is definitely worth going low-level to get that extra performance boost and simply have more control over what happens - ultimately, it will make anyone learning it explore the sockets API, how protocols over the Internet work, OSI model, etc...

![photo](/assets/osi.png)

But most importantly, it will be the perfect excuse to think outside of the box when trying to make things work because often times, net code is so specific to the [problem](https://llguy.github.io/game/2020/05/01/multiplayer4.html) you're trying to tackle, that answers on the Internet won't come easily.

 It is great to be able to play with your friends, but at some point, it will be quite useful to get some sort of way of keeping track of the players' usernames, accounts, or the game servers to which the players will connect to - which leads on to my next point.
 
## Databases
 
This was something that I recently started exploring and using for my current project. I was always interested in lightly getting into web development, but was always quite overwhelmed by the sheer amount of things to learn - your front end HTML, CSS, JS/TS and the endless amount of "frameworks" that I always hear about, and your backend web servers, Ruby / PHP / whatever other thing people are using, MySQL or NoSQL or (once again) whatever other thing people are using. However, having the goal of creating a database of users on a webserver, has certainly helped me overcome some of that fear.

Not coming from a high-level sort of webdev background, I was pleased to see that by doing simple HTTP requests to a web server to execute [PHP scripts](https://github.com/llGuy/vkPhysics-meta) really de-obfuscated what I understood to be a web server and has actually helped me understand (at a basic level) how the browsers interface with the Apache or any other web server.

## Bonus: AI and ML

When it comes to game development, something that sparks curiosity in many people is getting AI into the game and being able to play with computer-controlled entities. Guess what - game development gives you a great excuse to try out machine learning! Not only does it give you an excuse to try out machine learning for fun, but if you wrote your game from scratch in C++ or C, you can even try out implementing your own neural networks and get a deeper understanding of how ML pipelines usually work. Furthermore, it won't feel like you're randomly making the computer decipher 28x28 images of handwritten numbers - you'll be able to integrate the network in the game to do something very cool! Here is (in my opinion) the best explanation of how ML works: https://adventuresinmachinelearning.com/neural-networks-tutorial/.

## Conclusion

Overall, I really think that for people starting out in computer programming, game programming can be an enormous benefit. It gives you warranted excuses to try out so so so many things in computer science / programming. I remember that as I started, I would just do small random projects to learn very specific topics but never had something to lead me through all of it - until I ran into game development.

And of course, isn't it everyone's childhood dream to make video games?

![photo](/assets/sunset.jpeg)
