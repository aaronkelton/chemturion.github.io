---
layout: post
title:  "Codewars: My First Kata — Clear the Catacombs!"
date:   2017-08-30 14:24:00 -0500
categories: ruby codewars
published: true
---
You'll never guess how much Codewars paid me to create [a problem for their website][1]...

{% highlight ruby %}
payment.zero? # => true
{% endhighlight %}

This crowdsourced phenomenon that derives their content is similar to my business model for Narify: people giving of their time and effort to create something for their own, and everyone else's, gain. So yes, I created [a very simple but thoughtful Kata][1], or so I thought.

Here's the problem statement:

>There are many ways to clear the catacombs, er um, I mean methods to modify your strings! Let's say you've got a string called `catacombs`, and it's full of spiders and bats, like so:
>
>```ruby
catacombs = "spiders and bats!"
```
>
>While you may be tempted to call `catacombs.overtune` to clear the catacombs, unfortunately that's an undefined method in Ruby!
>
>Your job is to find and use the method on the `catacombs` so that it's cleared and returns an empty string. Good luck!
>
>```ruby
clearify("spiders and bats!") # => ""
```

It started by me wanting a quick and easy win for my first attempt. So I looked at the [Ruby docs'](https://ruby-doc.org/core-2.3.0/String.html) String methods, and found [the `#clear` method](https://ruby-doc.org/core-2.3.0/String.html#method-i-clear). So easy! Take a string like `catacombs = "spiders and bats!"`, then run `catacombs.clear # => ""` to return an empty string. I thought this would be a good String method for newbies to learn. Sure, you could reassign the variable like so: `catacombs = ""`. But that assumes your variable already exists. If not, you've just created a new variable set to an empty string. Another thing that reassignment does is it changes your object_id!

{% highlight bash %}
catacombs = "spiders and bats!"
=> "spiders and bats!"

catacombs.object_id
=> 70232165920920

catacombs = ""
=> ""

catacombs.object_id
=> 70232169423060
{% endhighlight %}

Also, reassignment can work on most, if not any, unfrozen variable of any data type. And this reassignment will slip by silently when an error might be more appropriate. For example:

{% highlight bash %}
catanums = 1
=> 1

catanums.clear
NoMethodError: undefined method `clear' for 1:Integer
{% endhighlight %}

Here's the desired solution with object id inspection:

{% highlight bash %}
catacombs = "spiders and bats!"
=> "spiders and bats!"

catacombs.object_id
=> 70232169411420

catacombs.clear
=> ""

catacombs.object_id
=> 70232169411420
{% endhighlight %}

Creating the description wasn't that difficult because I'm used to using Markdown for this Jekyll blog. The difficult part was writing the tests, because 1) I'm not incredibly familiar with writing Ruby tests, and 2) I'm not familiar with Codewars' test suite. And while I was able to test that the method which houses where `catacombs.clear` should return zero, I wasn't able to test that the `catacombs` variable was changed to zero. I need to go back and do that. Also, I need to check that the object_id remains the same, otherwise you could pass the Kata by simply returning `''`. Not what I intended. Moreover, out of its 6 completions so far, 3 have voted on its Satisfaction rating: 1 for "Somewhat Satisfied" and 2 for "Not Satisfied". 2 of the 6 solutions were mine, and hey!— whaddayaknow!— There's three people who submitted this answer:

{% highlight ruby %}
def clearify(catacombs)
  ''
end
{% endhighlight %}

OK, so I've since updated my tests as follows:

{% highlight ruby %}
Test.assert_equals(clearify("spiders and bats!"), "")
Test.expect_error("Both function and variable should return an empty string using a particular method. Try again or go to https://ruby-doc.org/core-2.3.0/String.html") { clearify(1) }
{% endhighlight %}

The second test is only run on the final attempt, so answers like `''` or `catacombs = ''` will work initially, but fail upon final submission. This test forces the user to employ the `#clear` method. The testing documentation wasn't helpful in getting the `Test.expect_error` to work, so I had to go looking for someone else's example. Lessons learned, and I hope to make many more Katas. :)

Here's the link to the Kata in full: [https://www.codewars.com/kata/clear-the-catacombs/train/ruby](https://www.codewars.com/kata/clear-the-catacombs/train/ruby)

[1]: https://www.codewars.com/kata/clear-the-catacombs/train/ruby
