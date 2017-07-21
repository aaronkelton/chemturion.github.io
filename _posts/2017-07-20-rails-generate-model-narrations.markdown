---
layout: post
title:  "Generating a Rails Model the Hard Way"
date:   2017-07-20 17:00:00 -0500
categories: rails beginner model
published: true
---
So far we've created a rails app from scratch using the generator command as a guide. At this point I can navigate to `localhost:3000`, which will `root to:` my `narrations#home` controller action. This is effectively my home page. Ideally we'd have different views based on if the user is logged into to site, and what roles that user has. For now, I either want to develop the model with a database so I can seed the database with some dummy data for the user to peruse. This would consist of narration cards. The user would click on a card and be taken to the card's player page. I'm torn between going this route, or delving into the design of the website. I think it may be best to avoid the styling until later on so that I can treat the user (at this point, me) as super user, just able to do everything. And then I'll restrict the access and features of the web app. So when the user goes to `narify.com`, they'll just see a list of narrations. I'll need a model, so let's go this route. Using my dummy app I'll run `rails generate model Narration` (singular). Here's what I get:

```
Running via Spring preloader in process 8484
      invoke  active_record
      create    db/migrate/20170720213340_create_narrations.rb
      create    app/models/narration.rb
      invoke    test_unit
      create      test/models/narration_test.rb
      create      test/fixtures/narrations.yml
```

Straight away, I don't think I'll mess with any tests. I can barely wrap my mind around the little I've done so far, so let's only create the migration and model file. So I'll create a `db/migrate` folder in the root path (is that how you say it?) and a `models` folder in the `app` directory; plus these `.rb` files. I don't think I even need the `seeds.rb` file until I'm ready to create seed data. Now, I could run `rails generate migration CreateNarrations` so it'll create that handy timestamped filename, but I'm electing to create it manually since that generator only creates a single file. I'll use the convention to create something similar like `2017072013340_create_narrations.rb`. I'll leave the file as it was generated, i.e. not adding extra fields. I'll create a separate migration for that. Barebones table creation.

{% highlight ruby %}
class CreateNarrations < ActiveRecord::Migration[5.1]
  def change
    create_table :narrations do |t|
      t.timestamps
    end
  end
end
{% endhighlight %}

The `models` folder also has a nested `concerns/.keep` folder/file and an `application_record.rb` file. I'm going to ignore the former and only add the latter when I encounter an error. So just for now, let's add the `narration.rb` file.

{% highlight ruby %}
class Narration < ApplicationRecord
end
{% endhighlight %}

I've only created some files, so I should still be able to run `rails server` without problems, right? Unfortunately, when I tried, I encountered an error in my browser:

```
ActiveRecord::PendingMigrationError
Migrations are pending. To resolve this issue, run: bin/rails db:migrate RAILS_ENV=development
```

So let's run `rails db:migrate` to effectively tell the database to create the Narrations table. I'm not going to worry about the `RAILS_ENV=development` option at this point. Should I worry? I don't know, so I'll put that off until later. So what'd we get?

```
rails aborted!
No Rakefile found (looking for: rakefile, Rakefile, rakefile.rb, Rakefile.rb)
/Users/~/Sites/narify/bin/rails:9:in `require'
/Users/~/Sites/narify/bin/rails:9:in `<top (required)>'
/Users/~/Sites/narify/bin/spring:15:in `<top (required)>'
bin/rails:3:in `load'
bin/rails:3:in `<main>'
(See full trace by running task with --trace)
```

So let's create that `Rakefile` in the root directory and populate it with the following (with the verbose `.rb`) and run `rails db:migrate` again:

{% highlight ruby %}
# Add your own tasks in files placed in lib/tasks ending in .rake,
# for example lib/tasks/capistrano.rake, and they will automatically be available to Rake.

require_relative 'config/application.rb'

Rails.application.load_tasks
{% endhighlight %}

AND... because we're using Postgres, we get a fancy error that we probably wouldn't get otherwise:

```
== 20170720214453 CreateNarrations: migrating =================================
-- create_table(:narrations)
rails aborted!
StandardError: An error has occurred, this and all later migrations canceled:

