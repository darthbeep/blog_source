---
layout: post
title: "Analyzing an Open Source Project: Part II -- Reading Installation Script Arguments"
date: 2019-07-03 03:56:58 -0400
comments: true
categories: [libinput-gestures, gestures, bash, linux, getopts]
---
Here we begin Part II of Analyzing an Open Source Project. For Part I, which deals with setting up and looking at a makefile, click here. The open source project I'm analyzing is libinput-gestures, and this part begins looking at the setup script. Click here for the code. To keep each article from getting too long, the setup script will require multiple articles. This part will focus on getting the program name and article parsing.

Some things start off with a bang, but libinput-gestures-setup starts off with a shebang. So we know that the script is meant to be run in bash, but what else do we know? Well, just like us, the script starts by trying to learn more about itself.

{% codeblock lang:bash libinput-gestures-setup %}
PROG="$(basename $0)"
NAME=${PROG%-*}
{% endcodeblock %}

The argument `$0` is the name of the command or script that is run. For example, the `$0` for `ls -l` is `ls`. The command `basename` strips away both the path to the file and the file extensions. So if you were to call `libinput-gestures/libinput-gestures-setup` or even .`/libinput-gestures-setup`, it would take away the first part, making `$PROG` result in `libinput-gestures-setup`.

In bash, a `%` means to remove what comes after it from the suffix. And since what comes after it is `-*`, meaning the `-` and everything after it are removed, making `$NAME` become `libinput-gestures`. For some interesting and related facts, while `%` removes everything after the the last instance it finds, `%%` removes everything after the first instance. While `%` and `%%` deal with suffixes, their prefix equivalents are `#` and `##` (however, this time `##` looks for the last instance). And finally, it is an absolute pain to google the symbol `%`. Yes, even with quotes.

Great! Now we've gotten through two lines of code! Wasn't that exciting? Well, now we're going to skip around a bit. Don't worry, well cover everything we missed. But first, let's zoom out for a second. The setup script allows for two command line arguments, -d and -r. -d takes and argument, which is used to determine the destination where everything is installed. By default, it's empty and causes everything to be installed in children of the root folder, but the script does give you the option to specify. You probably shouldn't need to if you just plan on installing, but it's good to have options. The -r argument force allows root to perform user commands, and is not recommended. For a more in depth look at these commands, keep reading my blog. Until then, let's look at how arguments are actually parsed.

DESTDIR=""
FORCEROOT=0
while getopts d:r c; do
    case $c in
    d) DESTDIR="$OPTARG";;
    r) FORCEROOT=1;;
    \?) usage;;
    esac
done

shift $((OPTIND – 1))

The first two lines are simple, they're setting the defaults of variables that may or may not be overridden later. But the third is where things get interesting. Given that many bash functions take in arguments of the form `-a test -b`, there needs to be a builtin to parse them. And that's what getopts is. It's used in a while loop (or for each loop, whichever helps to visualize it better) to make sure that every option is parsed, as the loop will run for every option. Now let's look at the first argument, d:r. The script's two arguments, -d and -r are present, with -d, which takes an argument, on the left, and -r, which does not, on the right. The second getopts argument, `c`, will be filled with the argument the user uses. At the same time, if applicable, the builtin $OPTARG will be filled with the argument's argument. For example, if the user types in the command ./libinput-gestures-setup -d place -r, the first pass of the while loop will fill c with d and $OPTARG with test, and on the second pass c will fill with r.

Now that we know c is being filled in with something, the question is what we can do about that. And the answer is a case statement. Interestingly enough, the case command's man page calls it obsolete and says it should be replaced by switch statements. Though case statements seem far more popular online. In this particular case, the programmer probably doesn't need to switch, but it's something to keep in mind.

Once you get past the confusing getopts syntax, case statements are fairly readable. The syntax checks what $c is against d, and if so sets DESTDIR to $OPTARG, which holds -d's argument. All case statements are terminated with two semicolons. Similar is the case for r. If an illegal option is called, the case \? will trigger. This calls the function usage, which outputs the script's usage as text and then quits with an error. After the case is chosen, the case statement will end with `esac`.

Why `esac`? To slightly paraphrase this answer, the case / esac convention, as well as if statements ending in fi, comes from ALGOL68. Both the original sh shell and ALGOL68 have, the same author, Stephen Borne. One of the influences sh took from ALGOL68 are the case/esac and if/fi openings and closings. ALGOL68 also has do/od as a pair, but od is already a keyword in bash (octal dump), so that was scrapped in favor of done.

This leaves us with one final line of code, `shift $((OPTIND - 1))`. Of the statement above, this part is the most enigmatic. If you google around to understand getopts, you'll find that this part is usually ignored. You don't achieve anything by modifying it, and it's something you put at the end no matter what. So, what does this command actually do, and how does it work?

Starting from the beginning, the variable $OPTIND, short for option index, is part of the getopts utility. When getopts runs through the command line argument, like a for each loop, it needs something to look at next. At all times, the next argument for getopts is stored in $OPTIND. How does this work so easily? $OPTIND isn't actually storage, but a number. Remember, $1 in bash will get you the first argument, $2 the second, and so on. So $OPTIND starts at 1 and is incremented with each pass. If you run the program with 3 arguments, $OPTIND will end as 4.

In bash, double parentheses evaluate an expression, and a $ in front of them returns the result as a statement. So shift is being called on the result of $OPTIND - 1. `shift` is a command which shifts the position of arguments to the left by the number given to it as argument. That is, if you run shift 2, the variable that used to be referenced with $4 can now be referenced with $2. Because $OPTIND is always one more than the number of optional arguments, shifting by $OPTIND - 1 will make the main arguments always be in the same place. So for example calling `./libinput-gestures-setup -d “place” install` and `./libinput-gestures-setup install` will both lead to install being $1 after the shift happens.

The next function is fairly simple.

if [[ $# -ne 1 ]]; then
    usage
fi

The program wants you to input only one main command, in this case start, stop, restart, autostart, autostop, or status. If you give it a number of commands that is not one, it will display the usage. The program checks if you are giving it only one argument (after the shift), with `[[ $# -ne 1 ]]`. Double square brackets surround a statement. `-ne` compares two numbers to see if they are not equal, in this case `$#` or the number of arguments, and 1. So if the number of arguments is not equal to one, the usage function is triggered. Otherwise, you can proceed on to

cmd="$1"

Whatever the main command is, it's stored in the argument $1. So now the script knows what you're trying to do, and sets it as the cmd variable.

That's all for this entry, but keep reading to find out what's next. Check out this blog next time for more exciting code. In either way you interpret that sentence.
