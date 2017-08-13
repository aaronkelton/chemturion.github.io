---
layout: post
title:  "Converting Rails View to React using Webpacker"
date:   2017-08-07 18:30:00 -0500
categories: rails react webpacker
published: false
---
I previously installed `react-rails`, but I've decided to go the `webpacker` route to use React. In order that my app files don't become bloated, I first need to delete everything that was installed and get back to a good base level. I don't know how to do this using git, otherwise I'd give that a go. Maybe I'll write a post on that later.

So I've deleted all the files and lines of code from using `react-rails`, and changed my Gemfile to have `gem 'webpacker'`, and run `bundle`. The beauty of blogging my past experiences means I can quickly reference how to do a webpacker setup. So I'll run `rails webpacker:install` and then `rails webpacker:install:react`. The one important bit I'm going to include from [my React favorite tutorial](https://x-team.com/blog/get-in-full-stack-shape-with-rails-5-1-webpacker-and-reactjs/) is to use [Foreman](https://github.com/ddollar/foreman). So let's create a new file in the project file (root) called `Procfile.dev` and put in this code:

{% highlight ruby %}
web: bundle exec rails server -p 3000
webpacker: ./bin/webpack-dev-server
{% endhighlight %}

You can leave off the `-p 3000`, and specify it in the terminal at runtime. Run `gem install foreman` so you can run Rails & React simultaneously. Instead of running `rails server`, we'll run `foreman start -f Procfile.dev`. We'll still go to `localhost:3000` in our browser. OK, config complete for me at this point. All is well and good. Let's move fast and break things.

I'm first going to use the default `app/javascript/packs/hello_react.jsx` by adding `= javascript_pack_tag 'hello_react'` to my `app/views/narrations/home.html.haml` file. Running `foreman start -f Procfile.dev` shows "Hello React!" at the bottom of the homepage when I access `localhost:3000`. Let's rename this file to `narrations.jsx`, along with the call in like so `= javascript_pack_tag 'narrations.jsx'` (just to bit a bit verbose in the beginning). Let's take a look at the code in `narrations.jsx` (formerly `hello_react.jsx`):

{% highlight javascript %}
import React from 'react'
import ReactDOM from 'react-dom'
import PropTypes from 'prop-types'

const Hello = props => (
  <div>Hello {props.name}!</div>
)

Hello.defaultProps = {
  name: 'David'
}

Hello.propTypes = {
  name: PropTypes.string
}

document.addEventListener('DOMContentLoaded', () => {
  ReactDOM.render(
    <Hello name="React" />,
    document.body.appendChild(document.createElement('div')),
  )
})
{% endhighlight %}

It looks like we're using "props", which I'm assuming stands for "properties" (though I've never seen this explicitly stated as such). Let's get rid of these "props", as it looks like they're being set from inside this file. We eventually want to pull from our database. Here's my new code barebones:

{% highlight javascript %}
import React from 'react'
import ReactDOM from 'react-dom'

ReactDOM.render(
  <div>
    <ul>
      <li>Life Is Short</li>
      <li>Drugs and the Meaning of Life</li>
      <li>The Power Of A Digital Sabbath</li>
    </ul>
  </div>,
  document.getElementById('narrations')
)
{% endhighlight %}

Standard first two lines to use React. The `ReactDOM.render( element, container )` statement says to render "what" and "where". So my element is just a div with a static unordered list, and "where" is in a div with an id of "narrations". So I added that to `app/views/narrations/home.html.haml`:

{% highlight haml %}
#narrations
= javascript_pack_tag 'narrations'
{% endhighlight %}

So that works, and now let's slowly replace that list by pulling each title from the database. Later we'll add links to the show page. I wish that we could use React directly in our views, so that we didn't have to learn a new convention. For example, instead of having `home.html.haml`, we could have `home.html.jsx`. That way I could directly use `@narrations` set from my controller directly in my React view. Maybe they'll go that way someday.

Anyway, it seems the convention from both the X-Team post, and from Kurt Tomlinson's [React and Ruby on Rails Lessons Learned](https://blog.kurttomlinson.com/posts/react-and-ruby-on-rails-lessons-learned) is to use JSON via JBuilder. Let's give that a whack.

First thing is to namespace our Narrations in an API, running `rails generate controller api/narrations`. This namespacing seems to be a best practice in order to use later as a module of sorts. Now in addition to `app/controllers/narrations_controller.rb`, we have `app/controllers/api/narrations_controller.rb`. Here's the total output:

```
Running via Spring preloader in process 19280
      create  app/controllers/api/narrations_controller.rb
      invoke  erb
      create    app/views/api/narrations
      invoke  test_unit
      create    test/controllers/api/narrations_controller_test.rb
      invoke  helper
      create    app/helpers/api/narrations_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/api/narrations.coffee
      invoke    scss
      create      app/assets/stylesheets/api/narrations.scss
```

I'm only going to keep the controller and view, so I'll delete the test, helper, coffee, and scss files. Let's go into our controller and add a `#show` method.

{% highlight ruby %}
# app/controllers/api/narrations_controller.rb
class Api::NarrationsController < ApplicationController  
  def show
    @narration = Narration.find(params[:id])
  end
end
{% endhighlight %}

Now let's update our `routes.rb` file so that we can have `/narrations/:id` and `api/narrations/:id`. I think the latter will eventually just overtake the former. We do the namespace for the api, which takes json and only allows it for api/narrations/show, essentially.

{% highlight ruby %}
Rails.application.routes.draw do
  root to: 'narrations#home'

  namespace :api, defaults: { format: :json } do
    resources :narrations, only: [ :show ]
  end

  get 'narrations/:id', to: 'narrations#show', as: :narration
  get '/about', to: 'home#about', as: :about
  get 'narrator/:id', to: 'narrators#show', as: :narrator
end
{% endhighlight %}

So going to `api/narrations/:id` should show the narration in question, but we need a view to display. Because it's json, let's create the file, `app/views/api/narrations/show.json.jbuilder`:

{% highlight json %}
json.extract! @narration, :id, :title, :published, :source_url
{% endhighlight %}

Good, navigating to `localhost:3000/api/narrations/:id` gives us the json record from our database.
