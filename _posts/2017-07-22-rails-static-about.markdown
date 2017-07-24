---
layout: post
title:  "Adding a Static About Page in Rails"
date:   2017-07-22 12:00:00 -0500
categories: rails beginner route
published: true
---
I've been building the Narify application based on a user that hasn't logged into the system, i.e. unauthenticated. The Ruby on Rails people in my life have informed me that there's this nifty [Devise apparatus](https://github.com/plataformatec/devise) for handling user authentication. Looking forward, I'll want to have the following in my `routes.rb` file:

{% highlight ruby %}
unauthenticated do
  match '/about', to: 'home#about', via: :get
end
{% endhighlight %}

Which for me at this point in the app's infancy, means I'll need a HomeController, home folder for views, and the following in my routes file:

{% highlight ruby %}
get '/about', to: 'home#about', as: :about
{% endhighlight %}

So let's make this happen. We'll refactor (change our code) at a later date once we decide to have authenticated users. And I'm pretty sure we'll need to refactor the `root to:` narrations as the home page. Eventually we'll want that as a page-within-a-page. Maybe I'll do that next, but first let's create that controller to correspond to my new 'about' route.

{% highlight ruby %}
class HomeController < ApplicationController
  def about
  end
end
{% endhighlight %}

And now I need a Haml view to render, so let's create `app/views/home/about.html.haml`:

{% highlight haml %}
%h1 This is the static 'about' page.
= link_to 'Back to Narrations', root_path
{% endhighlight %}

And testing `localhost:3000/about` successfully renders the about page, so the last bit I want to do is create a link on my Narrations home page to take me to the About page. Here's my updated `home.html.haml` file:

{% highlight haml %}
%h1 This is the Narrations home page.
%h2
  = link_to 'About', about_path

- @narrations.each do |narration|
  %ul
    %li
      = narration.title
      = link_to 'Listen', narration_path(narration)
{% endhighlight %}

Nothing fancy, just a secondary heading between the primary heading and my narrations list.

Even though this Rails app is extremely rudimentary, I can understand the need for tests and proper organization. I'm already thinking I'll need to refactor my code and folder/file structure. I think I'll do that next.
