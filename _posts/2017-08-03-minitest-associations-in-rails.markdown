---
layout: post
title:  "Minitest Associations in Rails"
date:   2017-08-03 14:00:00 -0500
categories: rails minitest testing
published: true
---
I had some trouble after my last post on writing tests for my Narrations model. The guide I was using detailed how to test for associations from the `has_many` side, which would be for my Narrator model. I tried for quite some time testing from the Narration side like so:

{% highlight ruby %}
test 'invalid without narrator reference' do
  assert_equal 1, @narrator.narrations.size
end
{% endhighlight %}

But I kept getting this error in terminal:

```
Error:
NarrationTest#test_invalid_without_narrator_reference:
ActiveRecord::Fixture::FixtureError: table "narrators" has no column named "narration".
```

My `narrations.yml` fixture looked like:

{% highlight yaml %}
valid:
  title: How to Test Rails Models with Minitest
  published: true
  source_url: https://semaphoreci.com/community/tutorials/how-to-test-rails-models-with-minitest
  narrator: heidar

valid:
  title: Mocking in Ruby with MiniTest
  published: true
  source_url: https://semaphoreci.com/community/tutorials/mocking-in-ruby-with-minitest
  narrator: heidar
{% endhighlight %}

And my `narrators.yml` fixture looked like:

{% highlight yaml %}
heidar:
  username: henrikbernhardsson
  first_name: Heidar
  last_name: Bernhardsson
  narration: valid
{% endhighlight %}

I thought that the two fixtures would know about each other this way. Basically I just needed to remove the test from my `narration_test.rb` and the `narration: valid` from my `narrators.yml` fixture. From the fixture standpoint, the foreign key of `narrator_id` lives in the `narrations` table, so I didn't need to tell my Narrator fixture which Narrations it had; I only need to tell my Narration who its Narrator was, and the association test would only live in the `narrator_test.rb` test file. Here's my Narrator test, and the Narrator fixture that worked:

{% highlight ruby %}
test 'narrator has many narrations' do
  assert_equal 2, @narrator.narrations.size
end
{% endhighlight %}

{% highlight yaml %}
heidar:
  username: henrikbernhardsson
  first_name: Heidar
  last_name: Bernhardsson
  # no need to reference the narration fixture
{% endhighlight %}

Running `rails test` gave me a very satisfying:

```
Running via Spring preloader in process 30957
Run options: --seed 397

# Running:

.........

Finished in 0.437782s, 20.5582 runs/s, 34.2636 assertions/s.
9 runs, 15 assertions, 0 failures, 0 errors, 0 skips
```

Next up let's tackle the (conceptually) easy task of testing the view for the About page.