PG::DuplicateTable: ERROR:  relation "narrations" already exists
: CREATE TABLE "narrations" ("id" bigserial primary key, "created_at" timestamp NOT NULL, "updated_at" timestamp NOT NULL)
/Users/kelton/Sites/narify/db/migrate/20170720214453_create_narrations.rb:3:in `change'
/Users/kelton/Sites/narify/bin/rails:9:in `require'
/Users/kelton/Sites/narify/bin/rails:9:in `<top (required)>'
/Users/kelton/Sites/narify/bin/spring:15:in `<top (required)>'
bin/rails:3:in `load'
bin/rails:3:in `<main>'
ActiveRecord::StatementInvalid: PG::DuplicateTable: ERROR:  relation "narrations" already exists
: CREATE TABLE "narrations" ("id" bigserial primary key, "created_at" timestamp NOT NULL, "updated_at" timestamp NOT NULL)
/Users/kelton/Sites/narify/db/migrate/20170720214453_create_narrations.rb:3:in `change'
/Users/kelton/Sites/narify/bin/rails:9:in `require'
/Users/kelton/Sites/narify/bin/rails:9:in `<top (required)>'
/Users/kelton/Sites/narify/bin/spring:15:in `<top (required)>'
bin/rails:3:in `load'
bin/rails:3:in `<main>'
PG::DuplicateTable: ERROR:  relation "narrations" already exists
/Users/kelton/Sites/narify/db/migrate/20170720214453_create_narrations.rb:3:in `change'
/Users/kelton/Sites/narify/bin/rails:9:in `require'
/Users/kelton/Sites/narify/bin/rails:9:in `<top (required)>'
/Users/kelton/Sites/narify/bin/spring:15:in `<top (required)>'
bin/rails:3:in `load'
bin/rails:3:in `<main>'
Tasks: TOP => db:migrate
(See full trace by running task with --trace)
```

So that's weird, right? Looks like it tried to run the migration three times! I think this is because I once tried to create a Postgres-based Rails app in this directory, or something. Time to google "PG::DuplicateTable: ERROR: relation already exists". Seems that there's already a database, so I need to run `rails db:drop`:

```
Dropped database 'narify_development'
Dropped database 'narify_test'
```

And now to recreate the database, `rails db:create`:

```
Created database 'narify_development'
Created database 'narify_test'
```

Alas, now we can run `rails db:migrate`:

```
== 20170720214453 CreateNarrations: migrating =================================
-- create_table(:narrations)
   -> 0.0290s
== 20170720214453 CreateNarrations: migrated (0.0293s) ========================
```

Eureka! Now I notice that a new file has been created as a result of the migration: `db/schema.rb`:

{% highlight ruby %}
# This file is auto-generated from the current state of the database. Instead
# of editing this file, please use the migrations feature of Active Record to
# incrementally modify your database, and then regenerate this schema definition.
#
# Note that this schema.rb definition is the authoritative source for your
# database schema. If you need to create the application database on another
# system, you should be using db:schema:load, not running all the migrations
# from scratch. The latter is a flawed and unsustainable approach (the more migrations
# you'll amass, the slower it'll run and the greater likelihood for issues).
#
# It's strongly recommended that you check this file into your version control system.

ActiveRecord::Schema.define(version: 20170720214453) do

  # These are extensions that must be enabled in order to support this database
  enable_extension "plpgsql"

  create_table "narrations", force: :cascade do |t|
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

end
{% endhighlight %}

Let's add another migration file to create some new fields. A narration needs a title at a minimum, so let's do that and no more. My new file will be `20170720224547_add_title_to_narrations.rb`:

{% highlight ruby %}
class AddTitleToNarrations < ActiveRecord::Migration[5.1]
  def change
    add_column :narrations, :title, :string
  end
end
{% endhighlight %}

Let's run `rails db:migrate` once more and check out the change in `schema.rb` sans prefixed comments:

```
== 20170720224547 AddTitleToNarrations: migrating =============================
-- add_column(:narrations, :title, :string)
   -> 0.0303s
