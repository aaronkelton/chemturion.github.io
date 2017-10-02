---
layout: post
title:  "RSpec Hidden Config Hell"
date:   2017-10-02 12:36:00 -0500
categories: ruby rspec
published: true
---
I'm over 50% through the Lynda course [Ruby: Testing with RSpec](https://www.lynda.com/Ruby-tutorials/RSpec-Testing-Framework-Ruby/183884-2.html). I've seen the basics and even built a [hefty question bank](https://github.com/chemturion/rubystems/blob/master/rspec_question_bank.md) for the Rubystems app I have in mind. But I hit a roadblock about an hour ago in the Challenges section and finally broke through. Here's what happened and here's what I did to get past it:

## Damn Outdated Material

Last week as I was going thru the Rails 5: Integrating Stripe course, I was disappointed that Stripe is now version 3 and the course was on version 2, leaving me stranded and unable to complete the tutorial. So whenever my app's behavior deviated from the Lynda course, needless to say I went to my gut explanation: the versions I'm working on and the tutorial's versions have screwed everything up! I was wrong.

### TL;DR
> Copying sample files into your working directory excludes the hidden "dotfiles". So when you're expecting there to be `.rspec` containing `--require spec_helper.rb`, your tests will go haywire.

## The Long Story

My first error was `NameError`, and it seemed as though my spec file didn't know about `APP_ROOT`:

{% highlight bash %}
Failures:

  1) Guide includes NumberHelper and #number_to_currency
     Failure/Error: @filepath = File.join(APP_ROOT, path)

     NameError:
       uninitialized constant RestaurantFile::APP_ROOT
     # ./lib/restaurant_file.rb:12:in `filepath='
     # ./lib/restaurant_file.rb:7:in `initialize'
     # ./lib/restaurant.rb:14:in `new'
     # ./lib/restaurant.rb:14:in `load_file'
     # ./lib/guide.rb:15:in `initialize'
     # ./spec/guide_spec.rb:6:in `new'
     # ./spec/guide_spec.rb:6:in `block (2 levels) in <top (required)>'
     # ./spec/guide_spec.rb:18:in `block (2 levels) in <top (required)>'
{% endhighlight %}

I knew that `init.rb` defined `APP_ROOT`, and that the app worked fine when running `ruby init.rb`. So then my question was, "Why doesn't the spec file know about `APP_ROOT` when running `rspec`?" I did some googling and then would place some "puts" statements here and there to see output when running tests. I also would redefine `APP_ROOT` locally in each file to see the behavior. I traced it down from the spec to the `filepath=` call in `restaurant_file.rb`, and all the way back to `guide_spec.rb`.

Typically the first line in the error trace thing is the line that needs fixing, and then voila— all the other errors with dependency vanish. But this guy was tricky in that I needed to investigate `guide_spec.rb`... all the way at the bottom of the `NameError` message.

Once inside `guide_spec.rb` I kept asking myself how this file should know about other files and their code. Then I remembered `spec_helper.rb`. So I looked in there and whadayaknow:

{% highlight ruby %}
# The spec_helper file is the place for loading up any code needed
# by all tests (especially the code being targeted by the tests).

require 'guide'

# Set the application root for easy reference.
APP_ROOT = File.expand_path('../..', __FILE__)
{% endhighlight %}

I could have explicitly required `spec_helper.rb` in my spec file, but that comment up top about "loading up any code needed by all tests..." was the key. "How does my spec file know about the spec helper?" Is it some Ruby or RSpec magic? Well, there is no magic; just obfuscation. Then I got the bright idea to run `cat .rspec` in my terminal, and got `cat: .rspec: No such file or directory`.

It seems as though code newbies could benefit from a class on common "gotchas". Configuration and gotchas seem to be a huge time-suck for folks in their first rodeo. Anyway, I'm gonna get back to writing RSpec now.
