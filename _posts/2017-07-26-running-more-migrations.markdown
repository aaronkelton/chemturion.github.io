---
layout: post
title:  "Rails Migration published:boolean, and Duplicate Records"
date:   2017-07-26 13:14:00 -0500
categories: rails migration
published: true
---
I almost decided to implement bootstrap-sass today, but decided against it in favor of more functionality. Today I'm going to make my Narration model more robust. At present it's just a title and some meta data. This process is an exercise in preparing to develop more models and the relationships between them. It's been about 5 years since I was neck deep in database design, but I think I remember the gist to be dangerous enough. Let's see what `rails generate migration` does without arguments:

```
Running via Spring preloader in process 4959
Usage:
  rails generate migration NAME [field[:type][:index] field[:type][:index]] [options]

Options:
      [--skip-namespace], [--no-skip-namespace]  # Skip namespace (affects only isolated applications)
  -o, --orm=NAME                                 # ORM to be invoked
                                                 # Default: active_record

ActiveRecord options:
  [--primary-key-type=PRIMARY_KEY_TYPE]  # The type for primary key

Runtime options:
  -f, [--force]                    # Overwrite files that already exist
  -p, [--pretend], [--no-pretend]  # Run but do not make any changes
  -q, [--quiet], [--no-quiet]      # Suppress status output
  -s, [--skip], [--no-skip]        # Skip files that already exist

Description:
    Stubs out a new database migration. Pass the migration name, either
    CamelCased or under_scored, and an optional list of attribute pairs as arguments.

    A migration class is generated in db/migrate prefixed by a timestamp of the current date and time.

    You can name your migration in either of these formats to generate add/remove
    column lines from supplied attributes: AddColumnsToTable or RemoveColumnsFromTable

Example:
    `rails generate migration AddSslFlag`

    If the current date is May 14, 2008 and the current time 09:09:12, this creates the AddSslFlag migration
    db/migrate/20080514090912_add_ssl_flag.rb

    `rails generate migration AddTitleBodyToPost title:string body:text published:boolean`

    This will create the AddTitleBodyToPost in db/migrate/20080514090912_add_title_body_to_post.rb with this in the Change migration:

      add_column :posts, :title, :string
      add_column :posts, :body, :text
      add_column :posts, :published, :boolean

Migration names containing JoinTable will generate join tables for use with
has_and_belongs_to_many associations.

Example:
    `rails g migration CreateMediaJoinTable artists musics:uniq`

    will create the migration

    create_join_table :artists, :musics do |t|
      # t.index [:artist_id, :music_id]
      t.index [:music_id, :artist_id], unique: true
    end
```

So I'll eventually want to do a JoinTable migration, but not right this moment. Let's take it slow and just [add a simple column](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_column), similar to a Jekyll post's front matter, we'll add a `published` column to the database. Let's see if this works: `rails generate migration AddPublishedToNarrations published:boolean` and the generated migration file:

```
Running via Spring preloader in process 8565
      invoke  active_record
      create    db/migrate/20170726153200_add_published_to_narrations.rb
```

{% highlight ruby %}
class AddPublishedToNarrations < ActiveRecord::Migration[5.1]
  def change
    add_column :narrations, :published, :boolean
  end
end
{% endhighlight %}

Now we run `rails db:migrate`:

```
== 20170726153200 AddPublishedToNarrations: migrating =========================
-- add_column(:narrations, :published, :boolean)
   -> 0.0357s
== 20170726153200 AddPublishedToNarrations: migrated (0.0358s) ================
```

I didn't set any default value for new records (on purpose), and my existing records have published values of `nil`. Let's update that in our `seeds.rb` file.

{% highlight ruby %}
n1 = Narration.create(title: "Life Is Short", published: true)
n2 = Narration.create(title: "Drugs and the Meaning of Life", published: true)
n3 = Narration.create(title: "The Power Of A Digital Sabbath", published: false)
{% endhighlight %}

I went ahead and added another record; this one from Feld Thoughts. It should be published, but for illustration let's leave it `false` for now. Let's run `rails db:seed` and then `rails db` to then run `select * from narrations;` and here's what we get:

```
 id |         created_at         |         updated_at         |             title              | published
----+----------------------------+----------------------------+--------------------------------+-----------
  1 | 2017-07-21 17:27:12.603507 | 2017-07-21 17:27:12.603507 | Life Is Short                  |
  2 | 2017-07-21 17:27:12.682051 | 2017-07-21 17:27:12.682051 | Drugs and the Meaning of Life  |
  3 | 2017-07-26 15:48:53.383737 | 2017-07-26 15:48:53.383737 | Life Is Short                  | t
  4 | 2017-07-26 15:48:53.40201  | 2017-07-26 15:48:53.40201  | Drugs and the Meaning of Life  | t
  5 | 2017-07-26 15:48:53.404381 | 2017-07-26 15:48:53.404381 | The Power Of A Digital Sabbath | f
(5 rows)
```

