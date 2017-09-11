---
layout: post
title:  "Rails 5 render() in the Controller"
date:   2017-09-06 22:47:00 -0500
categories: rails
published: true
---
I've written a few times already about being verbose and concrete when teaching and learning programming. This best practice is especially relevant to learning Ruby on Rails, where so much functionality is obfuscated. For example, how does my Rails app know which template to render when a route is requested? When I navigate to `localhost:3000/about` in my browser, I'm requesting the About page (I want this Rails app to render a particular template). The `routes.rb` file has the corresponding route code:

{% highlight ruby %}
Rails.application.routes.draw do
  get '/about', to: 'home#about', as: :about
end
{% endhighlight %}

The route's job is only to match the HTTP verb, in this case **GET**, and some string, in this case **/about**. The `routes.rb` file in effect now says, "Go to the **Home** controller and see the `#about` method for further instructions. And let's just say among friends that this reference is known *as 'about'*." Ok, let's follow the breadcrumbs and see what awaits us:

{% highlight ruby %}
# app/controllers/home_controller.rb
class HomeController < ApplicationController
end
{% endhighlight %}

Umm.. where to next? Seems like we've reached a dead end. Rather than being explicit and verbose, Rails obfuscates its default behavior, which in this case is to look for a template named "about" in the directory `app/views/home`. It is precisely this convention in Rails that makes it simultaneously less difficult to develop in, and harder to understand as a newbie. So while we want to end up with the aforementioned controller code for our simple About page, let's build it up with verbosity. It's almost as if I'm tapping the controller with a magic wand, and watching the obfuscated code appear.

{% highlight ruby %}
class HomeController < ApplicationController
  def about
  end
end
{% endhighlight %}

So coming from Ruby, it looks like we just have an inherited class with an instance method definition sans logic. This code works and will render the About page; same as before, just a tad more explicit. But it's still obfuscating the `#render` method. Let's throw that in:

{% highlight ruby %}
class HomeController < ApplicationController
  def about
    render
  end
end
{% endhighlight %}

Looks similar to `return` or `raise` in Ruby. Simple. Let's take it another step, by telling it *what* to render:

{% highlight ruby %}
class HomeController < ApplicationController
  def about
    render "about"
  end
end
{% endhighlight %}

Ok, but what is "about", exactly? We want the "about" template located at `app/views/home`, so let's be more explicit:

{% highlight ruby %}
class HomeController < ApplicationController
  def about
    render template: "about"
  end
end
{% endhighlight %}

Drats! We get the following error now, which means that when using the `template:` argument, the path must become explicit:

```
Template is missing
Missing template /about with {:locale=>[:en], :formats=>[:html], :variants=>[], :handlers=>[:raw, :erb, :html, :builder, :ruby, :coffee, :jbuilder, :haml]}. Searched in: * "/Users/kelton/Sites/narify/app/views"
```

Let's make the path more explicit so it knows to look inside the `home` folder inside `views`:

{% highlight ruby %}
class HomeController < ApplicationController
  def about
    render template: "home/about"
  end
end
{% endhighlight %}

Alright, at this point you can probably see some duplication (which is why Rails obfuscates this if it can). We have `class HomeController` and `def about` followed by the `render template:` method where we're repeating our controller and method: `"home/about"`. I personally like to be even more explicit, like so:

{% highlight ruby %}
class HomeController < ApplicationController
  def about
    render template: "/app/views/home/about.html.haml"
  end
end
{% endhighlight %}

But this code is even more illustrative regarding the path, as evidenced by the error I get in my browser:

```
Template is missing
Missing template app/views/home/about.html.haml with {:locale=>[:en], :formats=>[:html], :variants=>[], :handlers=>[:raw, :erb, :html, :builder, :ruby, :coffee, :jbuilder, :haml]}. Searched in: * "/Users/kelton/Sites/narify/app/views"
```
Particularly that last bit: `Searched in: * "/Users/~/Sites/narify/app/views"`. If I'm not mistaken, the full path being searched would be `/Users/kelton/Sites/narify/app/views/app/views/home/about.html.haml`, so we could reasonably assert that we only need to specify the contents within the `views` folder, i.e. a relative path.

{% highlight ruby %}
class HomeController < ApplicationController
  def about
    render template: "home/about.html.haml"
  end
end
{% endhighlight %}

> `render template: "home/about.html"` also works, but not `render template: "home/about.haml"`

I'm going to start being explicit in my controllers' `#render` methods. Similar to the testing mantra of "test until fear turns to boredom", I'm going to explicitly render until I'm comfortable with the obfuscated version. Only now will I not take any magic for granted, nor will I fail to understand what's *actually* happening. You might think that overriding the obfuscated render could produce unwanted side effects, and that would be a reason for not being explicit. But I'd argue that such an event would be yet another opportunity to better understand this method and how it relates to the rest of our application.
