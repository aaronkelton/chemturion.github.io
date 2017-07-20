---
layout: post
title:  "Create a New Rails App the Hard Way"
date:   2017-07-19 18:00:00 -0500
categories: rails beginner
published: true
---
As a new Rails developer, I found myself getting confused and overwhelmed by the obfuscation and generator magic from Rails. So I've decided it will be best for my understanding to create a new Rails app from scratch. No generators and no git.

The first step is to create the local root directory on my local machine: `~/Sites/narify`.

Since I've never published an app to a production server, I only know how to develop locally and run a localhost. So beginning with the end in mind, I ran `rails server` from my terminal. Here's what I got:

```
Usage:
  rails new APP_PATH [options]

Options:
  -r, [--ruby=PATH]                                      # Path to the Ruby binary of your choice
                                                         # Default: /Users/~/.rbenv/versions/2.4.1/bin/ruby
  -m, [--template=TEMPLATE]                              # Path to some application template (can be a filesystem path or URL)
  -d, [--database=DATABASE]                              # Preconfigure for selected database (options: mysql/postgresql/sqlite3/oracle/frontbase/ibm_db/sqlserver/jdbcmysql/jdbcsqlite3/jdbcpostgresql/jdbc)
                                                         # Default: sqlite3
      [--webpack=WEBPACK]                                # Preconfigure for app-like JavaScript with Webpack (options: react/vue/angular)
      [--skip-yarn], [--no-skip-yarn]                    # Don't use Yarn for managing JavaScript dependencies
      [--skip-gemfile], [--no-skip-gemfile]              # Don't create a Gemfile
  -G, [--skip-git], [--no-skip-git]                      # Skip .gitignore file
      [--skip-keeps], [--no-skip-keeps]                  # Skip source control .keep files
  -M, [--skip-action-mailer], [--no-skip-action-mailer]  # Skip Action Mailer files
  -O, [--skip-active-record], [--no-skip-active-record]  # Skip Active Record files
  -P, [--skip-puma], [--no-skip-puma]                    # Skip Puma related files
  -C, [--skip-action-cable], [--no-skip-action-cable]    # Skip Action Cable files
  -S, [--skip-sprockets], [--no-skip-sprockets]          # Skip Sprockets files
      [--skip-spring], [--no-skip-spring]                # Don't install Spring application preloader
      [--skip-listen], [--no-skip-listen]                # Don't generate configuration that depends on the listen gem
      [--skip-coffee], [--no-skip-coffee]                # Don't use CoffeeScript
  -J, [--skip-javascript], [--no-skip-javascript]        # Skip JavaScript files
      [--skip-turbolinks], [--no-skip-turbolinks]        # Skip turbolinks gem
  -T, [--skip-test], [--no-skip-test]                    # Skip test files
      [--skip-system-test], [--no-skip-system-test]      # Skip system test files
      [--dev], [--no-dev]                                # Setup the application with Gemfile pointing to your Rails checkout
      [--edge], [--no-edge]                              # Setup the application with Gemfile pointing to Rails repository
      [--rc=RC]                                          # Path to file containing extra configuration options for rails command
      [--no-rc], [--no-no-rc]                            # Skip loading of extra configuration options from .railsrc file
      [--api], [--no-api]                                # Preconfigure smaller stack for API only apps
  -B, [--skip-bundle], [--no-skip-bundle]                # Don't run bundle install

Runtime options:
  -f, [--force]                    # Overwrite files that already exist
  -p, [--pretend], [--no-pretend]  # Run but do not make any changes
  -q, [--quiet], [--no-quiet]      # Suppress status output
  -s, [--skip], [--no-skip]        # Skip files that already exist

Rails options:
  -h, [--help], [--no-help]        # Show this help message and quit
  -v, [--version], [--no-version]  # Show Rails version number and quit

Description:
    The 'rails new' command creates a new Rails application with a default
    directory structure and configuration at the path you specify.

    You can specify extra command-line arguments to be used every time
    'rails new' runs in the .railsrc configuration file in your home directory.

    Note that the arguments specified in the .railsrc file don't affect the
    defaults values shown above in this help message.

Example:
    rails new ~/Code/Ruby/weblog

    This generates a skeletal Rails installation in ~/Code/Ruby/weblog.
```

