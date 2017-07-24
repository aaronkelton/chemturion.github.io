---
layout: post
title:  "DRY Out your Header in Rails App"
date:   2017-07-24 12:00:00 -0500
categories: rails beginner application
published: true
---
I've got a root home page, an about page, and a couple show pages in my app. I'd like to have a navigation bar at the top that doesn't have to be hard coded in every page. I think this is done in `app/views/layouts/application.html.erb`, and all the other pages get "yielded" to, if that makes sense. I'm first going to confirm these changes in my dummy generated app, and then create from scratch in my narify app.

So let's make our "navbar" just a primary header with our app's title linked to the root, and a link to the About page. I'm going to create a new file called `app/views/layouts/_navbar_header.html.haml` and put in the following code:

{% highlight haml %}
%h1 Narify Navbar Header Partial
%h2
  = link_to 'Home', root_path
%h3
  = link_to 'About', about_path
{% endhighlight %}

The `h1` will remind me what the element is, the `h2` and `h3` links will always be at the top of every page, no matter what. So now let's remove links to `Home` and `About` in our other existing pages. So now let's tell `app/views/layouts/application.html.erb` to `render` the navbar header partial view immediately before the `yield`.

{% highlight erb %}
<body>
  <%= render 'layouts/navbar_header' %>
  <%= yield %>
</body>
{% endhighlight %}

Notice that we don't have to include the underscore in the `render 'layouts/navbar_header'` command. Also, because we used the generic (abstracted) routes and link_to earlier, we don't have to update our `link_to` code or our `routes.rb` file. Last step is to redo this approach by hand in my narify app.

First step is to create the partial view `_navbar_header.html.haml`, just like above, and then remove the references to Home and About from individual pages. However, the first difference I notice is that I don't have a `layouts` folder, so I'll create that folder. And because I just created that folder, I now realize that I've been operating without `layouts/application.html.erb` this whole time! Before we create the partial, let's create this important file first.

{% highlight haml %}
!!!
%html
  %head
    %title Narify
    = csrf_meta_tags
  %body
    = render 'layouts/navbar_header'
    = yield
{% endhighlight %}

I basically Hamlized the auto-generated code (minus the stylesheet and JS references) and included the `render` statement. Let's make that partial view now (same as above).

{% highlight haml %}
%h1 Narify Navbar Header Partial
%h2
  = link_to 'Home', root_path
%h3
  = link_to 'About', about_path
{% endhighlight %}

Admittedly, when I Hamlized the `application.html.erb` file, I got an error in browser, so I'll have to remember to bring those lines back in when I'm ready to include CSS and Javascript.