== 20170720224547 AddTitleToNarrations: migrated (0.0304s) ====================
```

{% highlight ruby %}
ActiveRecord::Schema.define(version: 20170720224547) do

  # These are extensions that must be enabled in order to support this database
  enable_extension "plpgsql"

  create_table "narrations", force: :cascade do |t|
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.string "title"
  end

end
{% endhighlight %}

I think we're setup minimally at this point, so let's create some seed data to populate our database. Using the generated `seeds.rb` file as a reference, I'll put the title of two of my favorite articles into my newly created `db/seeds.rb` file (one by Paul Graham; the other by Sam Harris):

{% highlight ruby %}
# This file should contain all the record creation needed to seed the database with its default values.
# The data can then be loaded with the rails db:seed command (or created alongside the database with db:setup).
#
# Examples:
#
#   movies = Movie.create([{ name: 'Star Wars' }, { name: 'Lord of the Rings' }])
#   Character.create(name: 'Luke', movie: movies.first)

n1 = Narration.create(title: "Life Is Short")

n2 = Narration.create(title: "Drugs and the Meaning of Life")
{% endhighlight %}

Okay, we're ready to seed the database, presumably. Let's run `rails db:seed` and check out my brand new error! :)

```
rails aborted!
NameError: uninitialized constant ApplicationRecord
/Users/~/Sites/narify/app/models/narration.rb:1:in `<top (required)>'
/Users/~/Sites/narify/db/seeds.rb:9:in `<top (required)>'
/Users/~/Sites/narify/bin/rails:9:in `require'
/Users/~/Sites/narify/bin/rails:9:in `<top (required)>'
/Users/~/Sites/narify/bin/spring:15:in `<top (required)>'
bin/rails:3:in `load'
bin/rails:3:in `<main>'
Tasks: TOP => db:seed
(See full trace by running task with --trace)
```

I'm really only interested in the `NameError: uninitialized constant ApplicationRecord` line. Well, that, and the second line: `/Users/~/Sites/narify/app/models/narration.rb:1:in '<top (required)>'`, which reminds me of the missing neighbor file, `app/models/application_record.rb`. Let's add that straight from my generated dummy app.

{% highlight ruby %}
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
end
{% endhighlight %}

Success! Or at least I'm trusting it was a success. No errors, but no success indicator either. At this point, I'd like to visually confirm the seed data. Enter Postgresql. I'm going to run `psql --dbname=narify_development`:

```
psql (9.6.3)
Type "help" for help.
```

New let's `select * from narrations;`, which returns:

```
 id |         created_at         |         updated_at         |             title
----+----------------------------+----------------------------+-------------------------------
  1 | 2017-07-21 17:43:01.822212 | 2017-07-21 17:43:01.822212 | Life Is Short
  2 | 2017-07-21 17:43:01.835691 | 2017-07-21 17:43:01.835691 | Drugs and the Meaning of Life
```

I'll take that as a "Yes", we have data in our database, namely in the narrations table. We ultimately want to have an unordered list of narrations on our homepage, so we need some variable to hold all of the narrations, and this variable needs to be accessible when we `root to:` our `narrations#home`. Why not create an instance variable within the `#home` method called `@narrations` and assign it the value of `Narration.all`, Narration being the corresponding model. Then we'll be able to do our `@narrations.each` stuff in the view.

So, inside our `narrations_controller.rb` file, let's add that code. Here's what it looks like for me:

{% highlight ruby %}
class NarrationsController < ApplicationController
  def home
    @narrations = Narration.all
  end
end
{% endhighlight %}

So only now should we be able to use the `@narrations` instance variable because we've sort of evoked an instance of the `NarrationsController` via the browser. Let's do that in our `app/views/narrations/home.html.haml` file:

{% highlight haml %}
- @narrations.each do |n|
  %ul
    %li
      = n.title
{% endhighlight %}

Holy smokes! Iteratively, and without much bloat (that I know of), we've got ourselves a list from the database.

Where to next? I think I'll hold off with the new, create, update, and delete methods you see in so many Rails tutorials. I think I'll next create the `#show` method so that I can click on one of my list items and open it's respective page. So that's it for now.
