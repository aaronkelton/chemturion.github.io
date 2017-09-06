---
layout: post
title:  "Bash Profile and cmd + delete"
date:   2017-09-05 10:20:00 -0500
categories: fail
published: true
---
I've been taking the [Unix for Mac OS X Users](https://www.lynda.com/Mac-OS-X-10-6-tutorials/Unix-for-Mac-OS-X-Users/78546-2.html?org=dallaslibrary.org) course on Lynda, and decided to venture out on my own by editing my `.bash_profile` file. I already had some `PS1` lines, and another for setting the Ruby environment, so I inserted a harmless line: `echo "edit your shell startup in .bash_profile"` so I'd see that line every time I started my terminal program. Unfortunately when I restarted my terminal, it became inoperable! So I can't `nano .bash_profile` to fix it either.

Now I'm on the hunt for how to access the `.bash_profile` file using Finder. Unfortunately it's quite difficult to view hidden files on one's machine, and I always have to google the keyboard shortcut because for some reason you can't simply find the option in the menu. I don't want to repeat what I've done in the past (because I'm lazy)...

So then I decide to use Spotlight. I move my fingers to `cmd + spacebar`, type in ".bash_profile", and... nothing. So I then use my nimble fingers to erase that query in one-fail-swoop using `cmd + delete`, but instead of erasing the search parameter, I hear the sound of files being chunked into the trash. No problem (I thought)! I'll just undo that operation.

Well, when you select multiple files from the trash, there is no option to "put back" each into their respective directories whence they came. Instead, you've got to select individual files to perform that operation. Wow, I'm learning so much this morning!

Ok, so I looked up the keyboard shortcut again to view hidden files and folders. I would link to the website where I got the info, but macworld.co.uk has a shit-ton of ads, videos, and x-knows-what-else (I can see the browser tab's favicon spasming into the reload icon), resulting in glitchy scrolling and an overall very poor user experience, so I'll spare you the misery.

`cmd + shift + .` to view hidden files and folders from Finder.
