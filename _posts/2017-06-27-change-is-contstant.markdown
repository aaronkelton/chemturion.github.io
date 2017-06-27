---
layout: post
title:  "Adapting to Constant Change"
date:   2017-06-27 13:45:00 -0500
categories: rails jquery
published: true
---
OK, so I just encountered an error and discovered a solution to the fix. But first, some background. Going thru the book, __Agile Web Development with Rails 5__, which was published late 2016. So it's been 8 months and one of the lines published doesn't work.

We're making the recently added line item *highlighted* using `jquery-ui-rails`, so we add the following to the Gemfile and `app/assets/javascripts/application.js`.

(Gemfile)
{% highlight ruby %}
gem 'jquery-rails'
gem 'jquery-ui-rails'
{% endhighlight %}

(application.js)
{% highlight javascript %}
//= require jquery
//= require jquery-ui/effect-blind
//= require jquery_ujs
{% endhighlight %}

Unfortunately, something has changed since the published code worked. `Sprockets::FileNotFound in Store#index`
![jQuery UI Effect Blind](/assets/jquery-ui_effect_blind.png "Error in need of application.js fiddling")

I found [the solution in the publisher's forum](https://forums.pragprog.com/forums/311/topics/12000), albeit for the book's pervious version. Slip in `/effects/` between the `jquery-ui` and `effect-blind`.

(application.js)
{% highlight javascript %}
//= require jquery
//= require jquery-ui/effects/effect-blind
//= require jquery_ujs
{% endhighlight %}

But apparently, `//= require jquery.ui.effect-blind` used to be how this effect was specified. How long will the current solution hold?

### Takeaway

The means by which solutions are implemented are subject to change. This prospect sucks, because it redirects our energy from building to patching. You might think we're done and should move on back to building. But I want to better understand where this solution came from. So let's see if we can reproduce what [Sam R](https://forums.pragprog.com/users/279043) suggested. The source wasn't referenced, so we gotta track it down.

I first checked out the [jQuery API for the Blind Effect](https://api.jqueryui.com/blind-effect/) to no avail. Next I thought it's probably related to the actual [`jquery-ui-rails` Ruby gem](https://github.com/jquery-ui-rails/jquery-ui-rails). Sure enough, in the README.md file, you can find the most up-to-date code for [including jQuery UI Effects](https://github.com/jquery-ui-rails/jquery-ui-rails/blob/master/README.md#effects).

But even if the README.md was incorrect, you could verify for yourself by digging into the repo, where the source code presumably lives: `jquery-ui-rails/app/assets/javascripts/jquery-ui/effects/effect-blind.js`. I'm guessing it follows this path. You can also [look into the history of this repo](https://github.com/jquery-ui-rails/jquery-ui-rails/commit/d504a40538fe5f7998439ad2f8fc5c4a1f843f1c) to see the changes by doing a Cmd+F for "effects".
