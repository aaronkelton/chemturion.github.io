---
layout: post
title:  "The Bare Minimum to Run rails server"
date:   2017-07-19 19:00:00 -0500
categories: rails beginner
published: true
---
After I ran `rails new narify --database=POSTGRESQL` I removed from the directory as many files and folders as possible while still being able to run `rails server`. Here's what I found:

```
.
bin
config   
config.ru
Gemfile
Gemfile.lock
```

When you run `rails server` it'll recreate the `log` and `tmp` directories. So what's been removed? First I removed the local git with `rm -rf .git`, and here are the files and folders removed:

```
.
app
db
lib
log
public
test
tmp
vendor
.gitignore
package.json
Rakefile
README.md
```

Now begins the arduous task of building my own app without generators. I suspect I'll need the `app`, `db`, and `lib` folders, but perhaps can leave out the others.
