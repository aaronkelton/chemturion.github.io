---
layout: post
title:  "Ruby Singleton: Ballon dOr"
date:   2017-08-29 18:39:00 -0500
categories: ruby soccer
published: true
---
Every year FIFA selects the best soccer player in the world. There can only be one. My guess is that this year (2017), Cristiano Ronaldo will match Lionel Messi with his fifth Ballon d'Or (Golden Ball). So how can we model this in Ruby using the Singleton pattern?

The Singleton pattern ensures that a particular class can only have one instance of itself. Let's try to build this from scratch:

{% highlight ruby %}
class GoldenBall
end
{% endhighlight %}

Presently this code is the foundation for any regular class in Ruby. `GoldenBall.new` will create a new object each time it's invoked. Not good. There can only be one.

{% highlight bash %}
ronaldo = GoldenBall.new
=> #<GoldenBall:001>

messi = GoldenBall.new
=> #<GoldenBall:002>
{% endhighlight %}

We should be able to create a `ronaldo` instance of the GoldenBall class, but not a `messi` instance. Sorry Leo. Our `GoldenBall` class should know about itself, and how many instances has been created. Let's try using a class variable, `@@instances`. Each time we create a new instance, let's capture it therein.

{% highlight ruby %}
class GoldenBall
  # define the class variable as an empty array
  @@instances = []

  # when initialized, push the instance of the new
  # GoldenBall class into the class variable array
  def initialize
    @@instances << self
  end

  # provide a method to access the instances array
  def self.instances
    @@instances
  end
end
{% endhighlight %}

{% highlight bash %}
ronaldo = GoldenBall.new
=> #<GoldenBall:001>

GoldenBall.instances
=> [#<GoldenBall:001>]

messi = GoldenBall.new
=> #<GoldenBall:002>

GoldenBall.instances
=> [#<GoldenBall:001>, #<GoldenBall:002>]
{% endhighlight %}

OK, so we've added the functionality to track our newly created instances. Each new instance is an object of the `GoldenBall` class, and that object is pushed into our `@@instances` class variable array. How can we prevent more than one instance being initialized? Let's add some conditional logic to our `#initialize` method so that we only add the new instance if our array is empty.

{% highlight %}
class GoldenBall
  # define the class variable as an empty array
  @@instances = []

  # when initialized, push the instance of the new
  # GoldenBall class into the class variable array
  # only if the array is empty
  def initialize
    @@instances << self if @@instances.empty?
  end

  # provide a method to access the instances array
  def self.instances
    @@instances
  end
end
{% endhighlight %}

{% highlight bash %}
ronaldo = GoldenBall.new
=> #<GoldenBall:001>

GoldenBall.instances
=> [#<GoldenBall:001>]

messi = GoldenBall.new
=> #<GoldenBall:002>

GoldenBall.instances
=> [#<GoldenBall:001>]
{% endhighlight %}

Hmm. After creating `ronaldo` we were still able to create a second object `messi`, even though our `GoldenBall.instances` only holds a single `ronaldo` object. At this point, we have a `Singleish` design pattern; not a true `Singleton`. But even this `Singleish` implementation is an illusion. How can we actually determine [how many instances exist in memory](https://stackoverflow.com/questions/14318079/how-do-i-list-all-objects-created-from-a-class-in-ruby)?

{% highlight bash %}
ObjectSpace.each_object(GoldenBall).count
=> 2
{% endhighlight %}

What if instead of *not doing anything* on subsequent instantiation, we raise an error. That would prevent subsequent instances, right? Here's my new initialize method:

{% highlight ruby %}
# GoldenBall class
def initialize
  @@instances.empty? ? @@instances << self : raise
end
{% endhighlight %}

{% highlight bash %}
ronaldo = GoldenBall.new
=> #<GoldenBall:001>

GoldenBall.instances
=> [#<GoldenBall:001>]

messi = GoldenBall.new
RuntimeError:
	from (irb):9:in `initialize'
	from (irb):20:in `new'
	from (irb):20
	from /Users/~/.rbenv/versions/2.4.1/bin/irb:11:in `<main>'

GoldenBall.instances
=> [#<GoldenBall:001>]
{% endhighlight %}

Well there we go! I'm sure there's something missing, but this will do for a newbie. Sorry Leo.
