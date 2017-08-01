---
layout: post
title:  "Configuring React on Rails via Webpacker and Postgresql"
date:   2017-08-01 10:00:00 -0500
categories: rails react webpacker
published: true
---
**EDIT:** _when creating this walkthrough for the Narify app, I didn't append React as follows: `rails new narify --webpack=react -d postgresql`. I add the React-specific part after updating Yarn._

Let's try using Webpacker to run some React on Rails. And because we prefer Postgresql to SQLite in production, let's run `rails new narify --webpack -d postgresql`, and the following terminal output is the only difference when creating a new app with the `--webpack` flag.

```
  ...
Fetching webpacker 2.0
Installing webpacker 2.0
  ...
  rails  webpacker:install
Webpacker requires Node.js >= v6.4 and you are using v4.4.1
Please upgrade Node.js https://nodejs.org/en/download/
  ...
```

For me, I had to update Node.js first. I followed the instructions at the provided link and then ran `node -v` which gave me `v6.11.1`. NOTE: from an existing app (5.1+) sans webpacker, add `gem 'webpacker', '~> 2.0'` to your Gemfile, run `bundle`, and then `rails webpacker:install`, which gave me:

```
Webpacker requires Yarn version >= 0.20.1. Please download and install the latest version from https://yarnpkg.com/lang/en/docs/install/
```

I then ran `brew install yarn`, which did A LOT of stuff and took a while, but after it was done I verified using `yarn --version` and got `0.27.5`. Ok, back to the new app installed with Webpacker. `rails server` gives us "Yay! You're on Rails!". Good; now let's add the React-specific part to webpacker, `rails webpacker:install:react`:

```
Webpack binstubs not found.
Make sure the bin directory or binstubs are not included in .gitignore
Exiting!
```

I <3 errors. Remember earlier when I had to upgrade Node.js? I did that. We need to run what came immediately before that notification (but failed), `rails webpacker:install`, which takes a while, creates some new folders & files, then gives us "Webpacker successfully installed 🎉 🍰". Here's those bits that were added to our Rails structure:

```
  create  config/webpacker.yml
Copying webpack core config and loaders
  create  config/webpack
  create  config/webpack/configuration.js
  create  config/webpack/development.js
  create  config/webpack/production.js
  create  config/webpack/shared.js
  create  config/webpack/test.js
  create  config/webpack/loaders
  create  config/webpack/loaders/assets.js
  create  config/webpack/loaders/babel.js
  create  config/webpack/loaders/coffee.js
  create  config/webpack/loaders/erb.js
  create  config/webpack/loaders/sass.js
Copying .postcssrc.yml to app root directory
  create  .postcssrc.yml
Copying .babelrc to app root directory
  create  .babelrc
Creating javascript app source directory
  create  app/javascript
  create  app/javascript/packs/application.js
Copying binstubs
  exist  bin
  create  bin/webpack-dev-server
  create  bin/webpack
  append  .gitignore
```

Then it goes on installing all the JavaScript dependencies. Next we'll run `rails webpacker:install:react`, which is funny because you'd think we wouldn't have gotten that error message above running this in the first place. Hmm.. anywho, you should finally get "Webpacker now supports react.js 🎉". Here's the bits that were added from `rails webpacker:install:react`:

```
Using .../config/webpacker.yml file for setting up webpack paths
Copying react preset to your .babelrc file
Copying react loader to config/webpack/loaders
      create  config/webpack/loaders/react.js
Copying react example entry file to .../app/javascript/packs
      create  app/javascript/packs/hello_react.jsx
```

Then it goes on installing the React dependencies, etc. So we have more config & bin stuff, a `javascript/packs/application.js` folders/file outside of the `assets` folder, and a `hello_react.jsx` sample file. Yay! Open a new terminal tab and run `./bin/webpack-dev-server`, and in your original tab run `rails server`. This still gives us the welcome to rails page, so let's open `app/javascript/packs/hello_react.jsx` and look for a hint:

{% highlight javascript %}
// Run this example by adding <%= javascript_pack_tag 'hello_react' %> to the head of your layout file,
// like app/views/layouts/application.html.erb. All it does is render <div>Hello React</div> at the bottom
// of the page.

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

Okie dokie, so let's add that `= javascript_pack_tag` in `app/views/layouts/application.html.haml`, like so:

{% highlight haml %}
!!!
%html
  %head
    %title Narify
    = csrf_meta_tags
    = stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload'
    = javascript_include_tag 'application', 'data-turbolinks-track': 'reload'
    = javascript_pack_tag    'hello_react'
  %body
    = yield
{% endhighlight %}

And `rails server` still only renders the welcome page (not the div at the bottom of the page, as promised). I think maybe we need to generate our first controller and view. `rails generate controller Narrations index` and change `routes.rb` code to `root to: 'narrations#index'`. Now localhost:3000 should show the generated page with "Hello React!" below therein. I'll post more on React & Rails using Webpacker as I continue using it.

I used the [webpacker GitHub readme](https://github.com/rails/webpacker) as a guide for this configuration process.
