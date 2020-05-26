---
layout: post
title: "An ergonomic and comfy Emacs config"
date: 2020-5-25 12:00:00
categories: "Misc"
---

I'm far from an Emacs veteran, but I love this operating system and have spent
an unhealthy amount of time configuring the thing. I *almost* live in
it. There are, however, a couple problems with the default build of
Emacs: the keybindings are painful and I find the constant movement
of the eyes quite uncomfortable.

*So*, I'm hoping, through this article to show a couple things I have done
to make Emacs a little more friendly to your eyes, fingers and especially
your pinky!

## Ease on your eyes
Let's first go through some more trivial things: the theme.

Unless you're a psychopath, you're going to want to find yourself
a nice dark theme. I personally prefer low contrast dark themes. so
I opted for the [Nord theme](https://github.com/arcticicestudio/nord-emacs).

![photo](/assets/nord.png)

> By default, the theme doesn't come with a color for the current line number
that you are on, so here are a few lines of code to fix that up:

{% highlight lisp %}
(set-face-attribute 'line-number-current-line nil :background "#3B4252")
(set-face-attribute 'line-number-current-line nil :foreground "#81A1C1")
{% endhighlight %}

The next thing, is mostly a matter of personal preference and it is that
I've noticed that my eyes naturally gravitate towards the centre of the screen,
or the top of the screen. This is a problem because by default, I often have
to look down at the minibuffer for example if I want to find a file.
Thankfully there is a package: `ivy-posframe` which solves this issue.
It basically sort of makes your minibuffer into a sort of box of which
you can chose the location. I put it in the centre of the screen like in the image
below.

![photo](/assets/posframe.png)

Notice the box that appears in the centre of the screen corresponding to the
prompt of the `find-file` command (`C-x C-f`). Any command which will summon the minibuffer
will now appear in this box at the centre which is very nice!

Here is the short setup code for it.

{% highlight lisp %}
(require 'ivy-posframe)
(setq ivy-posframe-display-functions-alist '((t . ivy-posframe-display-at-frame-center)))
(ivy-posframe-mode 1)
{% endhighlight %}

You can of course change the position to be at the top, at your cursor, or the bottom.

The next very useful package that solves not only an "eye" issue, but also a simple
annoyance is `popwin`. This package fixes the problem where for example say you
call `M-x compile` to compile your code. You will have the problem that the
`compilation` buffer just at a random location and may override another buffer
*which is super frustrating*.

What popwin does, is it basically makes it so that when one of these things happens
there is just a window that pops out from the bottom or the top, without messing up
your previous window configuration. You can very simply close it with `C-g`.

In the picture above, popwin is used to make the compilation buffer appear at the
top and not mess up any of my window layouts. Here is the setup code:

{% highlight lisp %}
(require 'popwin)
(popwin-mode 1)

(push '(compilation-mode :position top :noselect 1 :stick 1) popwin:special-display-config)
{% endhighlight %}

> Of course if you change `top` to `bottom` it makes it pop up from the bottom instead.

That's pretty much it for fixing any sort of eye strain, now for the hand strain and Emacs pinky.

## Avoiding the Emacs pinky

Now, I know this may be blasphemous for hardcore Emacs users, but I decided to learn
the vim keybindings to see if it helped with pain in the pinky and my god is the difference
huge. The constant key *chords* with the Control key sometimes caused enough strain
to justify a long break. Now, there is no need for that. I'll take you through
how I got to pretty much completely removing the need to press the Control key.

### Evil-mode
It goes without saying, that evil-mode is one of the best vim keybinding emulation
packages out there. The only thing was to remap Escape to TAB with the `evil-escape`
package. Here's the setup code:

{% highlight lisp %}
(evil-escape-mode)
(setq-default evil-escape-key-sequence "TAB")
(global-set-key (kbd "TAB") 'evil-escape)
{% endhighlight %}

> Not Caps lock because I actually somehow use the caps lock key quite a lot
To indent lines, I just highlight the line and press `M-3`.

### Replacing the Control key for everything else!
I know there is Spacemacs and Doom Emacs out there which both solve this issue
quite elegantly, however I didn't want to lose all my Emacs configurations so
opted out of switching. Instead of pressing Control all the time (for example
for finding files, or projectile commands), I now switch to normal mode
and invoke all the commands with the Space key instead because my thumb
is a bit stronger than my pinky. I also wanted all the keybindings to be
as similar as possible to the default ones (for example, going to another window
would be `SPC-o` instead of `C-x o` or finding a file in projectile would be
`SPC-p f` instead of `C-c p f`). To this I used my new favourite package:
`general` (found [here](https://github.com/noctuid/general.el)).

Here is my setup code:


{% highlight lisp %}
(use-package general :ensure t
  :config
  (general-define-key
   :states '(normal visual insert emacs)
   :prefix "SPC"
   :non-normal-prefix "C-SPC"

    ;; simple command
    "f" 'find-file
    "h" 'split-window-vertically
    "v" 'split-window-horizontally
    "1" 'delete-other-windows
    "p" 'projectile-command-map        ; SPC-p basically replaces C-c p
    "w" 'popwin:close-popup-window     ; Instead of C-g to close popwin
    "i" 'evil-window-up                ; IJKL are like WASD in games
    "j" 'evil-window-left
    "k" 'evil-window-down
    "l" 'evil-window-right
    "s" 'swiper
    "c" 'kill-buffer-and-window
    "+" 'balance-windows
    "0" 'delete-window
    "t" 'treemacs
    "b" 'ivy-switch-buffer
    "x" 'execute-extended-command
    "r" 'recenter
    "`" 'next-error
    "m" 'my-compile-current           ; Calls compile function (make compile)
    "e" 'my-run                       ; Calls make runs
    "dc" 'gud-cont
    "dn" 'gud-next
    "ds" 'gud-step
    "db" 'gud-break
    "df" 'gud-finish
    "dr" 'gud-run
    "dd" 'gud-remove
    "dl" 'gud-refresh
    "dg" 'my-start-gdb
    "o" 'other-window))
{% endhighlight %}

`gud-cont` for example has 2 keys, basically translating to `SPC-d c`.

The final challenge was completely removing the need to press `C-g`.
I replaced it with `M-q` which is still fine because I can press the `Alt` key
with my thumb. Here is the code for that:

{% highlight lisp %}
(global-set-key (kbd "M-q") nil)
(global-set-key (kbd "M-q") 'minibuffer-keyboard-quit)
{% endhighlight %}

There are a couple more things. When you are presented with the prompt of find file (with ivy-mode)
you need to press `C-n` and `C-p` to move around. I replaced these with `M-k` to go down and
`M-l` to go up.

{% highlight lisp %}
(global-set-key (kbd "M-k") 'ivy-next-line)
(global-set-key (kbd "M-l") 'ivy-previous-line)
{% endhighlight %}

The final thing is switching to other windows when you are in a mode that has the space key bound to
something already which is a problem (for example dired). For this I just added `M-;` which also
calls `other-window`.
{% highlight lisp %}
(global-set-key (kbd "M-;") 'other-window)
{% endhighlight %}

And voila! A nice comfy and ergonomic Emacs configuration! Happy coding!