![rick perry oops](http://3.bp.blogspot.com/-O9dIDbReWcA/Ucygtl-j26I/AAAAAAAAFLI/KRhDWkJ6YJU/s902/rick_perry_oops_2.jpeg)

Well, this is fun, isn't it? Let's update our `seeds.rb` file to be a bit more robust using [find_or_create_by](http://api.rubyonrails.org/classes/ActiveRecord/Relation.html#method-i-find_or_create_by):

{% highlight ruby %}
[
  {title: 'Life Is Short', published: true},
  {title: 'Drugs and the Meaning of Life', published: true},
  {title: 'The Power Of A Digital Sabbath', published: false}
].each do |rec|
  Narration.find_or_create_by(title: rec[:title], published: rec[:published])
end
{% endhighlight %}

We'll deal with the first two records later. For now, `nil` is "falsey", so let's update our view to only show `published: true` records.

{% highlight haml %}
- @narrations.each do |narration|
  %ul
    - if narration.published
      %li
        = narration.title
        = link_to 'Listen', narration_path(narration)
{% endhighlight %}

I can see how if some publisher or author decided they didn't want anyone listening to a narrated article of theirs, or if the narration for some reason slipped thru our quality checks, we could quickly go in from an admin portal and change the published value to "False". Another field we can add to our narrations table is the source URL. Let's repeat the above process, and hopefully our new `seeds.rb` method won't create duplicate records.

Let's generate a migration using `rails generate migration AddSourceURLToNarrations source_url:string`. I know string type is the default, but I prefer to be verbose when starting. Also, it helps beginners while "senior" folks can ignore the verbosity.

```
Running via Spring preloader in process 14500
      invoke  active_record
      create    db/migrate/20170726165805_add_source_url_to_narrations.rb
```

{% highlight ruby %}
class AddSourceUrlToNarrations < ActiveRecord::Migration[5.1]
  def change
    add_column :narrations, :source_url, :string
  end
end
{% endhighlight %}

Now let's migrate this generated migration with `rails db:migrate`:

```
== 20170726165805 AddSourceUrlToNarrations: migrating =========================
-- add_column(:narrations, :source_url, :string)
   -> 0.0785s
== 20170726165805 AddSourceUrlToNarrations: migrated (0.0788s) ================
```

Alright, now to give these narrations their respective source URLs. Let's update `seeds.rb`:

{% highlight ruby %}
[
  {title: 'Life Is Short',                  published: true,  source_url: 'http://www.paulgraham.com/vb.html'},
  {title: 'Drugs and the Meaning of Life',  published: true,  source_url: 'https://www.samharris.org/podcast/item/drugs-and-the-meaning-of-life'},
  {title: 'The Power Of A Digital Sabbath', published: false, source_url: 'https://feld.com/archives/2017/02/power-digital-sabbath.html'}
].each do |rec|
  Narration.find_or_create_by(title: rec[:title], published: rec[:published], source_url: rec[:source_url])
end
{% endhighlight %}

Crossing my fingers, let's `rails db:seed` this bad boy and `rails db` plus `select * from narrations;`:

```
 id |         created_at         |         updated_at         |             title              | published |                              source_url
----+----------------------------+----------------------------+--------------------------------+-----------+----------------------------------------------------------------------
  1 | 2017-07-21 17:27:12.603507 | 2017-07-21 17:27:12.603507 | Life Is Short                  |           |
  2 | 2017-07-21 17:27:12.682051 | 2017-07-21 17:27:12.682051 | Drugs and the Meaning of Life  |           |
  3 | 2017-07-26 15:48:53.383737 | 2017-07-26 15:48:53.383737 | Life Is Short                  | t         |
  4 | 2017-07-26 15:48:53.40201  | 2017-07-26 15:48:53.40201  | Drugs and the Meaning of Life  | t         |
  5 | 2017-07-26 15:48:53.404381 | 2017-07-26 15:48:53.404381 | The Power Of A Digital Sabbath | f         |
  6 | 2017-07-26 17:11:33.044511 | 2017-07-26 17:11:33.044511 | Life Is Short                  | t         | http://www.paulgraham.com/vb.html
  7 | 2017-07-26 17:11:33.086072 | 2017-07-26 17:11:33.086072 | Drugs and the Meaning of Life  | t         | https://www.samharris.org/podcast/item/drugs-and-the-meaning-of-life
  8 | 2017-07-26 17:11:33.096889 | 2017-07-26 17:11:33.096889 | The Power Of A Digital Sabbath | f         | https://feld.com/archives/2017/02/power-digital-sabbath.html
(8 rows)
```

Well, fiddlefucks. At least I know what my next post is going to be about. Before we deal with this data dilemma, let's create a dynamic link from the narration show page to view the source article, preferably in a new tab.

{% highlight haml %}
= @narration.title
= link_to 'View original article', @narration.source_url, target: '_blank' if !!@narration.source_url
= link_to 'Back to Narrations', root_path
{% endhighlight %}

The `source_url` attribute should probably have a "not nil" qualifier on it, but for now the double bang will prevent links for "nilfull" source URL records. Now once a user clicks to listen to a narration, they'll have the option to open the source in a new tab.
