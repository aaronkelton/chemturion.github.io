---
layout: post
title:  "Single-Line if-Statement sans Else"
date:   2017-07-15 18:00:00 -0500
categories: ruby ternary conditional syntax
published: true
---
<!-- show how sans else statement requires expression if evaluation -->

Traditional if-statement in Ruby...
{% highlight ruby %}
if !!1
  "traditional sans else"
end
# => "traditional sans else"
{% endhighlight %}


Ternary if-statement (sans else-y colon) raises an exception...
{% highlight ruby %}
!!1 ? "ternary sans else-y colon"
# => syntax error, unexpected end-of-input, expecting ':'
{% endhighlight %}

So while you could append `: nil` to lower the exception...
{% highlight ruby %}
!!1 ? "forced ternary with else-y colon" : nil
# => "forced ternary with else-y colon"
{% endhighlight %}

I guess it's better to abandon the ternary and adopt the single-line...
{% highlight ruby %}
"syntactic non-ternary single-line sugar" if !!1
# => "syntactic non-ternary single-line sugar"
{% endhighlight %}

So the lesson I learned is that in addition to the traditional approach, there are both ternary and single-line means of writing if-statements. You could technically use any value instead of the forced `: nil`, like `: false`, but I chose `: nil` because both the traditional and single-line statements return `nil` if their conditional evaluates to `false`.

Side note: I used the double bang (`!!`) because I just learned that little trick, which takes something "truthy" like the integer 1, makes it "not truthy", aka `false` with the first bang, and then makes it explicitly `true` with the second bang.
