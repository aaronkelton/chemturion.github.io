---
layout: post
title:  "Prevent Duplicate Records when Updating seeds.rb"
date:   2017-07-26 15:14:00 -0500
categories: rails database migration
published: true
---
In my last post, I inadvertently created duplicate records every time I updated my `seeds.rb` file after running `rails db:migrate` to add a column to my `narrations` table. Here's my basic setup at this point:

```
 id |             title              | published |                              source_url
----+--------------------------------+-----------+----------------------------------------------------------------------
  1 | Life Is Short                  |           |
  2 | Drugs and the Meaning of Life  |           |
  3 | Life Is Short                  | t         |
  4 | Drugs and the Meaning of Life  | t         |
  5 | The Power Of A Digital Sabbath | f         |
  6 | Life Is Short                  | t         | http://www.paulgraham.com/vb.html
  7 | Drugs and the Meaning of Life  | t         | https://www.samharris.org/podcast/item/drugs-and-the-meaning-of-life
  8 | The Power Of A Digital Sabbath | f         | https://feld.com/archives/2017/02/power-digital-sabbath.html
```

Since we're early on, it's probably easiest to just drop the table and recreate the table with our most recent data. Effectively, ids 6, 7, and 8 would become 1, 2, and 3 in the new database table. But going forward, I need to know the best approach for managing `seeds.rb`. So let's figure that out first and then drop/recreate the table. Here's my `seeds.rb` file as it stands:

{% highlight ruby %}
[
  {title: 'Life Is Short',                  published: true,  source_url: 'http://www.paulgraham.com/vb.html'},
  {title: 'Drugs and the Meaning of Life',  published: true,  source_url: 'https://www.samharris.org/podcast/item/drugs-and-the-meaning-of-life'},
  {title: 'The Power Of A Digital Sabbath', published: false, source_url: 'https://feld.com/archives/2017/02/power-digital-sabbath.html'}
].each do |rec|
  Narration.find_or_create_by(title: rec[:title], published: rec[:published], source_url: rec[:source_url])
end
{% endhighlight %}

Suppose we want to provide a preview of the article's text content, so we add a new column called `snippet`. If we add `snippet: 'some string'` to our hash array, and then append `snippet: rec[:snippet]` to our `#find_or_create_by` method, Rails would never find a record with all 4 attributes, so it would create 4 new records. It's as if we really need a `#find_then_update_or_create` method instead.

***

I'm typing this sentence a couple hours after diving into this problem. It appears it would become quite complicated to abstract away this logic into a dynamic method, so let's table that approach (_too soon?_) and simply drop the table from our database. From [some googling](https://stackoverflow.com/questions/4116067/purge-or-recreate-a-ruby-on-rails-database), let's try `rails db:schema:load`:

```
-- enable_extension("plpgsql")
   -> 0.0642s
-- create_table("narrations", {:force=>:cascade})
   -> 0.1919s
-- enable_extension("plpgsql")
   -> 0.1501s
-- create_table("narrations", {:force=>:cascade})
   -> 0.0507s
```

So that seems a bit strange that it creates the `narrations` table twice. And it's unfortunate that our data is not in the table. Let's try `rails db:seed` and run a query from within `rails db`:

```
 id |             title              | published |                              source_url
----+--------------------------------+-----------+----------------------------------------------------------------------
  1 | Life Is Short                  | t         | http://www.paulgraham.com/vb.html
  2 | Drugs and the Meaning of Life  | t         | https://www.samharris.org/podcast/item/drugs-and-the-meaning-of-life
  3 | The Power Of A Digital Sabbath | f         | https://feld.com/archives/2017/02/power-digital-sabbath.html
```

So now I can add records to my `seeds.rb`, but if I want to add columns I'll need to recreate the table and seed the db. Let's try it all again for shits and giggles, but this time using `rails db:reset`, which according to [Jaco Pretorius's very helpful article](http://jacopretorius.net/2014/02/all-rails-db-rake-tasks-and-what-they-do.html) runs `rails db:drop` and `rails db:setup`, where the latter implements what we did above (`rails db:schema:load` followed by `rails db:seed`):

```
Dropped database 'narify_development'
Dropped database 'narify_test'
Created database 'narify_development'
Created database 'narify_test'
-- enable_extension("plpgsql")
   -> 0.0523s
-- create_table("narrations", {:force=>:cascade})
   -> 0.0120s
-- enable_extension("plpgsql")
   -> 0.0536s
-- create_table("narrations", {:force=>:cascade})
   -> 0.0093s
```

So now I have a better idea why it ran `create_table` twice... one for each database environment! And our seed data is replete and peachy. Next article I think I'll try to implement another model, Narrator, where a Narrator can have many Narrations, and a Narration belongs to a single Narrator.
