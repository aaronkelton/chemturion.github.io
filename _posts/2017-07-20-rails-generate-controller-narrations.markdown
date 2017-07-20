---
layout: post
title:  "Generating a Rails Controller the Hard Way"
date:   2017-07-20 08:00:00 -0500
categories: rails beginner controller
published: true
---
Most Rails tutorials start by generating a controller, route, and view— without a model. That's where I'll start. So in my dummy app, I'll run `rails generate controller Narrations` and see what the terminal yields:

```
Running via Spring preloader in process 68542
      create  app/controllers/narrations_controller.rb
      invoke  erb
      create    app/views/narrations
      invoke  test_unit
      create    test/controllers/narrations_controller_test.rb
      invoke  helper
      create    app/helpers/narrations_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/narrations.coffee
      invoke    scss
      create      app/assets/stylesheets/narrations.scss
```

I think I can get away with only creating the first controller file and view folder without the test, helper, coffee, and scss files. However, there were some other folders and files in the generated `app` folder so I'll try to run my app barebones and slowly add in the necessary pieces. I'm still getting the following image from localhost, so I think I need to define a route to override this behavior.

![rails new image]({{ site.url }}/assets/rails_new_image.png)

Because my `localhost:3000` represents `narify.com`, I need to use the [`root to: 'controller#action'`](http://guides.rubyonrails.org/routing.html#using-root), or just `root 'controller#action'` for short. But I prefer being as verbose as possible in the beginning, so let's do it the long and hard way. I'll do `root to: 'narrations#home'`. So here's my code so far:

{% highlight ruby %}
# config/routes.rb
Rails.application.routes.draw do
  root to: 'narrations#home'
end
{% endhighlight %}

{% highlight ruby %}
# app/controllers/narrations_controller.rb
class NarrationsController < ApplicationController
  def home
    #code
  end
end
{% endhighlight %}

At this point `localhost:3000` will see that it should literally `root to:` the narration controller's home action. But, what will pop up in my browser? What view is going to be rendered? Well, I haven't created that bit yet, and is why I got the following error in my browser:

```
ActionController::UnknownFormat in NarrationsController#home
NarrationsController#home is missing a template for this request format and variant. request.formats: ["text/html"] request.variant: [] NOTE! For XHR/Ajax or API requests, this action would normally respond with 204 No Content: an empty white screen. Since you're loading it in a web browser, we assume that you expected to actually render a template, not nothing, so we're showing an error to be extra-clear. If you expect 204 No Content, carry on. That's what you'll get from an XHR or API request. Give it a shot.
```

So let's give it a shot by creating the file `app/views/narrations/home.html.haml`. Usually you'd see `.erb` instead of `.haml`, but I prefer to use Haml templates. To use Haml, put `gem 'haml'` in your Gemfile and run `bundle install`. Now we have a narrations controller with a home method, and a views/narrations folder with a home template. Peachy! It works.

At this point I want to replicate what I've done in my dummy app for my actual barebones app. After doing so, I get the following error in my browser:

```
Routing Error
uninitialized constant ApplicationController
```

I think I'm getting this error because I don't have the generated `application_controller.rb` file. Let's add that now.

{% highlight ruby %}
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
end
{% endhighlight %}

Success! That concludes this effort to get a barebones Rails app up and running.
