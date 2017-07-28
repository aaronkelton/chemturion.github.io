---
layout: post
title:  "Show All Narrations for a Narrator"
date:   2017-07-28 14:10:00 -0500
categories: rails relationship
published: true
---
Eventually we'll have a way to search for Narrator profiles, but we won't have an index page to show all Narrators. Right now the only way is to navigate from a Narration show page. Currently the Narrator show page works, but we'll want to list all Narrations from that Narrator. Let's do that now. We may need a nested resource route. For starters, let's paste our code from our Narration index page over on the Narrator show page and see what happens, so we're modifying `app/views/narrators/show.html.haml`:

{% highlight haml %}
%h4 Welcome to the Narrator show page.
= @narrator.username

- @narrations.each do |narration|
  %ul
    - if narration.published
      %li
        = narration.title
        = link_to 'Listen', narration_path(narration)
{% endhighlight %}

And we get the following error:

```
NoMethodError in Narrators#show
undefined method `each' for nil:NilClass
```

So our `Narrators#show` action doesn't have access to the instance variable, `@narrations`. Let's fix that in our `narrators_controller.rb` file:

{% highlight ruby %}
class NarratorsController < ApplicationController
  def show
    @narrator = Narrator.find(params[:id])
    @narrations = @narrator.narrations
  end
end
{% endhighlight %}

Now the `@narrations` instance variable corresponds to this particular narrator's narrations. Ok, that was relatively painless!
