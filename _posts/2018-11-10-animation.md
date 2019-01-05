---
layout: post
title: "Skeletal Animation"
date: 2018-11-10 14:40:00
categories: "GameEngine"
---

Ok, here we go. Animations.

For implementing animations, I used once again, [ThinMatrix's tutorial](https://www.youtube.com/watch?v=f3Cr8Yx3GGA).

### COLLADA Parsing

The first thing that I had to do is have a way to load the model of the animation into the game. For this, I used the .dae(COLLADA) file format.

To parse it, I used the rapidxml library on github.

Loading the model was the easiest part of the Whole process.

![photo](/assets/model.PNG)

So far so good...

Next up, loading in the animation.

This part wasn't too complicated either. The only new OpenGL concept was using `glVertexAttribIPointer` for ints instead of `glVertexAttribPointer`.

Again, similar to the animation, a ton of things need to work, for this to work.

After having written hundreds of lines of code for loading in the animation and playing it, I decided to test out my luck.

![photo](/assets/fail1.PNG)

Well that did not work so well did it.

After a day of debugging, I found a problem in my code!!! Ah I was so happy. I forgot to transpose the matrices provided by the COLLADA file!!! That must explain the weird bird-like shape of the model right? I rushed quickly to test it out.

![photo](/assets/fail2.PNG)

#### AHHHHHHH

I was fuming. I had no idea what the problem WAS. AT ALL. I was so lost.

The following problem cost me 3 days of my life(obviously I didn't spend all day those days debugging, only like 1 or  hours2).

#### What was the problem???

I had a struct for representing a vertex attribute

{% highlight c++ %}
struct attribute
{
	/* attribute number */
	u32 number;	
	/* type of the attribute */		
	u32 t;
	/* size of the attribute */
	GLenum s;
	/* normalized */
	GLenum n;
	/* stride */
	u32 st;
	/* start pointer */
	void * p;
	/* enable divisor or not */
	std::optional<i32> divisor;
};
{% endhighlight %}

When I passed this struct into my `vertex_layout` struct, and into the `glVertexAttribIPointer()`, I wrote this

{% highlight c++ %}
glVertexAttribIPointer(attrib.number, attrib.t, attrib.s, attrib.s, attrib.p);
{% endhighlight %}

See the problem? There is `attrib.s` instead of `attrib.st` for the stride parameter.

Oh my god when I found this I had very mixed feelings. Firstly, the first time I use terrible variable names (n, t, s ...), I get cursed with this stupid error.

Secondly, I was upset that I even used these terrible names.

Thirdly, AHHH I was so happy! Finally I found a problem !! Let's try it out!

![photo](/assets/fail3.PNG)

Ugh another problem!

Honestly, the way I found the next problem was amazing. I was in the car when I decided to have a few looks back and forth from my code and ThinMatrix's code. And suddenly I had a epiphany!!!

I had a vec3 and I needed the values to add up to 1. But I was normalizing the vector. AHHH how could I have been so dumb.

Ok I go back to my computer and fix this problem!

![photo](/assets/fail4.PNG)

Yay it's taking shape finally!

#### But what the hell is with his arms??

By this time, school started so I had a lot less time to think about it. Two days after I had this problem, I was really angry. I had no idea what was wrong.

#### However

After debugging for like an hour, I found the problem. The final problem. The ultimate problem. The problem that has been staring in my face for the whole time.

I was applying a rotation adjustment to every joint (because the Z-axis in blender is up) whereas I had to just put one. In this setup, each joint would have the rotation applied multiple times which is why the arms look like this.

OK Finally I fix the problem and

![photo](/assets/success.PNG)

### Ahhhh finally.
