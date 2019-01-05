---
layout: post
title: "Component System"
date: 2018-11-10 10:40:00
categories: "GameEngine"
---

Component Systems... I had some trouble designing the initial version of this component system...

### What is a component system based architecture

A component system based architecture is an alternative to your traditional hierarchal entity branches.

The `entity` class is just a simple structure that has only an id and some data common to all types of entities :
{% highlight c++ %}
class entity
{
	u32 id;
	vec3 position, direction, size;
};
{% endhighlight %}
The way you create different types of entities is by attaching components to them. Each component does something concerning the entity that it is pointing to. The basic idea is that all the components work in parallel so that if you remove a component, all the other components should still work and your game should keep running.

For example, you can add to a Player Entity, an input component(for mouse and key control), an animation component to animate the player, a network component...

[Here's an article about entity component systems](http://www.randygaul.net/2013/05/20/component-based-engine-design/)


So, the initial concept of components and component systems is relatively simple : components act upon some data stored in an entity in a decoupled fashion.

However, a few problems arose...

### How do you remove an entity?

Why is this a problem. The way I organized my component systems was in such way that the entities accessed their components through indices. If I simply stored all my components in a vector, then removing a component with the
{% highlight c++ %}
std::vector<>::erase(std::remove())
{% endhighlight %}
idiom, would shift the indices of the components creating a huge problem...

The way I addressed this problem was by creating a new data structure in which, when a component is removed, the index of that component gets pushed onto a stack of removed components, and when the user wants to add a component, this structure pops the stack and initializes the new component at that index. If there isn't anything in the "remove stack", then the structure just pushes the new item as a vector would usually.

### How do you optimize the access pattern of the components without storing all of them in fragmented sections of the heap?

I addressed this problem using templates instead of polymorphism. Each "system" of components stores a vector of
{% highlight c++ %}
template <typename T/* type of component */> struct component;
{% endhighlight %}

### How do components interact?

I didn't use a fancy messaging system for this one, I just allow the components to acces eachother using indices.

And voila, a simple component system for entities.




After having used component systems for entities for a while, I decided to expand my component system architecture to models.

### Why??

Because I often found myself in a situation in which using standard polymorphism and inheritance caused problems when the models started to get complicated.

For example, some models need a UVs buffer, normals buffer, some use an index buffer (glDrawElements) whereas others use glDrawArrays...

It makes stuff a lot easier especially when loading models. If in a .obj file, there are UVs, vertices, indices and normals, the model loader can just add all these components to the "model" instance. 

I therefore had to make one little adjustment to the current component system. I just had to add an extra template parameter to the component systems and add an "object" class.

{% highlight c++ %}
template <typename Data> class object
{	 
	Data object_data;
	std::unordered_map<component_type, component_index> components;
};

template <typename Data> class component_system;
{% endhighlight %}

The entity then became `object < entity_data > ` and the model became `model < model_data >`.

![photo](/assets/reflection.png)
Photo of a model that uses this component system.
