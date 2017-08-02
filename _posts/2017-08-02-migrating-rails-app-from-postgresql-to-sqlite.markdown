---
layout: post
title:  "Migrating your Rails App from Postgresql to SQLite"
date:   2017-08-02 13:55:00 -0500
categories: rails react
published: true
---
NOTE: data wasn't migrated in this guide. Instead we changed database from Postgresql to SQLite3.
...

Usually it's the other way around, right? Migrating from the default SQLite database to a more industry standard for production, Postgresql. But I've been going through the free [React on Rails course](https://learnetto.com/course_parts/657) and per usual my app's behavior veers from the demonstrated behavior. I suspect that the difference could be due to my using Postgresql instead of the default SQLite. So let's learn how to migrate a database!

As expected, all the Google results are about migrating the other way, so let's fiddle and observe. I'm going to make a newbie move and just change my Gemfile, run `bundle install`, and then `rails server`. Here's the relevant snippet from my Gemfile and the corresponding error in the browser:

{% highlight ruby %}
# gem 'pg', '~> 0.18'
gem 'sqlite3'
{% endhighlight %}

```
ActiveRecord::ConnectionNotEstablished
No connection pool with 'primary' found.
```

The [Stack Overflow thread I found](https://stackoverflow.com/questions/38176304/no-connection-pool-for-activerecordbase) hints at the `config/database.yml` file. So let's [change that to be more SQLite-y](https://gist.github.com/danopia/940155).

{% highlight yaml %}
development:
  adapter: sqlite3
  database: db/development.sqlite3
  pool: 5
  timeout: 5000
test:
  adapter: sqlite3
  database: db/test.sqlite3
  pool: 5
  timeout: 5000
production:
  adapter: sqlite3
  database: db/production.sqlite3
  pool: 5
  timeout: 5000
{% endhighlight %}

Running `rails server` gives us a new error now:

```
ActiveRecord::PendingMigrationError
Migrations are pending. To resolve this issue, run: bin/rails db:migrate RAILS_ENV=development
```

So let's run `rails db:migrate`:

```
== 20170802165856 CreateAppointments: migrating ===============================
-- create_table(:appointments)
   -> 0.0015s
== 20170802165856 CreateAppointments: migrated (0.0016s) ======================
```

Ok, now running `rails server` gives us the form as expected. Looking forward, I would want to actually migrate the data because at this point it's all gone. That's something I've just learned and should be mindful of when data is crucial.

And after entering data into the form, per section 3.1 of the React on Rails guide, I still get the behavior I observed before. So maybe it wasn't a Postresql database issue. At least we can better appreciate where the database dependencies live: `Gemfile`, `config/database.yml`, and then running `rails db:migrate`.
