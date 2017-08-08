---
layout: post
title:  "Removing Ruby Version Manager (RVM) after Strange Rails Error"
date:   2017-08-08 13:30:00 -0500
categories: rails rvm errors
published: true
---
This morning I was going to create a small app as a coding test for an interview. But instead of building, I'm climbing through a tangled mess of errors. Normally you should be able to `rails new myapp`, `cd myapp`, and `rails server` gets your success page. Well, I got that far, but the next command, `rails generate controller Bots` gave me a slew of errors, starting with this one on top:

```
/Users/~/.rvm/gems/ruby-2.4.1/gems/actionpack-5.1.3/lib/action_dispatch/routing/mapper.rb:327:in `check_part': Missing :controller key on routes definition, please check your routes. (ArgumentError)
```

I recently went thru a React tutorial where we used RVM, but I had previously used RBENV to manage my Ruby versions. I don't know if this was the problem, but I decided to wipe RVM from my system using this [Rich on Rails guide](https://richonrails.com/articles/uninstalling-rvm).

`rvm implode` is the first command I ran. Here's my terminal output:

```
Are you SURE you wish for rvm to implode?
This will recursively remove /Users/~/.rvm and other rvm traces?
(anything other than 'yes' will cancel) > yes
Removing rvm-shipped binaries (rvm-prompt, rvm, rvm-sudo rvm-shell and rvm-auto-ruby)
Removing rvm wrappers in /Users/~/.rvm/bin
Hai! Removing /Users/~/.rvm
/Users/~/.rvm has been removed.

Note you may need to manually remove /etc/rvmrc and ~/.rvmrc if they exist still.
Please check all .bashrc .bash_profile .profile and .zshrc for RVM source lines and delete or comment out if this was a Per-User installation.
Also make sure to remove `rvm` group if this was a system installation.
Finally it might help to relogin / restart if you want to have fresh environment (like for installing RVM again).
```
Next I ran `open .bashrc`, `open .bash_profile`, and `open .profile`, and `open .zshrc` and deleted all lines that had any mention of "rvm". Then I ran `rm -rf .rvm` and `rm -rf .rvmrc`. When I tried to `cd` to another folder, I got this warning:

```
__rvm_do_with_env_before:source:5: no such file or directory: /Users/~/.rvm/scripts/initialize
__rvm_do_with_env_before:source:5: no such file or directory: /Users/~/.rvm/scripts/initialize
ruby-2.4.1 is not installed.
To install do: 'rvm install ruby-2.4.1'
__rvm_after_cd:source:5: no such file or directory: /Users/~/.rvm/scripts/hook
```

So I think I need to delete the hidden .rvm folder. But I first need to figure out how to view hidden folders. I've done it before but don't remember, and I don't think it's in the menu. `cmd + shift + .`. But... there's no .rvm folder. So now I need to understand why I'm getting this warning. I'm getting it no matter what folder I `cd` into. Let's restart my machine and see if that does anything.

OK! That seems to have worked! I was able to `cd` without warning, and also generated a controller like usual. I'm not sure if rbenv and rvm were at odds with one another, or if rvm just became corrupted, but I like to think that software doesn't just spontaneously combust.
