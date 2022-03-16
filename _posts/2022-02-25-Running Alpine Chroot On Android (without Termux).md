---
title: Running Alpine Chroot On Android (without Termux)
author: 
  name: Cypherpunk Samurai
  link: https://github.com/CypherpunkSamurai 
source: _posts/2022-02-27 Running Alpine Chroot On Android (without Termux).md
categories: [ NoteDown, Tutorials ] 
tags: [ android, no-termux, shell, chroot, proot, rootfs ]
date: 2022-02-25 19:25:04 +0000
---


This is my post on how I managed to run a alpine chroot system on android without root 🌿

## Intro
So, I was recently using Termux when a thought came to my mind 🤔.

It's fascinating how Android has grown since KitKat era. We used to have terminals that looked like xterm without styling. Jackpal Android Term used to be what came along Kali Nethunter 😌. 

The only ways to run Linux on android was chrooting (change root), on a rooted device. Linux Deploy one of very first apps that perfected it. It came Pre-installed on Kali Nethunter so you can run a Kali subsystem on android, very cool back in days 😌. Now.... Now we have Termux that works without root. You can now run Arch to Kali, heck even Gentoo without root 🤯.

> How did we come this far? 🤨

Good Question 🤔, I strongly believe this is because of the simple tool `proot`. That uses user-level resources to chroot into a rootfs. Termux project added patches that allowed them to make it more compatible for Android devices, and then Termux was born ✨.

Let Me Show You How 🪴

## Required
* We need Android Terminal Emulator
  Suggested Terminals:
  * [Terminal by Alif Software](https://play.google.com/store/apps/details?id=com.qamar.terminal) - (recommended)
  * [TermOne Plus](https://play.google.com/store/apps/details?id=com.termoneplus)
  * [Jackpal Android Term](https://play.google.com/store/apps/details?id=jackpal.androidterm)
* Alpine RootFs tarball
* Busybox

## Configs
Set this as initial command so everytime we start terminal we are at app_HOME folder 📂. 

```script
cd $HOME && clear 
```

Then we setup a small rootfs for terminal emulator.
```shell
mkdir bin
cd bin
```

We need to add this `bin` folder to `PATH`, so we can access binaries inside it.

```shell
# Assuming we are at ./bin folder
EXPORT PATH=$PATH:$(pwd)
echo $PATH
```

## Download Busybox
Downloading busybox is easy. We will use [Busybox from Magisk-Modules-Repo](https://github.com/Magisk-Modules-Repo/busybox-ndk) which are compiled from (osm0sis Android Busybox Repo)[https://github.com/osm0sis/android-busybox-ndk]. 
We will download it with curl, because wget is not installed in android. I'm using arm64 binary, use binary according to your cpu. Use `cat /dev/cpuproc` or `uname -a` to find. 
```shell
curl -o ./bin/busybox https://github.com/Magisk-Modules-Repo/busybox-ndk/raw/master/busybox-arm64
``` 
Now we should make it executable by giving it perms 
```shell
chmod 755 ./bin/busybox
# or
chmod +x ./bin/busybox
``` 
We should now be able to run `busybox tar -xvf` and it wont give us `gunzip` not found error. 

Installing applets is a nice addition, so we will install busybox applets as commands. To link applet to busybox we run: 
```shell
ln -sf ./path/to/busybox ./path/to/applet/applet_name
``` 
Linking one command: 
```shell
# Assuming we are in $HOME
cd bin
ln -sf busybox xargs
``` 
Next let's link all applets. 
> But Busybox has so many applets, How do we install them all? :( 

We use the `busybox --install -s ./directory` command. 
```shell
# Considering we are at $HOME directory
busybox  --install -s ./bin
# Check
ls ./bin
``` 

Great! Now we can access all of busybox tools 👍

## Getting Alpine rootfs
Go to [Alpine Linux Downloads](https://alpinelinux.org/downloads/) page and copy download link of mini rootfs according to arch. 
I'm downloading arm64 rootfs so i have to run: 
```shell
wget -O alpine.tar.gz https://dl-cdn.alpinelinux.org/alpine/v3.15/releases/aarch64/alpine-minirootfs-3.15.0-aarch64.tar.gz
``` 
Isn't it pretty cool that we can run wget now ✨! 
Anyway, now we use `tar -x` to extract to alpine dir. 
```shell
mkdir alpine
tar -xvf alpine.tar.gz -C alpine
```

Great! Now we have a alpine rootfs! :)

## How to Chroot without root?
> Now that we have a alpine rootfs, how do we chroot into it? 🤨

We use `proot`, like termux. 

> Where do we get proot from? We don't even have apt or pkg 😐

We download publicly available binaries.

One of the repos i found searching for 'android proot binary' was [green-green-avk/build-proot-android](https://github.com/green-green-avk/build-proot-android). It's a fork of [termux/proot](https://github.com/termux/proot) with little patches. You can also use termux/proot.

I downloaded and extracted the proot binary from the repo.

To Download proot:
```shell
curl -LO "https://github.com/green-green-avk/build-proot-android/blob/master/packages/proot-android-aarch64.tar.gz"
```
To Move binaries:
```shell
tar -xvf proot-android-aarch64.tar.gz
mv ./root/bin/* ~/bin/
mv ./root/* ~/
```

Great, proot android is installed. Try running it with `proot` command.

## I'M PROOT
Now time to proot into our alpine rootfs. To do so we need to do it in multiple steps. So i made a simple script.

```shell
#!/system/bin/sh
export PATH=$PATH:$HOME/bin

unset LS_PRELOAD

ROOTFS_PATH=$HOME/alpine

PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
proot \
-r $ROOTFS_PATH \
-p \
-H -0 -l -L \
-b /sys \
-b /dev \
-b /proc \
-b /proc/mounts:/etc/mtab 
```

Lot's of thigs going on here. Let's go line by line.

> #!/system/bin/sh

We First declare which shell to use when this script file is run.

> export PATH=$PATH:$HOME/bin

We set the path so we can access proot (just in case) `PATH` is not set.

> unset LS_PRELOAD

Unset `LD_PRELOAD` to prevent errors.

> ROOTFS_PATH=$HOME/alpine

We set the folder of the alpine rootfs. Here its in ~/alpine so $HOME/alpine.

> PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

We set the new path that tells proot where to look for binaries.

> proot \
> -r $ROOTFS_PATH \

We run `proot` with `-r` root location.

> -p \

We don't have root access, so we use higher ports for binding.

> -H -0 -l -L \

We hide files starting with `.proot` with `-H`. Give our user fake root `-0`. We make current user group appear as "uid:gid" with `-i`. We use correct size of lstats `-L`.

> -b /sys \
> -b /dev \
> -b /proc \
> -b /proc/mounts:/etc/mtab 

Here you can see something you'll be familiar as a linux user. As Common chroot practice here we bind hardware locations to new root.

and on running it we should get....

![](/assets/img/Running-Alpine-Chroot-On-Android-(without-Termux)_3.png)

# Objective Complete!
That's It for this post. I'll save cooler stuff for lated posts. Hope you learnt something new today.

Let me know if you liked / disliked the post by leaving a comment below! :)

<font face="monospace">
<i>
<b>
~ **CypherpunkSamurai**
</b> Logs_Off....
</i>
</font>
