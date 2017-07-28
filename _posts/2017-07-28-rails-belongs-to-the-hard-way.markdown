---
layout: post
title:  "Creating a Rails Relationship the Hard Way"
date:   2017-07-26 13:14:00 -0500
categories: rails database relationship
published: true
---
Currently we have a list of narrations the user can click on to go to the narration's show page. From there, they can navigate back to the narrations list or view the original article in a new tab. Looking forward, I'd like to not only show who narrated the narration, but let the listener click on their name/avatar, and then go to their profile page. Each narrator will have many narrations. Each narration will have exactly one narrator. We could have duos some day, but we'll table that until we need it. So let's get started; beginning with the end in mind, I'd like to start from the Narration view. Let's add a link to take us to a Narrator show page:

{% highlight haml %}
= @narration.title
= link_to 'Narrated by Narrator', narrator_path
{% endhighlight %}

There's some pseudo-code as a placeholder for now. I can envision the Narrator table having a `first_name` and `last_name` column, and I'll need to use the Narrator controller to create a `#full_name` method. But first, let's define the route `narrator_path`:

{% highlight ruby %}
get 'narrator/:id', to: 'narrators#show', as: :narrator
{% endhighlight %}

Here's the error we get if we click on the link in this state:

```
Routing Error
uninitialized constant NarratorsController
```

So in case you were wondering whether we should create our model or controller next, now you know! Let's make a new file called `app/controllers/narrators_controller.rb` and give it a show method:

{% highlight ruby %}
class NarratorsController < ApplicationController
  def show
  end
end
{% endhighlight %}

Here's my next error in the browser:

```
ActionController::UnknownFormat in NarratorsController#show
NarratorsController#show is missing a template for this request format and variant.
```

Now let's create `app/views/narrators/show.html.haml`:

{% highlight haml %}
%h4 Welcome to the Narrator show page.
{% endhighlight %}

Now each Narration show page has a link that takes us to its Narrator show page, sort of. Our route is using the same id as the Narration. Let's create our Narrator model now and add some data to shine some light, `app/models/narrator.rb`:

{% highlight ruby %}
class Narrator < ApplicationRecord
end
{% endhighlight %}

Now, let's create a migration, `db/migrate/20170728110334_create_narrators.rb`, and similar to Twitter, to give personality and possible anonymity, let's give the Narrator a `username`:

{% highlight ruby %}
class CreateNarrators < ActiveRecord::Migration[5.1]
  def change
    create_table :narrators do |t|
      t.string :first_name
      t.string :last_name
      t.string :username
      t.timestamps
    end
  end
end
{% endhighlight %}

Run `rails db:migrate`:

```
== 20170728110330 CreateNarrators: migrating ==================================
-- create_table(:narrators)
   -> 0.0961s
== 20170728110330 CreateNarrators: migrated (0.0964s) =========================
```

Now let's create some seed data to work with. Here's what I'll drop into `seeds.rb` and then run `rails db:seed`:

{% highlight ruby %}
narrators_array = [
  {first_name: 'Aaron', last_name: 'Kelton', username: 'chemturion'},
  {first_name: 'Morgan', last_name: 'Freeman', username: 'shawshank'}
]
narrators_array.each do |rec|
  Narrator.find_or_create_by(first_name: rec[:first_name], last_name: rec[:last_name], username: rec[:username])
end
{% endhighlight %}

I double checked that duplicate records weren't created for `Narrations` and confirmed using `rails db` and `select * from narrators;` that we have our table populated:

```
id | first_name | last_name |  username  
----+------------+-----------+-----------
 1 | Aaron      | Kelton    | chemturion
 2 | Morgan     | Freeman   | freeman    
```

Now for the purpose of demonstration, how can we assign a narrator to a narration? Let's suppose I narrated the "Life in Short" article, and Morgan Freeman narrated the "Drugs and the Meaning of Life" article. Generically, I the Narrator could have many narrations. Let's define this relationship in each model file. First, `app/models/narrator.rb` `has_many` narrations:

{% highlight ruby %}
class Narrator < ApplicationRecord
  has_many :narrations
end
{% endhighlight %}

And second, `app/models/narration.rb` `belongs_to` a narrator:

{% highlight ruby %}
class Narration < ApplicationRecord
  belongs_to :narrator
end
{% endhighlight %}

Now, conceptually this makes me think that only the Rails objects know about each other. I'm not sure that we've successfully linked the data in the tables together using a foreign key. Since we've already generated a migration to create the tables, let's generate a migration to add the connections. Here's the Rails Guide that helped me: http://edgeguides.rubyonrails.org/association_basics.html#updating-the-schema. Unfortunately their migration isn't for after the table's been created. Let's do something comparable, so new file `db/migrate/20170728114023_add_reference_to_narrations.rb` and using http://api.rubyonrails.org/v5.1.2/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_reference as guidance:

