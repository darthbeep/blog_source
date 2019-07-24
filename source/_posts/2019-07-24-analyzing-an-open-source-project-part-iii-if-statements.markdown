---
layout: post
title: "Analyzing an Open Source Project: Part III -- If Statements"
date: 2019-07-24 15:49:35 -0400
comments: true
categories: [libinput-gestures, gestures, bash, linux, conditionals, or]
---
And here's Part III of Analyzing an Open Source Project. Last time we started reading the install script of libinput-gestures (here's the file we're looking at), and we haven't sped up since. The goal of this blog is to read open source projects, but to look at everything. Every. Single. Thing! There are plenty of tangents but you won't be missing out on a single detail. So without further ado.

Now that we've set our cmd variable to the entered command and done our variable parsing, it's time to use our variables. While the next things to look at are functions that will be run later, we're going in execution order, so now we're going to skip to line 147 where the script really starts happening.

if [[ $cmd == install || $cmd == uninstall ]]; then

Everything until the end is wrapped in this if statement and its corresponding else. This is a typical example of a bash if statement, and contains no surprises. It can be read with knowledge of any other programming language, or even English. Simply put, the if statement checks the truthfulness of what's in the double brackets. Single brackets could also be used. While double brackets have less surprises and are more smoothed out, single brackets are a POSIX standard and therefore more portable. Because double brackets are a bash extension, they do require the user to learn a little more, but most of that is intuitive or googleable. But if you code, you can generally choose which one you want to use.

The contents of the if statement are exactly what they seem. $cmd == install returns, well, if $cmd is equal to install. The reason == is used instead of -eq is because -eq is for numbers and == is for strings.

At first I wanted to say that || worked differently in bash based on the fact that is often used outside of scripts. It's common to write `function || error`, where error will run if function returns an error (manifested by a non-zero value, in contrast to most programming languages, though this makes sense because many bash scripts use their number outputted as the error message). However, instead of working differently in and out of conditionals, || works exactly the same, though it is often used differently. It goes to the first value, sees if it's true or not, and continues until something is true. It then returns if it is able to find a true statement or not.

Realizing that in bash || is more commonly used outside of conditionals than in them, I wondered if it were true in other languages. In perl, if a function might error, it's common to write code similar to `function() or die`, where if the function fails the die command is run with an optional error message. But what about other languages, where || is confined to if statements? Well, I opened up a python shell.

'psyduck'
>>> 1 and "psyduck"
'psyduck'
>>> 0 and "penn"
0
>>> 1 or "muack"
1
>>> 0 or "blob"
'blob'

And C?

#include <stdio.h>

int main() {
        1 && printf("psyduck ");
        0 && printf("penn ");
        1 || printf("muack ");
        0 || printf("blob\n");

        //prints "psyduck blob"

        return 0;
}

Note: Psyduck, Penn, Muack, and Blob are my stuffed platypodes. They are very cute.

It seems that instead of one line if statements, and or or operands with error messages could work as well. In fact, I may start using them to shorten my code. But back on topic. After the brackets are closed again, a semicolon stands in place for a newline, which is followed by a then, which says that everything after should be executed. Then is needed because if statements don't need to be confined to just one set of brackets (and in fact can have any amount of brackets from 0 to infinity, with any number of things stringing them together). Now that the then is in place, we can finally analyze the if statement. Let's look at it's first line.

DESTDIR="${DESTDIR%%+(/)}"

Last time we determined that DESTDIR is by default `""` but can be set to any directory with the -d argument. We also covered substitution, which tells us that everything after the %% containing exactly `+(/)` is removed. Because this substitution is extremely specific and doesn't contain a common case, I believe it is an artifact from testing.

Next we have another if statement.

if [[ -z $DESTDIR && $(id -un) != root ]]; then
	echo "Install or uninstall must be run as sudo/root."
	exit 1
fi

I'm going to avoid repeating myself and not discuss anything previously discussed. For the if statement to trigger, two conditions must be met. First, -z checks if something is null, so $DESTDIR be an empty string. Second the output of `id -un` must not be equal to root. `id` is a function that gives information on users, with -u giving only information on the user running the command and -n giving the user's name. So id -un will only output root if the user is running the script as root. Therefore, the if statement will only trigger if $DESTDIR is an empty string (the root directory with a / tacked onto it) and if the user is not root. Essentially checking if the user is trying to build the directory in a place they have access to. If the user doesn't have access to the directory, the script will exit with an error.

However, this method of checking is not perfect. If the user tries to build in a non root directory they don't have access to as non root (and this is why you shouldn't give a person and a place the same name), this error checker wont trigger.

The smarter thing would be to check if the user is not root and not the owner of the file (admittedly this leaves out groups and everyone-can-edit directories, but those are really edge cases). The command `stat` gives information on the file, and its --format argument allows it to display only certain things. When --format is set to `"%U"`, it only prints out the user. So put together.

if [[ $(id -un) != $(stat --format="%U" "$DESTDIR/") && $(id -un) != root ]]; then
	echo "Install or uninstall must be run as sudo/root."
	exit 1
fi

Because apparently six lines merit a whole entry, that's all for today. But on the bright side, I actually wrote some code. And I got to plug my stuffed platypodes. Keep reading and you might get a picture. But that's all for today.
