---
layout: post
title:  "Codewars: Transpose two strings in an array"
date:   2017-08-09 14:55:00 -0500
categories: ruby codewars
published: true
---
I'd like to walk you through my thought process solving a problem, or Kata, on Codewars: [Transpose two strings in an array](https://www.codewars.com/kata/transpose-two-strings-in-an-array/train/ruby/)

> You will be given an array that contains two strings. Your job is to create a function that will take those two strings and transpose them, so that the strings go from top to bottom instead of left to right.
>
> e.g. transposeTwoStrings(['Hello','World']);
>
> should return
>
> ```
> H W  
> e o  
> l r  
> l l  
> o d
> ```
>  
> A few things to note:
>
> There should be one space in between the two characters
> You don't have to modify the case (i.e. no need to change to upper or lower)
> If one string is longer than the other, there should be a space where the character would be

I typically approach a solution with brute force on my first pass. I decided to loop through "num" times based on the longer string. Then I check for presence for each string's i'th character, and add the appropriate string to a res array. Increment i by 1 until the loop's done, and then join the res array's strings separated by a new line. Like so:

{% highlight ruby %}
def transpose_two_strings(arr)
  arr[0].length > arr[1].length ? num = arr[0].length : num = arr[1].length
  i = 0
  res = []
  while i < num do
    if arr[0][i] && arr[1][i]
      res << "#{arr[0][i]} #{arr[1][i]}"
    elsif arr[0][i] && !arr[1][i]
      res << "#{arr[0][i]}  "
    elsif !arr[0][i] && arr[1][i]
      res << "  #{arr[1][i]}"
    end
    i += 1
  end
  res.join("\n")
end
{% endhighlight %}

However, looking at other code after I submitted mine, I noticed a refactoring opportunity using the "implicit return-if-true OR" statement, like this: `res << "#{arr[0][i] || " "} #{arr[1][i] || " "}"`. So we take all that if/elsif/elsif business and replace it:

{% highlight ruby %}
def transpose_two_strings(arr)
  arr[0].length > arr[1].length ? num = arr[0].length : num = arr[1].length
  i = 0
  res = []
  while i < num do
    res << "#{arr[0][i] || " "} #{arr[1][i] || " "}"
    i += 1
  end
  res.join("\n")
end
{% endhighlight %}

I don't know if it's called the "implicit return-if-true OR" statement, but it makes sense to me. The next step is to clean up the "num" evaluator. Other solutions used that funny block notation, like so: `num = arr.map(&:length).max`. The long form looks like this:

{% highlight ruby %}
num = arr.map do |el|
  el.length
end.max
{% endhighlight %}

So it takes each element, or "el", and transforms the each element into their respective lengths. Calling `.max` at the end returns the maximum value. My current solution works for two strings, but going up to "n" strings in the array, this `#map` method works more appropriately. Another solution is to use "max_by" instead of "map". I'll show it here, and in fact I think I like it better. `num = arr.max_by(&:length).length`. The long form is:

{% highlight ruby %}
num = arr.max_by do |el|
  el.length
end.length
{% endhighlight %}

It takes the array and returns the element of maximum length, instead of returning an array of lengths to apply a max to. We could have used "size", but I prefer "length" for strings. Here's my code so far:

{% highlight ruby %}
def transpose_two_strings(arr)
  num = arr.max_by(&:length).length
  i = 0
  res = []
  while i < num do
    res << "#{arr[0][i] || " "} #{arr[1][i] || " "}"
    i += 1
  end
  res.join("\n")
end
{% endhighlight %}

Now, let's get rid of that "while" loop. We're effectively iterating from "i = 0" to "i = num", and plugging in 0 thru "num" for "i" to reference the character in our string. Let's instead take the range and iterate over each i'th value, and assign strings to our res array that way (and we can remove our `i = 0` variable assignment since it's given at the range beginning):

{% highlight ruby %}
def transpose_two_strings(arr)
  num = arr.max_by(&:length).length
  res = []
  (0...num).each do |i|
    res << "#{arr[0][i] || " "} #{arr[1][i] || " "}"
  end
  res.join("\n")
end
{% endhighlight %}

I think we can go further still. The `#each` method requires that we assign results to our res array. What if we could instead just transform our range into an array using `#map`? Let's give it a try.

{% highlight ruby %}
def transpose_two_strings(arr)
  num = arr.max_by(&:length).length
  (0...num).map do |i|
    "#{arr[0][i] || " "} #{arr[1][i] || " "}"
  end.join("\n")
end
{% endhighlight %}

So we don't need the res array since each evaluation is the return value given back to our transformed range (to array). But we then need to append `.join("\n")` to the `end`. So far so good. Can we make it better? Cosmetically, yes. We could plug num's definition into the range, and use `{ }` instead of `do end`. But let's leave it like so in order that folks following behind us can read it better.
