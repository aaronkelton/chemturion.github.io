---
layout: post
title:  "Highlighting in Atom for *.js.haml"
date:   2017-06-27 10:40:00 -0500
categories: rails haml atom
published: true
---
Having trouble this morning going through a tutorial about using Ajax in Rails. We're converting a sidebar cart to render after item is added, without reloading the entire page. All my `*.html.erb` files are already working with the `*.html.haml` replacement. But to use the Ajax, we're creating a file with called `create.js.erb`, and a simple file extension swap to `create.js.haml` isn't giving me any text highlighting in Atom.

{% highlight js erb %}
  $('#cart').html("<%=j render(@cart) %>");
{% endhighlight %}

The `erb` text is embedded in quotes into the `.html` call. I'd rather not use the clunky `erb` syntax with its percentage, equals, and closing tag.

### Solution

I've renamed my file to match the convention: `create.js.haml`. The jQuery (dollar sign) and html call stay the same, so I just replaced the `erb` portion with Haml syntax: `=j render(@cart)`. I tested locally that the code works. Now to get the highlighting, I found in the [Haml docs a thing called a filter](http://haml.info/docs/yardoc/file.REFERENCE.html#filters). To use the JavaScript filter, use a colon followed by the word: `:javascript`. Then, indent your code on the next line.

(create.js.haml)
{% highlight haml %}
:javascript
  $('#cart').html("=j render(@cart)");
{% endhighlight %}
