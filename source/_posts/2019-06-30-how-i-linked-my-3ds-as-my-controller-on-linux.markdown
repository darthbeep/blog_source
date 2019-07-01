---
layout: post
title: "How I Linked My 3DS As My Controller On Linux"
date: 2019-06-30 19:45:46 -0400
comments: true
categories: [3DS, linux, bash, gaming]
---
Early one morning my friend Jake texted me about playing Stardew Valley that night. I immidietly agreed, but was soon faced with a problem: my desk was currently full, leaving me unable to use my mouse. I don't usually work at my desk or use my mouse, as I favor the trackpad, but Stardew Valley is very hard to play without one. So I looked at my options. Finding a place to use a mouse was sub optimal, as was using the trackpad, but there was one thing I could do: using a controller. Which led to one more problem: the only controller I actually had was a 3DS. Well, not to worry, right? I'd used the 3DS as a console to play Smash before, and I'd heard rumors about the 3DS as a controller on Linux. So, I imminently got to work, setting up my 3DS as a controller for my computer.

A few software notes: My [3DS is homebrewed](https://3ds.hacks.guide/). I don't know if this is a requirement. But I do know it makes owning a 3DS a lot more fun. I originally did all of this on my computer, which runs Arch, but when writing this shortly after the fact, I tested it on Debian, both to ensure that I wasn't forgetting anything and to make sure this guide wasn't Arch only.

After a bit of googling, I decided on [this repository](https://github.com/phijor/ctroller), called ctroller. There was one other, but this had the prettiest readme.  One thing to keep in mind about the repository is that it has two parts. One for the Linux side, which has a server that interprets data sent from the 3DS into a device on the system, and the 3DS side, which builds a 3DS application the 3DS uses to send the data. Well, now that I had a repo, it was time to build all the things.

First, on to the Linux side of things. The repository tells you to install the AUR package `ctroller-git`, which couldn't be installed, or `ctroller-bin`, which could. Then I just typed in make install and it worked perfectly. That was easy. Going off of this it should be smooth sailing for the rest of the setup, right?

The next step is to get the 3DS files on your 3DS. You can either download them or make them yourself. Personally, I only use locally-sourced, grass-fed, vaccine-free binaries. So obviously I was going to build them myself. So I followed the instructions and went into the 3DS folder.

{% codeblock lang:bash %}
$ make release
Makefile:6: *** "Please set DEVKITARM in your environment. export DEVKITARM=<path to>devkitARM".  Stop.
{% endcodeblock %}

Alright, let's go up to the part in the repository where it tells us about DevkitARM. Wait-- why doesn't this link have any file?

So a bit of googling got me [here](https://devkitpro.org/wiki/devkitPro_pacman). And because entering random commands is what I do to keep life exciting, I entered the following into my computer (the page contains information for multiple operating systems, but I followed the instructions for Arch). For a more in depth explanation of what happened, follow the link. The next 3 code blocks are for Arch, but I'll get to other distributions in a second.

{% codeblock lang:bash Arch instructions %}
$ DEVKITPRO=/opt/devkitpro
$ DEVKITARM=/opt/devkitpro/devkitARM
$ DEVKITPPC=/opt/devkitpro/devkitPPC
$ sudo pacman-key --recv F7FD5492264BB9D0
$ sudo pacman-key --lsign F7FD5492264BB9D0sudo pacman -U https://downloads.devkitpro.org/devkitpro-$ keyring-r1.787e015-2-any.pkg.tar.xz
{% endcodeblock %}

Then I added these lines to /etc/pacman.conf

{% codeblock lang:bash /etc/pacman.conf %}
[dkp-libs]
Server = https://downloads.devkitpro.org/packages
[dkp-linux]
Server = https://downloads.devkitpro.org/packages/linux
{% endcodeblock %}

Followed by the commands

{% codeblock lang:bash Arch instructions %}
$ sudo pacman -Syu
$ sudo pacman -S 3ds-dev
{% endcodeblock %}

Because this blog was written slightly after the fact, and to ensure accuracy was tested again on a Debian VM. Distributions based on neither Debian nor Arch should probably follow the instructions on linked page, but will look a lot like this. The first part downloads a .deb binary I found [here](https://github.com/devkitPro/pacman/releases/tag/devkitpro-pacman-1.0.1), but you can find instructions for other distributions [on the main page](https://devkitpro.org/wiki/devkitPro_pacman).

{% codeblock lang:bash Debian instructions %}
$ wget https://github.com/devkitPro/pacman/releases/download/devkitpro-pacman-1.0.1/devkitpro-pacman.deb
$ sudo dpkg -i devkitpro-pacman.deb
$ sudo dkp-pacman -Syu
$ sudo dkp-pacman -S 3ds-dev
{% endcodeblock %}

No matter which path you choose, you'll need to enter the following afterwards.

{% codeblock lang:bash Any distribution %}
$ export DEVKITPRO=/opt/devkitpro
$ export DEVKITARM=/opt/devkitpro/devkitARM
{% endcodeblock %}

Alright, we got one thing working. Now let's run make release again. And... seems like we're missing bannertool. Fortunately, this is one problem the AUR can fix. Less fortunately, it seems like for any other operating system, we're dealing with [a repository](https://github.com/Steveice10/bannertool) with a vague makefile and no readme. Usually a makefile gives some clue as to how to run it, and this makefile certainly gave a clue: it got its pattern from a file in a subfolder.

The only problem? That subfolder was the submodule for [a different Github repository](https://github.com/Steveice10/bannertool). The submodule didn't copy over when cloning, but fortunately you can able to manually fix it by deleting the buildtools folder and cloning the linked buildtools repo instead. Now I just had to type make, and at least I got an executable out of it, even if it was hidden in the `output/linux-x86_64/` folder. Here's the executed code, for reference.

{% codeblock lang:bash %}
$ git clone https://github.com/Steveice10/bannertool
$ cd bannertool/
$ rm -rf buildtools/
$ git clone https://github.com/Steveice10/buildtools
$ make
$ ls -F output/linux-x86_64/ # -F will give executables a *, among other things
bannertool*
{% endcodeblock %}

So we've got an executable. What to do with it? Well, nothing yet. My gift of foresight tells me that the executable will have a twin before we're done. And where do we find this twin? Well, the original repository says to have both `bannertool` and `makerom` in your `$PATH`. And we already have an executable for bannertool, so now we need one for makerom. Following the link for the repository, we end up [here](https://github.com/profi200/Project_CTR/tree/master/makerom).

Anyone who clicks links before reading ahead will notice two things: first, that we aren't even in the head of a repository. We're in a subfolder of a repository with an entirely different goal. And second, the readme only links to a site which definitely does not contain installation information. We're back to running unidentified makefiles. At least this one gives less of a fight, all you have to do is clone the parent repository, `cd` into the `makerom` folder, and type `make`. And thus an executable is born. Reference code below.

{% codeblock lang:bash %}
$ git clone https://github.com/profi200/Project_CTR
$ cd makerom/
$ make
$ ls -F makerom # -F will give executables a *, among other things
makerom*
{% endcodeblock %}

So now we have the two executables we need. Now we just need to put them in our $PATH. There are probably a lot of ways to do this, but keep in mind when I had to get this done, I was doing it after a fair amount of figuring things out. I was tired. So this is what I did.

{% codeblock lang:bash %}
$ mkdir path
$ cp bannertool/output/linux-x86_64/bannertool path
$ cp Project_CTR/makerom/makerom path
$ cd path/
$ export PATH="$PATH:$(pwd)"
{% endcodeblock %}

Yeah.

For those of you who can't or don't want to read what I just did, I essentially made a new folder, added the two gained executables to it, and then added it to my `$PATH` variable. This is probably not advised, but in my defense, it worked.

Now, if we run make release again in the 3DS folder, it works. The 3DS files are generated. We can rest. Almost.

Now, all you have to do is plug in your SD card into your computer. There is an upload script mentioned by the repository, but I didn't test it. Once the SD is in the computer, you need to create a directory called ctroller in your sd card's root directory, and then you should put ctroller.cfg in it. ctroller.cfg should contain your computer's IP address instead of the default. You should also put `ctroller.cia` on the SD (or install the actual program however you usually transfer programs onto your 3DS).

Now, the hard part is over. Before you run the server on Linux, you need to run `$ modprobe uinput`, and  to run it you need to run the actual executable `ctroller` file, probably with `$ ./linux/ctroller`. Now that you have the server running on your computer, convert the cia to a program or install it like any other program and run it. Then make sure your computer and 3DS are on the same network. You're done! You can now use the 3DS as a controller for your computer!

To review the controller, it normally works just fine. However, it does go through periods of lag, where commands pause for a little while and then execute all at once. I would not recommended using the controller for a game with any stakes, but it works fine for Stardew Valley. Except when your internet goes down for a minute (hi Spectrum). But it's worth it to say you're gaming with a 3DS as a controller.
