---
layout: post
title: "solving git lfs issues -- for when your unity assets won't load"
date: 2019-08-11 21:35:17 -0400
comments: true
categories: [solutions, bash, git, unity]
---
Unlike my usual posts on how something works, this post is a solution post. It presents a problem I recently had, and how I solved it. If you’re also interested in solving things, go ahead and read this, and maybe try to find a solution and compare your answers with mine. But the intended audience for this post is people who are trying to solve this problem.

I’ll start by describing the problem. I recently participated in a game jam with some friends. [Here](https://hwkeyser.itch.io/one-engine-left) is our game, if you want to try it out. But on two computers, mine (Arch Linux), and a friend’s (oldish Mac), none of the assets would load. We were using Unity, with both our own custom asset files and TextMesh Pro assets, both of which failed to load.

So how did I solve this?

First I checked on the asset files themselves. At first I was worried they weren’t cloned correctly, and when I `cat`ed them I got this.

{% codeblock lang:bash %}
$ cat Sunny\ Days\ -\ Seamless.jpg
version https://git-lfs.github.com/spec/v1
oid sha256:ab6536daf74188dfc075d65f5d85aca9efeaead7f03b905514e53076d854f365
size 273488
{% endcodeblock %}

Well, that doesn’t look like what usually happens when you cat a jpg. Was it uploaded wrong? I went to the GitHub page for the file to check. But the image on GitHub looked normal. Well, with one added detail.

![Stored with Git LFS](images/2019/08/bloglfs.png)

`Stored with Git LFS`. Clicking on the question mark took me [here](https://help.github.com/en/articles/versioning-large-files). Essentially, Git LFS, or Git Large File Storage, is GitHub’s way of storing large files. Rather than put the files where they appear in the repository, it stores them elsewhere and puts a pointer file in their place. You can read more about it [here](https://help.github.com/en/articles/about-git-large-file-storage). But what does this have to do with solving the problem? Well now that we know the reason files look strange is because they are being hidden with Git LFS, we can use Git LFS to find them.

Unfortunately, Git LFS isn’t preinstalled, at least on some systems. Fortunately, it is at least on the AUR and homebrew. Weirdly enough, the package has to be downloaded manually on Debian/Ubuntu, but you can find the instructions for that [here](https://github.com/git-lfs/git-lfs/wiki/Installation#debian-and-ubuntu).

Once you’ve downloaded git-lfs, you need to install it. Just do it once with

{% codeblock lang:bash %}
$ git lfs install
{% endcodeblock %}

Now Git LFS is installed! Any new repos you clone will clone large files properly automatically. Unfortunately, that doesn’t include current ones. Those require a bit of extra work. If you’re feeling lazy, you can just delete and reclone the repo. Otherwise, just type

{% codeblock lang:bash %}
$ git lfs fetch
$ git lfs pull
{% endcodeblock %}

And you’re done! If you cat image files, you will be greeted with binary files and not plaintext. And now if you open up the project in unity, you will be able to use assets. Happy coding!
