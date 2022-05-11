---
title: Radare2 On Android (Termux)
author: 
  name: Cypherpunk Samurai
  link: https://github.com/CypherpunkSamurai 
source: _posts/2022-05-12 Radare2 On Android.md
categories: [ Tutorials ] 
tags: [ android, termux, re, radare2, reversing, setup, ghidra, decompiler, disassembly ]
date: 2022-05-12 12:52:04 +0000
---


In this tutorial we are going to install [radare2](https://rada.re) a reverse-engineering tool on android 📱  

## Intro
I was working on a few arm64 crackme with qemu [\[1\]](https://azeria-labs.com/arm-on-x86-qemu-user/) [\[2\]](https://azeria-labs.com/arm-on-x86-qemu-user/) and it struck my mind. 'My android device is arm64, If i can run it there it could be portable'.

Thats where the exploration began, and here's the story to it.

### Why Radare?
Radare is a tui based RE tool, and runs in most linux distros. It has a great community and is very famous. (It also has a fork rizin, famous for its GUI Cutter).   
Radare also supports plugins and most importantly there's a ghidra plugin (which we will be installing in this tutorial later on).  

Termux is a FOSS android proot debian based environment. It supports most android devices, and has a huge repo of tools including gcc, python, rust and... radare2.  
As a matter of fact, **(SPOILERS)** i've previously used radare2 on android, but i hadn't tried the ghidra plugin yet. This was the right time to try it.  

### Installing Radadre2
Installing `radare2` on termux is very easy. Termux repo provides us prebuilt radare2 binaries (thanks termux community) but we can also build it ourselves if we decided to.  

To install it i'm gonna update the packages first and then install radare2 using apt-get.  
```shell
apt update
apt-get install radare2 -y
```

This should install radare2. 😄  

### Trying it out
To test out radare2 i downloaded and compiled a few sample crackmes from [NoraCodes crackmes repo](https://github.com/NoraCodes/crackmes).  

```shell
git clone https://github.com/NoraCodes/crackmes.git
```

To compile them i needed gcc and make, so i installed `clang` and `make` packages.  
```shell
apt-get install clang make -y
```

Now it should compile using make  
```shell
make crackme01
```

Great, We now have a crackme to test. Let's run radare2.
```shel
r2 -AA crackme01

# or radare2 -AA crackme01
```
_**Note:** `-AA` here runs the analysis after opening radare2.
optionally we can type `aaa` in the radare console._

This should open the binary in radare and automatically start analyzing it.

Once it finishes we can list the functions using `afl`.

Then seek to the function using `s [function id here]`.

Great Radare2 Works on Android!

### Cheat Sheets
**For more commands i recommend these cheat sheets:**
* [Radare2 Cheet Sheet by williballenthin](https://gist.github.com/williballenthin/6857590dab3e2a6559d7)
* [Radare2 Gitbook](https://blog.conspirator.io/tools/untitled)
* [Radare2 Wiki](https://github.com/radareorg/radare2/blob/master/doc/intro.md)
* [Radare2 RTD](https://r2wiki.readthedocs.io/en/latest/home/misc/cheatsheet/)
* [ReHex Ninja Cheat Sheet](https://rehex.ninja/posts/radare2-rizin-cheatsheet/)
* [Radare2 Binary Adventures Playlist](https://www.youtube.com/watch?v=oW8Ey5STrPI&list=PLg_QXA4bGHpvsW-qeoi3_yhiZg8zBzNwQ)


### Taking it a JMP Further with Ghidra
We have a diassembler working on android, great, now let's get a little pseudocode magic.

For this we will be using [r2ghidra](https://github.com/radareorg/r2ghidra), a radare2 plugin that uses native ghidra for decompiling to `C` pseudocode code.

On checking the releases, r2ghidra doesn't provide any artifacts for arm64, so we will have to build it manually. This would require approximately 900 Mb of storage in your device (`clang` 300 Mb. build requires 300 Mb. `r2ghidra plugin` 300 Mb)

First lets download a release package. I choose the release `Source_Code.tar.gz` as it should preserve the `.git` folder ([we might require submodules](https://github.com/radareorg/r2ghidra/tree/master/third-party)).

```shell
wget https://github.com/radareorg/r2ghidra/archive/refs/tags/v5.6.8.tar.gz -o r2ghidra.tar.gz
tar -xvf r2ghidra.tar.gz
cd r2ghidra
```

Now that we have the sources, we need to follow the build instructions. The instructions say we must run these commands in order.
```shel
./preconfigure   # optional, but useful for offline-packagers, as its downloads the external repos
./configure
make
make install
```

This should start building and installing r2ghidra plugin.

_**Note:** Keep in mind there is a submodule inside the third-party folder. In case we get errors `pugixml.cpp` or `pugixml.hpp` not found, we will need to download / clone it from [pugixml](https://github.com/zeux/pugixml)._

### Using `r2ghidra`
To use r2ghidra we just gotta open a binary, and seek(`s [address/function]`) to the target function / sub. Then use `pdg` command.

```shel
> pd? # gives help output
> pdg # use ghidra
```

## Thanks to...
- Wonderfull Radare2 Team and Contributors.  
- Pankake for this wonderfull tool and r2ghidra plugin.  

- You, for reading this post 😉  

# Outro!
That's It for this post. I'll save cooler stuff for lated posts. Hope you learnt something new today.

Let me know if you liked / disliked the post by leaving a comment below! :)

<font face="monospace">
<i>
<b>
~ **CypherpunkSamurai**
</b> Logs_Off....
</i>
</font>
