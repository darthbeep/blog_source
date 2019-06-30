---
layout: post
title: "How Do Linux Gestures Work? Part I: Before You Install"
date: 2019-06-22 23:52:26 -0400
comments: true
categories: [libinput-gestures, gestures, bash, linux, make, groups]
---
I'm probably one of the only Linux users who really like gestures. Going back a web page and switching desktops with a swipe of my fingers is really easy and convenient. But when I switched to Linux, I found the support rediculously lacking. It took a week of searching and trial and error to finally figure out how to implemnet gestures, at which point I stumbled upon [libinput gestures][2b6bf2e2], which finally did solve my problem. Now that I have a solution (and a fair amount of time has passed) I want to work on a project: documenting open source. But not just going over the functions, but going extremely in depth and looking at EVERYTHING that happens.

This guide won't just show off what happens to make gestures work, but will talk about everything along the way. While the actual code is written in python, this part will only focus on bash and make. For an in depth analysis of the code, look for later parts. Because this code is based on the libinput-gestures code, you should familiarise yourself with the README and Makefile before or during your reading. Now let's get started.

  [2b6bf2e2]: https://github.com/bulletmark/libinput-gestures "libinput-gestures"

No matter where you go to do your gestures, you’ll encounter the same command before you start:

{% codeblock lang:bash %}
$ sudo gpasswd -a $USER input
{% endcodeblock %}

To understand this command, first you need to understand groups. Though they are commonly come across, they don’t get much attention. If you `chmod` a file to 764, you’re saying that the group is given permissions associated with 6 (reading and writing). The middle 3 characters when you use `$ ls -l` say the permissions the file’s group has when modifying it. Every file has not just a user who can access it, but a group. While said group is often just the user, that’s not always the case. In this case, we’re dealing with a series of files that can only be accessed by the group input.

The command gpasswd is used when administrating groups. The argument `-a` adds a user to a group. So `$ gpasswd -a $USER input` adds `$USER` (which is your username) to the group input. This is important because the group input has access to the files `/dev/input/event*` (as well as `/dev/input/mouse*`, though this is less important for gestures). So what is the point of this?

You may have heard the saying “In Linux, everything is a file.” Well, that includes input devices. And if you're going to treat physical files like devices, you need to include that actual files for the devices. These are said files. These files, whose type is character files, are essentially pipes between physical inputs and what the computer receives. Although they are simply labeled event followed by a number, running the command `$ xinput list` will tell you exactly which event pipe corresponds with which device. In this particular case, the only file necessary is the one for the trackpad.

Now that you're allowing the user access to the input devices, you can actually install libinput gestures. Immediately running `$ make install` is the easiest way to set up gestures, but that is neither the only way to set up functions nor the only way to use the makefile. Starting with the latter, the makefile includes `make all`, which tells you to run `make install` or `make uninstall`, which run the install script that will be covered next entry, and `make clean`, which cleans. The makefile also includes two more slightly more interesting rules, which I'm going to discuss in detail because their code is interesting. The first, `make doc`, is as follows (including all of the relevant code but leaving out everything else):

{% codeblock lang:make Makefile %}
DOC = README.md
DOCOUT = $(DOC:.md=.html)

doc: $(DOCOUT)

$(DOCOUT): $(DOC)
  markdown $< >$@
{% endcodeblock %}

The interesting part of this command comes not from the content, but the presentation. But what it actually does is fairly simple. The command markdown converts a text file to html. The make variable `$<` gives the name of the first prerequisite, in this case `$(DOC)`, which is defined earlier as `README.md`. The second variable, `>$@`, is a little more complex. Any variable prefixed with `>` means that if it involves a file, that file will be overridden, rather than appended to. And `$@` takes in the target of the rule, in this case `$(DOCOUT)`. `$(DOCOUT)` takes in `$(DOC)` (still `README.md`) and performs a substitution reference on it, substituting .md for .html. So essentially `$(DOCOUT)` is equivalent to `README.html`. Put together, all of this means convert to markdown README.md, outputting with override to the file `README.html`. Or, in simplest terms, converting the README from markdown to html.

The second rule, `make check` is less stylish or hard to read, but it's no less interesting. Instead, its uniqueness comes from its content. The rule is as follows:

{% codeblock lang:make Makefile %}
SHELLCHECK_OPTS = -eSC2053,SC2064,SC2086,SC1117,SC2162,SC2181,SC2034,SC1090,SC2115

check:
	flake8 libinput-gestures
	shellcheck $(SHELLCHECK_OPTS) libinput-gestures-setup list-version-hashes
{% endcodeblock %}

This is pretty standard make. The interesting part comes with the make code. The first line  just does exactly what it says, running the command `$ flake8 libinput-gestures`. `flake8` is a command that tests for proper conventions in python code. This check fails, as line 533 is longer than 80 characters. I don't know why the developers included a check if they don't intend to use it themselves, but if anyone is looking for an easy pull request, they can have one. The second line is similar in that it checks shell scripts for errors and bad code, though it differs in that the code passes the check. It also includes an argument `-e` which tells it to exclude the errors corresponding to the codes listed after it. In short, the allowed forms of bad code are redundant logic, double double quotation marks, not double quoting expanding variables, using `\n` rather than `\\n`, using read without `-r`, running then checking exit status in two steps, unused variables, not using a directive to specify the location of a source file with a variable location, and not checking to make sure a path variable isn't empty. The conclusion that can be drawn from this is shellcheck has a lot of different types of errors. I'm not sure if this is intentional or laziness, though there are a lot of fixable errors if shellcheck is run to allow these errors (though some do seem justified in the code).

So which shellcheck errors are justified? And what happens after you actually type make install? How does libinput gestures actually work? Find out next time on my blog.