Instead of running `rails new narify --database=postgresql`, I'm going to generate everything from scratch. In order to **not** set myself up for failure, I'm going to create a dummy app with the aforementioned statement to get started in another directory. And then I'll use this directory as a guide so I can fully wrap my mind around what is produced. Here's the source code for the `rails new` command. <https://github.com/rails/rails/blob/master/railties/lib/rails/generators/rails/app/app_generator.rb>

```
create
      create  README.md
      create  Rakefile
      create  config.ru
      create  .gitignore
      create  Gemfile
         run  git init from "."
Initialized empty Git repository in /Users/~/Sites/narify/.git/
      create  app
      create  app/assets/config/manifest.js
      create  app/assets/javascripts/application.js
      create  app/assets/javascripts/cable.js
      create  app/assets/stylesheets/application.css
      create  app/channels/application_cable/channel.rb
      create  app/channels/application_cable/connection.rb
      create  app/controllers/application_controller.rb
      create  app/helpers/application_helper.rb
      create  app/jobs/application_job.rb
      create  app/mailers/application_mailer.rb
      create  app/models/application_record.rb
      create  app/views/layouts/application.html.erb
      create  app/views/layouts/mailer.html.erb
      create  app/views/layouts/mailer.text.erb
      create  app/assets/images/.keep
      create  app/assets/javascripts/channels
      create  app/assets/javascripts/channels/.keep
      create  app/controllers/concerns/.keep
      create  app/models/concerns/.keep
      create  bin
      create  bin/bundle
      create  bin/rails
      create  bin/rake
      create  bin/setup
      create  bin/update
      create  bin/yarn
      create  config
      create  config/routes.rb
      create  config/application.rb
      create  config/environment.rb
      create  config/secrets.yml
      create  config/cable.yml
      create  config/puma.rb
      create  config/spring.rb
      create  config/environments
      create  config/environments/development.rb
      create  config/environments/production.rb
      create  config/environments/test.rb
      create  config/initializers
      create  config/initializers/application_controller_renderer.rb
      create  config/initializers/assets.rb
      create  config/initializers/backtrace_silencers.rb
      create  config/initializers/cookies_serializer.rb
      create  config/initializers/cors.rb
      create  config/initializers/filter_parameter_logging.rb
      create  config/initializers/inflections.rb
      create  config/initializers/mime_types.rb
      create  config/initializers/new_framework_defaults_5_1.rb
      create  config/initializers/wrap_parameters.rb
      create  config/locales
      create  config/locales/en.yml
      create  config/boot.rb
      create  config/database.yml
      create  db
      create  db/seeds.rb
      create  lib
      create  lib/tasks
      create  lib/tasks/.keep
      create  lib/assets
      create  lib/assets/.keep
      create  log
      create  log/.keep
      create  public
      create  public/404.html
      create  public/422.html
      create  public/500.html
      create  public/apple-touch-icon-precomposed.png
      create  public/apple-touch-icon.png
      create  public/favicon.ico
      create  public/robots.txt
      create  test/fixtures
      create  test/fixtures/.keep
      create  test/fixtures/files
      create  test/fixtures/files/.keep
      create  test/controllers
      create  test/controllers/.keep
      create  test/mailers
      create  test/mailers/.keep
      create  test/models
      create  test/models/.keep
      create  test/helpers
      create  test/helpers/.keep
      create  test/integration
      create  test/integration/.keep
      create  test/test_helper.rb
      create  test/system
      create  test/system/.keep
      create  test/application_system_test_case.rb
      create  tmp
      create  tmp/.keep
      create  tmp/cache
      create  tmp/cache/assets
      create  vendor
      create  vendor/.keep
      create  package.json
      remove  config/initializers/cors.rb
      remove  config/initializers/new_framework_defaults_5_1.rb
         run  bundle install
The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x86-mswin32, x64-mingw32, java. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x86-mswin32 x64-mingw32 java`.
Fetching gem metadata from https://rubygems.org/.........
Fetching version metadata from https://rubygems.org/..
Fetching dependency metadata from https://rubygems.org/.
Resolving dependencies..................................................................................................................................
Using rake 12.0.0
Using concurrent-ruby 1.0.5
Fetching i18n 0.8.6
Installing i18n 0.8.6
Using minitest 5.10.2
Using thread_safe 0.3.6
Using builder 3.2.3
Fetching erubi 1.6.1
Installing erubi 1.6.1
Using mini_portile2 2.2.0
Using rack 2.0.3
Using nio4r 2.1.0
Using websocket-extensions 0.1.2
Using mime-types-data 3.2016.0521
Using arel 8.0.0
Using bundler 1.15.1
Using method_source 0.8.2
Using thor 0.19.4
Fetching pg 0.21.0
Installing pg 0.21.0 with native extensions
Using puma 3.9.1
Fetching rb-fsevent 0.10.2
Installing rb-fsevent 0.10.2
Using ffi 1.9.18
Using tilt 2.0.7
Using execjs 2.7.0
Using coffee-script-source 1.12.2
Using turbolinks-source 5.0.3
Using multi_json 1.12.1
Using byebug 9.0.6
Using public_suffix 2.0.5
Using rubyzip 1.2.1
Using bindex 0.5.0
Using ruby_dep 1.5.0
Using tzinfo 1.2.3
Using nokogiri 1.8.0
Using rack-test 0.6.3
Using sprockets 3.7.1
Using websocket-driver 0.6.5
Using mime-types 3.1
Using rb-inotify 0.9.10
Fetching childprocess 0.7.1
Installing childprocess 0.7.1
Using uglifier 3.2.0
Using coffee-script 2.4.1
Using turbolinks 5.0.1
Using addressable 2.5.1
Fetching activesupport 5.1.2
Installing activesupport 5.1.2
Using loofah 2.0.3
Using xpath 2.1.0
Using mail 2.6.6
Fetching sass-listen 4.0.0
Installing sass-listen 4.0.0
Using listen 3.1.5
Fetching selenium-webdriver 3.4.4
Installing selenium-webdriver 3.4.4
Using rails-dom-testing 2.0.3
Using globalid 0.4.0
Fetching activemodel 5.1.2
Installing activemodel 5.1.2
Using jbuilder 2.7.0
Using spring 2.0.2
Using rails-html-sanitizer 1.0.3
Fetching capybara 2.14.4
Installing capybara 2.14.4
Fetching sass 3.5.1
Installing sass 3.5.1
Fetching activejob 5.1.2
Installing activejob 5.1.2
Fetching activerecord 5.1.2
Installing activerecord 5.1.2
Using spring-watcher-listen 2.0.1
Fetching actionview 5.1.2
Installing actionview 5.1.2
Fetching actionpack 5.1.2
Installing actionpack 5.1.2
Fetching actioncable 5.1.2
Installing actioncable 5.1.2
Fetching actionmailer 5.1.2
Installing actionmailer 5.1.2
Fetching railties 5.1.2
Installing railties 5.1.2
Using sprockets-rails 3.2.0
Using coffee-rails 4.2.2
Using web-console 3.5.1
Fetching rails 5.1.2
Installing rails 5.1.2
Using sass-rails 5.0.6
Bundle complete! 16 Gemfile dependencies, 70 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
The latest bundler is 1.15.2, but you are currently running 1.15.1.
To update, run `gem install bundler`
         run  bundle exec spring binstub --all
* bin/rake: spring inserted
* bin/rails: spring inserted
```

It appears that the command has three main parts:
1. Create the app configuration files like the Readme, Rakefile, config.ru, and initialize the local git repo.
2. Create the directories and files in alphabetical order.
3. Run `bundle install`, which I guess populates the previously generated Gemfile

So now as I attempt to rebuild this app from scratch, my question is, "What is the bare minimum I need to run `rails server`?" So to find out, I'll concurrently remove elements from the sample app.
