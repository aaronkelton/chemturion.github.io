---
layout: post
title:  "Rails - Show a Narration"
date:   2017-07-21 18:38:00 -0500
categories: rails beginner model
published: true
---
My Narify app currently has a list of sample narrations as an unordered list. At this point I want to add a `Listen` link, which will navigate to the narration's corresponding page. So... where to start? I think I'll need to define the `#show` method in the Narrations controller, and make available the narration in question as an instance variable, like `@narration = Narration.find(params[:id])`.

{% highlight ruby %}
class NarrationsController < ApplicationController
  def home
    @narrations = Narration.all
  end
  def show
    @narration = Narration.find(params[:id])
  end
end
{% endhighlight %}

Now let's update the route file with `get '/narrations/:id' => 'narrations#show', as: :narration`. This should let me use the `link_to` thingy, or let a user type a URL with an id.

{% highlight ruby %}
Rails.application.routes.draw do
  root to: 'narrations#home'
  get '/narrations/:id' => 'narrations#show', as: :narration
end
{% endhighlight %}

At this point I'd like to test that the route works and will display a narration. So I'm going to create a new file in `app/views/narrations/show.html.erb` and insert some code. Here it is:

{% highlight haml %}
!!!
%h1 This is a narration show page.

= @narration.title
{% endhighlight %}

The only way to get here is go to `narify.com/narrations/1` where `narify.com` is my localhost and `1` is just some id. I only have two sample records in my database, so there's only `1` and `2`. Let's add a link to the list on my homepage.

{% highlight haml %}
- @narrations.each do |narration|
  %ul
    %li
      = narration.title
      = link_to 'Listen', narration_path(narration)
{% endhighlight %}

I honestly had a hell of a time getting that `link_to` bit to work. The [Rails Routing from the Outside In](http://guides.rubyonrails.org/routing.html) instructions made it seem like the correct route would have been `link_to 'Listen', narration_path(@narrations)` or even the shorthand `link_to 'Listen', @narrations`, but I'm not sure in what situation I'd use that convention.

Let's add another link, this time on the `/narrations/:id` page; something like `link_to 'Back to Narrations', root_path`. Maybe that'll work. Let's see.

{% highlight haml %}
= @narration.title
= link_to 'Back to Narrations', root_path
{% endhighlight %}

And that works. I can go back and forth. Not sure what to do next, but I'll figure that out another day.