{% highlight ruby %}
class AddReferenceToNarrations < ActiveRecord::Migration[5.1]
  def change
    add_reference :narrations, :narrator, foreign_key: true
  end
end
{% endhighlight %}

Running `rails db:migrate` gives us:

```
== 20170728114023 AddReferenceToNarrations: migrating =========================
-- add_reference(:narrations, :narrator, {:foreign_key=>true})
   -> 0.1256s
== 20170728114023 AddReferenceToNarrations: migrated (0.1258s) ================
```

Now let's look inside running `rails db` and `select * from narrations;`. We should see a `narrator_id` column, but we can also confirm by looking in our `db/schema.rb` file:

{% highlight ruby %}
ActiveRecord::Schema.define(version: 20170728114023) do

  # These are extensions that must be enabled in order to support this database
  enable_extension "plpgsql"

  create_table "narrations", force: :cascade do |t|
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.string "title"
    t.boolean "published"
    t.string "source_url"
    t.bigint "narrator_id"
    t.index ["narrator_id"], name: "index_narrations_on_narrator_id"
  end

  create_table "narrators", force: :cascade do |t|
    t.string "first_name"
    t.string "last_name"
    t.string "username"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  add_foreign_key "narrations", "narrators"
end
{% endhighlight %}

Notice that the `narrations` table has a column named `narrator_id` with type `bigint`, and an index on that column. Nothing changed for the `narrator` table, but at the end of the `schema.rb` file, we have the `add_foreign_key` statement. Ok, back to the use case.

When a narrator creates a narration, their id will be plopped into the narration table as narrator_id. But for now using sample data, I think I'll need to manually put that into my seeds.rb file and do the whole reset & load dance `rails db:reset`.

{% highlight ruby %}
narrations_array = [
  {title: 'Life Is Short',                  published: true,  source_url: 'http://www.paulgraham.com/vb.html', narrator_id: 1},
  {title: 'Drugs and the Meaning of Life',  published: true,  source_url: 'https://www.samharris.org/podcast/item/drugs-and-the-meaning-of-life', narrator_id: 2},
  {title: 'The Power Of A Digital Sabbath', published: false, source_url: 'https://feld.com/archives/2017/02/power-digital-sabbath.html', narrator_id: 2}
]

narrations_array.each do |rec|
  Narration.find_or_create_by(title: rec[:title], published: rec[:published], source_url: rec[:source_url], narrator_id: rec[:narrator_id])
end
{% endhighlight %}

After the reset, here's my feedback:

```
Dropped database 'narify_development'
Dropped database 'narify_test'
Created database 'narify_development'
Created database 'narify_test'
-- enable_extension("plpgsql")
   -> 0.0463s
-- create_table("narrations", {:force=>:cascade})
   -> 0.0403s
-- create_table("narrators", {:force=>:cascade})
   -> 0.0086s
-- add_foreign_key("narrations", "narrators")
   -> 0.0162s
-- enable_extension("plpgsql")
   -> 0.0468s
-- create_table("narrations", {:force=>:cascade})
   -> 0.0248s
-- create_table("narrators", {:force=>:cascade})
   -> 0.0130s
-- add_foreign_key("narrations", "narrators")
   -> 0.0755s
```

Now, let's jump back into the `narrators_controller.rb` and assign the narrator in question to the `@narrator` instance variable.

{% highlight ruby %}
class NarratorsController < ApplicationController
  def show
    @narrator = Narrator.find(params[:id])
  end
end
{% endhighlight %}

I'm not totally sure this is going to work. I think the `params` dude is going to pass an `:id`, but I'm not sure how it knows to pass the `narration.narrator_id`, if that makes sense. Let's add this little code to our narrator's show view:

{% highlight haml %}
%h4 Welcome to the Narrator show page.
= @narrator.username
{% endhighlight %}

Unfortunately, my narrator's id matches the narration id, so I can't fully tell if it's working. To test, let's change the published value for narration id=3 to true, and see if the corresponding narrator route tries to pull 3 instead of 2... And it's pulling the wrong id. So I think we should now modify our `link_to` helper in `app/views/narrations/show.html.haml`:

{% highlight haml %}
= link_to 'Narrated by ' + @narration.narrator.username, narrator_path(@narration.narrator)
{% endhighlight %}

Now we can navigate to the narrator show page from the narration show page. Next thing we want to do is list all the narrations for the narrator when we go to their page, similar to seeing a user's Tweets on their Twitter profile page.
