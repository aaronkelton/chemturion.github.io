---
layout: post
title:  "Rails Active Support Array Extract!"
date:   2018-09-01 08:15:00 -0500
categories: ruby rails array
published: true
---
A couple weeks ago I got my standard [This Week in Rails](https://rails-weekly.ongoodbits.com/2018/08/18/cookies-with-purpose-array-extract-and-more) newsletter, and saw that someone had [added the `extract!` "Extract Bang" method](https://github.com/rails/rails/pull/33137) to the Array class in Rails (Active Support):

> The method removes and returns the elements for which the block returns a true value.

{% highlight ruby %}
numbers = [1, 2, 3, 4]
odd_numbers = numbers.extract!(&:odd?) # => [1, 3]
numbers # => [2, 4]
{% endhighlight %}

Arrays are my favorite, so I dove in a bit and learned more than I bargained for.

---

### Extending Ruby's Array Class in Active Support

First off, I was reminded that we can add our own methods to Ruby's core classes, and I saw the convention that Rails uses to incorporate these additions. If you write your `extract!` code located like so:

```
active_support/core_ext/array/extract.rb
```

then you must `require` it in a file located like so:

```
active_support/core_ext/array.rb
```

where `array.rb` has lines looking something like:

{% highlight ruby %}
require "active_support/core_ext/array/extract"
{% endhighlight %}

So you've got all these custom method files extending the core Array class, and one file that requires them all. I learned that these extensions, when added, should have supporting tests (I also learned that Rails doesn't use RSpec for testing!), located at

```
rails/activesupport/test/core_ext/array/extract_test.rb
```

I'd like to go deeper, but this overview is a good start to understanding the Rails framework better. Read more about [Active Support Core Extensions](https://guides.rubyonrails.org/active_support_core_extensions.html).

---

### Array Elements Can Already Be Extracted Though!

Ruby's Enumerable module has had the `#partition` method going all the way back to [Ruby version 1.8.6](https://ruby-doc.org/core-1.8.6/Enumerable.html#method-i-partition) (at least... I'm not sure how to go back further than that!). `#partition` returns two arrays based on an expression that returns `true` (first element), or `false` (second element):

{% highlight ruby %}
[1,2,3,4,5,6].partition { |v| v.even? }  #=> [[2, 4, 6], [1, 3, 5]]
{% endhighlight %}

If you want to assign these values, you gotta do that dual variable assignment bit:

{% highlight ruby %}
evens, odds = [1,2,3,4,5,6].partition { |v| v.even? }
evens #=> [2, 4, 6]
odds  #=> [1, 3, 5]
{% endhighlight %}

>For any beginners, remember that the [Array class includes the Enumerable module](https://ruby-doc.org/core-2.5.1/Array.html#includes-section), so it can use the `#partition` method.

The problem with this approach is that it's cumbersome. We want to extract elements, not partition-then-assign them! And later we'll show how [Bogdan](https://github.com/bogdanvlviv) ensured his `Array#extract!` method was better than the `Enumerable#partition` method.

### How to Extract!

Here's [the method](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/core_ext/array/extract.rb) in all its 6-line glory:

{% highlight ruby %}
def extract!
  return to_enum(:extract!) { size } unless block_given?

  extracted_elements = []

  reject! do |element|
    extracted_elements << element if yield(element)
  end

  extracted_elements
end
{% endhighlight %}

The first line `return to_enum...` ensures that other methods can be chained thereon. I get the idea, but I'm not sure when you'd call `[1,2,3].extract!.<another_method>`, so I'd need a concrete example to make it stick. I also don't know why `{ size }` is used. Need to be walked thru that one. Just think of the first line as the tooth pick that holds together the sandwich.

![Toothpick Sandwich]({{ site.url }}/assets/toothpick_sandwich.jpg)

The `extracted_elements` variable is the bread of our method sandwich. It starts as an empty array, and then returns its new self after extraction is complete.

Let's look at the meat of our method sandwich:

{% highlight ruby %}
reject! do |element|
  extracted_elements << element if yield(element)
end
{% endhighlight %}

When I first looked at this, I thought, "Does the `#reject!` method interact with my array, or each element from the block?" The syntactic sugar got a bit sweet, so here's that first line in all its salty glory:

{% highlight ruby %}
self.reject! do |element|
{% endhighlight %}

SELF! `self` in this case is our array. So if we have `[1,2,3,4,5,6].extract!{|n|n.even?}`, then `self` would be `[1,2,3,4,5,6]`.

I know how to shovel (`<<`) elements into an array, but I was a bit confused about the `yield(element)` bit. This is where our block comes into play. Imagine if we had a method called `#extract_even_numbers!`, it might look something like this:

{% highlight ruby %}
def extract_even_numbers!
  even_numbers = []

  self.reject! do |element|
    even_numbers << element if element.even?
  end

  even_numbers
end
{% endhighlight %}

This very specific extraction method for getting even numbers from an array depends on the `element.even?` part. OK, now put yourself back into the frame of mind for the `#extract!` method. The `yield(element)` bit (I almost used the forbidden "simply" term!) looks at the imposing block, holds the door open, and says, "After you!". Big block steps in, does it's little evaluation, and for each element that returns true, we shovel said element into the `extracted_elements` variable.

---
### Is extract! Actually Better than Partition?

[Take a look at the benchmark](https://github.com/rails/rails/pull/33137#issuecomment-397527806) that compares the methods to see which ones are faster!

At our Continuations meeting this week, [@AaronLasseigne](https://twitter.com/AaronLasseigne) also suggested trying `#with_object` to see if we could make this method sandwich an unwich. Here's the method, which does indeed function appropriately.

{% highlight ruby %}
def extract!
  return to_enum(:extract!) { size } unless block_given?

  reject!.with_object([]) do |element, extracted_elements|
    extracted_elements << element if yield(element)
  end
end
{% endhighlight %}

But when I ran the benchmark similar to Bogdan's, it turns out that it didn't run as fast as his. But it was worth a try! The results using `#with_object` are from `Array#extract_v3!`

```
Warming up --------------------------------------
     Array#partition     1.000  i/100ms
   Array#extract_v1!     1.000  i/100ms
   Array#extract_v2!     1.000  i/100ms
   Array#extract_v3!     1.000  i/100ms
Calculating -------------------------------------
     Array#partition      0.807  (± 0.0%) i/s -      4.000  in   5.002523s
   Array#extract_v1!      1.601  (± 0.0%) i/s -      8.000  in   5.073510s
   Array#extract_v2!      1.772  (± 0.0%) i/s -      9.000  in   5.079500s
   Array#extract_v3!      1.516  (± 0.0%) i/s -      8.000  in   5.285609s

Comparison:
   Array#extract_v2!:        1.8 i/s
   Array#extract_v1!:        1.6 i/s - 1.11x  slower
   Array#extract_v3!:        1.5 i/s - 1.17x  slower
     Array#partition:        0.8 i/s - 2.20x  slower
```
