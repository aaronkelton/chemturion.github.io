---
layout: post
title:  "React on Rails 5 - Getting Started using react-rails"
date:   2017-08-01 20:59:00 -0500
categories: rails react asset_pipeline
published: true
---
I've decided to save Bootstrap and all styling for a later date. I think the look and feel would detract from my focus on making things work. So I'm going through a [React on Rails course](https://learnetto.com/course_parts/656) and blogging my thoughts herein.

So first things first: confusion. Apparently there are 3 gems (at least) to use React. You've got `react-rails`, `react_on_rails`, and `webpacker`. The tutorial I'm going thru uses `react-rails`, so that's what I'll be using. Before I add that to my Gemfile, let's try to install React using a Rails generator: `rails generate react:install`:

```
Running via Spring preloader in process 35575
Could not find generator 'react:install'. Maybe you meant 'test_unit:job', 'channel' or 'assets'
Run `rails generate --help` for more options.
```

I just like to see error messages to get a better picture of who's-doin-what. Let's add that line to our Gemfile now.

{% highlight ruby %}
gem 'react-rails'
{% endhighlight %}

Ok, so `bundle install` and `rails generate react:install` and let's see what we get now:

```
Running via Spring preloader in process 40155
      create  app/assets/javascripts/components
      create  app/assets/javascripts/components/.gitkeep
      insert  app/assets/javascripts/application.js
      create  app/assets/javascripts/components.js
      create  app/assets/javascripts/server_rendering.js
      create  config/initializers/react_server_rendering.rb
```

Let's look at the Ruby file first `config/initializers/react_server_rendering.rb`:

{% highlight ruby %}
# To render React components in production, precompile the server rendering manifest:
Rails.application.config.assets.precompile += ["server_rendering.js"]
{% endhighlight %}

So, magic on the left adds a JavaScript file from the right. Let's peek inside that `app/assets/javascripts/server_rendering.js` file:

{% highlight javascript %}
//= require react-server
//= require react_ujs
//= require ./components
//
// By default, this file is loaded for server-side rendering.
// It should require your components and any dependencies.
{% endhighlight %}

And this is where I really get uneasy. Comments are supposed to be ignored, yet this kind of syntax explicitly uses them, and we get no syntax highlighting either. Looking past that momentarily, we're just requiring (think of it as importing or extending a module, so to speak) a couple files and the components folder, which is essentially empty except for a `.gitkeep` file. I know there's also a `components.js` file, which can be confusing. It's the folder because of the `./`.

Let's look inside the other files, too, starting with `components.js` since that's more React-specific than `application.js`:

{% highlight javascript %}
//= require_tree ./components
{% endhighlight %}

So, it also says to require the components folder. What about `application.js`?

{% highlight javascript %}
//= require react
//= require react_ujs
//= require components
{% endhighlight %}

The application file here requires the components file (not folder), and also requires react at the outset. So that gets our bearings around what just happened. If I had tests, hopefully they'd still pass amidst these new additions. Instead I'm going to run `rails server` to make sure it works for me. Ok, todo bien!

I'm going to change my home page, which is a list of Narrations, to use React. So I'll create a new file to match my controller, `app/assets/javascripts/components/narrations.jsx`:

{% highlight javascript %}
var Narrations = React.createClass({
  render: function() {
    return (
      <h1>This is my Narrations index/home page as COMPONENT.</h1>
    )
  }
})
{% endhighlight %}

Ok, this will just be static, but it's a start. Now let's jump over to our actual view: `app/views/narrations/home.html.haml` and plop that new _component_ on top:

{% highlight haml %}
= react_component 'Narrations'
{% endhighlight %}

But unfortunately nothing shows in my browser. I did inspect the page and got:

{% highlight html %}
<div data-react-class="Narrations" data-react-props="{}"></div>
{% endhighlight %}

So something React-y is working, but something else isn't. How to fix... let's try replicating it with a not-the-hard-way Rails app. And... that worked! So now my task is to understand who depends on what. Let's start with `application.js`. Straight away at the top comparing my from-scratch app to my generated-app, I notice this line missing: `//= require rails-ujs`. Having the name 'rails' in it makes me think it's fairly important, so let's include that and start the app again using `rails server`. Nothing, but I did get a chance to compare DOMs. Inspecting each output showed that the working one had the following HTML:

{% highlight html %}
<div data-react-class="Narrations" data-react-props="{}">
  <h1 data-reactroot="">Narrtions from class extends React component.</h1>
</div>
{% endhighlight %}

So there's something missing that gives me the `data-reactroot` attribute in my tag. Oh, I forgot to add that I changed my React Component code:

{% highlight javascript %}
class Narrations extends React.Component {
  render () {
    return <h1>Narrtions from class extends React component.</h1>
  }
}
{% endhighlight %}

I don't think Turbolinks has anything to do with the React component not rendering, so I'll add the `//= require_tree .` statement.

{% highlight javascript %}
//= require rails-ujs
//= require react
//= require react_ujs
//= require components
//= require_tree .
{% endhighlight %}

Ok, even after that and adding Turbolinks, still no success. So let's go up a level. We need to speak to the manager! The `javascripts` folder is nested in the `assets` folder. Its nearest neighbor of interest is the `config` folder, and therein lies the `manifest.js` file. Sounds important. Let's dive in.

{% highlight javascript %}
//= link_tree ../images
//= link_directory ../javascripts .js
//= link_directory ../stylesheets .css
{% endhighlight %}

Doesn't look too impressive, but when I create one for my from-scrath app, does it make React work? Negative. Ok, I feel like I'm not getting any closer to a solution taking this route. Inspecting the elements and viewing "Sources" in Chrome's dev tools, I see that my from-scratch app isn't loading my assets folder. Sensing that this has to do with the "asset pipeline", I discovered Ryan Bates' [Railscast episode on the Asset Pipeline](http://railscasts.com/episodes/279-understanding-the-asset-pipeline?autoplay=true). So let's look at my asset path's between both apps running `Rails.application.config.assets.paths`:

This is my from-scratch app:
```
- "/app/assets/javascripts"
```

And this is the generated app:
```
- "/app/assets/config"
- "/app/assets/images"
- "/app/assets/javascripts"
- "/app/assets/stylesheets"
```

Now, I'm not using any images or stylesheets, but I suspect that the config directory is pretty important. -(24 hours later)- So I went thru a webpacker implementation, and noticed my app's layout file differed, so I added `= javascript_include_tag 'application', 'data-turbolinks-track': 'reload'` to my `app/views/layouts/application.html.haml` file's head section. This tag is included in a Rails 5.1 new app, but remember I decided to do things the hard way, so it was missing for me and didn't raise any errors! But now it works and my check in `rails console` for my asset paths showed the config. Read the [Rails docs for javascript_include_tag here](http://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html#method-i-javascript_include_tag).
