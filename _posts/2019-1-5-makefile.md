---
layout: post
title: "The Windows-Emacs-Makefile Conundrum"
date: 2019-1-5 17:00:00
categories: "Misc"
---

# The Situation
Recently, I've been trying to make Emacs my main "IDE" for C++, OpenGL etc... because I'm faster on it, and also Visual Studio has been really really really buggy lately (crashing, not updating, bugging...). For a while now, I've been having as a mini side project trying to make Emacs a decent C++ and OpenGL editor. However, it worked terribly.

First of all, I told myself, "I need command line compiler!" so I installed MinGW and had access to GCC. I was so happy because I could finally compile C++ code! Next step : compiling it with OpenGL. That was very very complicated. Finding the ways to include the .lib files (which aren't compatible with GCC, you need .a files) was a tough task. There are few tutorials on Youtube and even though if I do manage to get the .a file compiled with CMake, there would probably be issues with like OpenGL, GLFW running in windows blah blah. Also CMake is annoying.

So I needed another solution. And that is when I thought : "What if I could use Visual Studio's compiler?".

So I researched everything there is to know about the MSVC compiler and it's called cl.exe. Ok. Where is it.

If you search up where to find cl.exe on the web, the stack overflow results are often old (2011 or something) so they didn't work with me. So, I tried to find it and managed to do it after a Quick Search. However, I needed this to be in the path. I read somewhere that simply putting it in the path variable wouldn't work as it depends on other stuff and there is a batch file : vscall.bat or something that you can call to set up all the necessary path variables. So I found it and ran it.

Yay I have access to cl.exe and it works!

Now. Compiling C++ works but how to I link with OpenGL. Simple! Add the .lib files and the include paths. That worked! I can compile OpenGL and GLFW with C++ on the command line which means that I can use Emacs !!!

But, if I'm going to use Emacs, I need to make the compilation process not stupid, as in, I shouldn't be recompiling all the source files over and over again (my build.bat looked like this : `cl *.cpp [libs...]`). Thankfully, I heard of something called a Makefile which handles this for me! So I read about Makefiles and use it  to compile with cl.exe.

This was a very painful experience.

First of all, in my batch file, I had windows still slashes for directories. I copy pasted that into the Makefile and hoped that it worked and that didn't seem to work. After I realised that it was causing problems, I replaced it with these '/' and compiled again. However, I kept getting link errors like `__imp___glfwblahblah unresolved external symbol`. I named my lib variable in the Makefile `LIB`. When I changed that to `LIBS` it worked. Why ? Idk. Really. But it works so whatever. But I got other unresolved externals : with the signature type `__im___blah`. Apparently, I need to link with msvcrt.lib and msvcmrt.lib (if anyone is trying to do this make sure you add these!).

It was compiling fine however when I ran the damn thing I got `glew.dll missing. Reinstalling might fix the problem`. Unfortunately I couldn't reinstall the program as I was the one compiling it so Windows's trusty solution was out of the box. I'm still not exactly sure what the problem was but I thought that I had the right lib files (with their directories and all) in the `LIBS` varialbe. However when I re-copy pasted what was in the original .bat file which I used, it worked. Idk why, but whatever. It works.

So yeah I can finally compile OpenGL and GLFW with C++ on Windows which means I can use Emacs AND I didn't need to recompile everything over and over again!

However, there is a new problem. What if I change something in the .h files. It might not update the .cpp file would it huh.

Well, GCC has an option `-MM` which outputs the dependencies of the .cpp file. So, it I just `>` it into a .dep file, I would be able to include that in the Makefile! Ok I do that. But wait. GCC outputs something like this `main.o: main.cpp foo.h bar.h....`. I'm using cl.exe. .o doesn't work with it! AH ok. There is a linux command sed that can manipulate a stream of a string with a script. If I `|` the output of `GCC -MM` into  ` [some string] 's/.o:/.obj' ` and `>` that into some .dep file, I finally get the correct `main.obj: main.cpp foo.h bar.h...`.

Does all this work ? Yes!

Here is the Makefile. Hopefully this was useful to anyone...

{% highlight makefile %}
CC := cl
CFLAGS := /std:c++latest -Zi /EHsc
LIBS := [libs] msvcrt.lib msvcmrt.lib # <- these are important !
DEF := /DGLM_ENABLE_EXPERIMENTAL /DSTB_IMAGE_IMPLEMENTATION /DGLEW_STATIC
INCS := [incs]
GCCINC := [incs with -I not /I]

SRCDIR := .
OBJDIR := .
BINDIR := .
CPPS := $(wildcard *.cpp)
OBJS := $(CPPS:.cpp=.obj)
DEPFILES := $(CPPS:.cpp=.dep)

all: output.exe

output.exe: $(OBJS)
	$(CC) $(CFLAGS) /Fe$@ $(DEF) $(INCS) $(OBJS) $(LIBS)

%.obj: %.cpp
	$(CC) -Zi /EHsc /c /Fo./ $(DEF) $(INCS) $< /std:c++latest

%.dep: %.cpp
	gcc -MM $(GCCINC) $< | sed 's/.o:/.obj:/' > $@

clean:
	rm *.exe *.obj *.ilk *.pdb *.dep

-include $(DEPFILES)
{% endhighlight %}