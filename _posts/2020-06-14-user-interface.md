---
layout: post
title: "User Interface and menus"
date: 2020-06-13 9:20:00
categories: "Game"
---

On the Internet, we often find articles and videos on the Internet explaining
how to get a 3D scene set up in which the user can move around, and see how
the world interacts with objects. However, something I rarely see on the Internet,
is how to make your application start to actually look like a game with a main menu,
intermediate menus, and what it takes to structure your code in such a way that
it becomes possible and not extremely difficult. In order to do that, there are
a couple things we are going to have to discuss.

## Event systems

Event systems are extremely important in games, and software in general - for example,
mobile app development is heavily event driven. As game developers we are going to have
to allow our codebase to integrate a sort of event system, providing so many benefits,
as we will see in just a second.

The goal of an event system is to allow the various modules that exist in
the codebase of our game (or application in general) to communicate. In games for example, and
specifically in this game, we have, at the time of writing this article, a network module,
a world module, a renderer module, an input module and an "engine" module which basically links them all together.

The idea is that from anywhere in the various modules that we have, we can submit
events which can trigger functionality, affecting other modules. For example, when the
input module receives a "resize" input from the user, the input module will submit an event
to which both the renderer module and the UI module listen to in order to update the
swapchain and framebuffers (in the renderer module's case) and to update the menu 
sizes / positions (in the UI module's case).

Now, something that I asked myself when I first heard of an event system, was
"Why don't we just call a function?". For example, if the input module receives
a resize input from the user, why doesn't it just call the functions like `resize_swapchain`
and `update_ui_menus`? To be honest, it is something that I kind of have trouble explaining
because it comes to make sense when you start using it, but I'll give it a try:

The convenience of an event system comes from the fact that it allows your modules to
be less "coupled" - they can work more independently from one another. With an event
system, modules don't rely on the code of other modules because events will be submitted
to those modules and they will take care of what needs to happen.

### How it works in the code

The code for the event system can be found under `source/common/event.hpp / cpp`.

The structure which keeps track of all the events being submitted is `struct event_submissions_t`.
It has 2 important parts: the listeners and the submitted events. Each listener is associated
with an ID - a listener is usually in this codebase a module - the network module has a listener,
the world module has a listener - even the code engine module has a listener for the more important
things like quitting the game and stuff...

When you want to create a listener, you call `set_listener_callback` which returns the ID
of a listener (which you will have to use whenever subscribing to an event). You also need to
pass to `set_listener_callback` a function pointer - the callback function which handles the event.
For example, the callback function of the network module looks like this (in `source/game/n_core.cpp`).

{% highlight c++ %}
static void s_net_event_listener(
    void *object,
    event_t *event,
    event_submissions_t *events) {
    switch (event->type) {

    case ET_START_CLIENT: {
        // ...
    } break;

    case ET_START_SERVER: {
        // ...
    } break;

    case ET_REQUEST_REFRESH_SERVER_PAGE: {
        // ...
    } break;

    case ET_REQUEST_TO_JOIN_SERVER: {
        // ...
    } break;

    case ET_LEAVE_SERVER: {
        // ...
    } break;

    }
}
{% endhighlight %}

At the start of every frame, the engine will dispatch all the events in which the event system
basically loops through all the events which were submitted and executes the callback function
for all the listeners who subscribed to those events.

There are for the moment about 20 events that can be triggered:

{% highlight c++ %}
enum event_type_t {
    ET_RESIZE_SURFACE,
    ET_CLOSED_WINDOW,
    ET_REQUEST_TO_JOIN_SERVER,
    ET_ENTER_SERVER,
    ET_SPAWN,
    ET_NEW_PLAYER,
    ET_START_CLIENT,
    ET_START_SERVER,
    ET_PRESSED_ESCAPE,
    ET_LEAVE_SERVER,
    ET_PLAYER_DISCONNECTED,
    ET_STARTED_RECEIVING_INITIAL_CHUNK_DATA,
    ET_FINISHED_RECEIVING_INITIAL_CHUNK_DATA,
    ET_SET_CHUNK_HISTORY_TRACKER,
    ET_REQUEST_REFRESH_SERVER_PAGE,
    ET_RECEIVED_AVAILABLE_SERVERS,
    ET_BEGIN_FADE, // With data, can be in / out
    ET_FADE_FINISHED, // Just so that game can know when to do some sort of transition or something...

    ET_EXIT_MAIN_MENU_SCREEN,
    ET_CLEAR_MENUS_AND_ENTER_GAMEPLAY,
    ET_CLEAR_MENUS,
    // All the other modes (launch server loading screen, etc...
    ET_LAUNCH_GAME_MENU_SCREEN,
    ET_LAUNCH_MAIN_MENU_SCREEN,
    ET_LAUNCH_INGAME_MENU,

    ET_BEGIN_RENDERING_SERVER_WORLD,

    ET_INVALID_EVENT_TYPE
};
{% endhighlight %}

Just from reading the name of these events, it starts to become clear how this can be used for
a menu system.

Furthermore, for each event, there can be data associated with it. When you call `submit_event`,
you pass in a pointer to a struct which contains data that the callback function will interpret.

## The Menu System

In this article, I won't get into how the UI code I wrote works, simply because writing
UI code is *extremely* painful and because I wrote it ages ago and haven't bothered to look
and understand how it works (because of how *painful* writing UI is). Instead, I will explain how
the menu system works and how it manages to go from the main menu, to a game menu, to gameplay.

![photo](/assets/main_menu_uncollapsed.png)

This is the main menu (when you start the game).

The way the main menu appears is when the `ET_LAUNCH_MAIN_MENU_SCREEN` event gets submitted
(could submitted from the engine core, the game menu, anything). Also, another listener to the
`ET_LAUNCH_MAIN_MENU_SCREEN` event is the world module. Why? Because the world you see
behind the menu is actually rendered (to enable a sort of cool mouse movement effect).

> Don't worry, it doesn't get loaded in with the calculations that usually happen when loading
a voxel world - I wrote a little API to bake a voxel world into a file (one VBO) which gets loaded
from the hard drive and can be rendered in one draw call.

The world module therefore needs to know if it should render the world that was loaded from
the hard drive (which is not interactable) or to render the world which is being taken care
of with the entire voxel system that is in fact interactable.

Another one, is simply the engine module. Every time a menu gets launched, the engine core
module will relay the mouse and keyboard input to the UI system instead of the world system
and it will do stuff like enabling the display of the mouse, etc...

There are 4 main buttons in the main menu screen.

The bottom one exits the game (in which the ET_CLOSED_WINDOW event get triggered). The other
3 buttons cause a menu to slide open as you can see with the first one (browse server button):

![photo](/assets/main_menu_collapsed.png)

In this menu, you can choose which server to connect to and request a connection with
the `Connect` button.

When you press `Connect`, the main menu gets closed (`ET_CLEAR_MENUS`) the screen fades
out (caused by the `ET_BEGIN_FADE`). The fade itself will trigger some other events,
like loading the world when the screen is all black (so that you don't see the world gradually
getting loaded it) and it will ultimately open the `game menu` which looks similar to the
main menu.

![photo](/assets/game_menu.png)

Instead of the first button being browse servers, it instead spawns you into the game.
When you click on the bottom button, the `ET_LEAVE_SERVER` gets called (as well as another
fade effect, and the main menu gets openede). When pressing spawn, the game menu gets closed
and the input gets passed onto the world module.

In game, you can also launch that game menu.

![photo](/assets/ingame_menu.png)

All in all, the event system was super useful in getting this game to start acting and
behaving a bit more like a game with menus and stuff!
