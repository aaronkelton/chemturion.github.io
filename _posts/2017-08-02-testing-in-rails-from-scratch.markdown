---
layout: post
title:  "Testing in Rails the Hard Way"
date:   2017-08-02 16:00:00 -0500
categories: rails rspec testing
published: true
---
So far I've built a couple models that relate to each other, their controllers and routes, and a little React code. Before things become too unwieldy, I think now is a good time to create some tests for these guys. I'm going mostly off faith that the convention to test is generally a good thing. I can't appreciate fully how much time and effort testing will save me, but I trust this method should drive my app's development.

Following along with the [Rail's Guide for testing](http://edgeguides.rubyonrails.org/testing.html), let's create a test that ensures our Narration model validates for the presence of a _title_. Since we created our app from scratch, we don't have the generated folders & files. Let's minimally create these one at a time. First up, the folder called `test`. This folder will be nestled between `log` and `tmp` for my project directory. Within `test`, let's create a folder for `models` and the file `test_helper.rb`. Inside `test_helper.rb`, let's add this code:

{% highlight ruby %}
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'

class ActiveSupport::TestCase
  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical order.
  fixtures :all

  # Add more helper methods to be used by all tests here...
end
{% endhighlight %}

This helper file holds the default configuration for our tests. Now within the `models` folder, let's create our `narration_test.rb` file. Therein type on the first line `require 'test_helper.rb'`. You can leave off `.rb` but I prefer being verbose at this stage. It's very evident now that you're requiring our testing configuration file. Next, create an inherited class from ActiveSupport::TestCase like so:

{% highlight ruby %}
require 'test_helper.rb'
class NarrationTest < ActiveSupport::TestCase
end
{% endhighlight %}

Let's add some pseudo-code and run the test. Here's my test file with a test code block added:

{% highlight ruby %}
require 'test_helper.rb'
class NarrationTest < ActiveSupport::TestCase
  test "the truth" do
    assert true
  end
end
{% endhighlight %}

Before we run the test, the big three things are starting with `test`, `do-end` block, and `assert`ing something that returns a Boolean. I could be wrong about this claim, but at present, this pattern rings true.

Back in our terminal, run `rails test`:

```
Running via Spring preloader in process 4209
Run options: --seed 34980

# Running:

.

Finished in 0.010372s, 96.4134 runs/s, 96.4134 assertions/s.
1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

Well, that's nifty. Let's change our test to make it fail!

{% highlight ruby %}
require 'test_helper.rb'
class NarrationTest < ActiveSupport::TestCase
  test "should not save narration without title" do
    narration = Narration.new
    assert_not narration.save
  end
end
{% endhighlight %}

And in "typical Man City" fashion, the test passed, which is actually a failure for our case (because we expected it to fail). The only code I have in my Narration model is `belongs_to :narrator`. When I comment this line out, it will fail. Here's the passing test:

```
Running via Spring preloader in process 5664
Run options: --seed 32928

# Running:

.

Finished in 0.061844s, 16.1697 runs/s, 16.1697 assertions/s.
1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

And the failing test:

```
Running via Spring preloader in process 6488
Run options: --seed 19291

# Running:

F

Failure:
NarrationTest#test_should_not_save_narration_without_title [/Users/kelton/Sites/narify/test/models/narration_test.rb:5]:
Expected true to be nil or false

bin/rails test test/models/narration_test.rb:3

Finished in 0.045048s, 22.1985 runs/s, 22.1985 assertions/s.
1 runs, 1 assertions, 1 failures, 0 errors, 0 skips
```

So in this case, it's actually not a very good test! So let's jump over to [Semaphore's testing guide](https://semaphoreci.com/community/tutorials/how-to-test-rails-models-with-minitest). Let's define a "valid" Narration as one that has a title, published value (true or false), and a source_url. We'll add the related Narrator later.

{% highlight ruby %}
test 'valid narration' do
  narration = Narration.new(title: 'How to Test Rails Models with Minitest', published: true, source_url: 'https://semaphoreci.com/community/tutorials/how-to-test-rails-models-with-minitest')
  assert narration.valid?, "Narration should include a title, published boolean, and source_url be valid."
end

test 'invalid without title' do
  narration = Narration.new(published: true, source_url: 'https://semaphoreci.com/community/tutorials/how-to-test-rails-models-with-minitest')
  refute narration.valid?, 'narration is valid without a title'
  assert_not_nil narration.errors[:title], 'no validation error for title present'
end

test 'invalid without published status' do
  narration = Narration.new(title: 'How to Test Rails Models with Minitest', source_url: 'https://semaphoreci.com/community/tutorials/how-to-test-rails-models-with-minitest')
  refute narration.valid?, 'narration is valid without a published status'
  assert_not_nil narration.errors[:published], 'no validation error for published status present'
end

test 'invalid without source_url' do
  narration = Narration.new(title: 'How to Test Rails Models with Minitest', published: true)
  refute narration.valid?
  assert_not_nil narration.errors[:source_url]
end
{% endhighlight %}

Ok, it seems like these tests could grow considerably for each valid/invalid check. I've commented out my `belongs_to :narrator` line and got the following results when running `rails test test/models/narration_test.rb`:

```
Running via Spring preloader in process 8647
Run options: --seed 24041

# Running:

F

Failure:
NarrationTest#test_invalid_without_title [/Users/kelton/Sites/narify/test/models/narration_test.rb:15]:
narration is valid without a title

bin/rails test test/models/narration_test.rb:13

.F

Failure:
NarrationTest#test_invalid_without_published_status [/Users/kelton/Sites/narify/test/models/narration_test.rb:21]:
narration is valid without a published status

bin/rails test test/models/narration_test.rb:19

F

Failure:
NarrationTest#test_invalid_without_source_url [/Users/kelton/Sites/narify/test/models/narration_test.rb:27]:
Expected true to not be truthy.

bin/rails test test/models/narration_test.rb:25

Finished in 0.079035s, 50.6105 runs/s, 50.6105 assertions/s.
4 runs, 4 assertions, 3 failures, 0 errors, 0 skips
```

Writing these tests first, I can see how this output should annoy me. For example, "narration is valid without a title" makes me cringe, so now we need to include validations in our model file.

{% highlight ruby %}
validates :title, :published, :source_url, presence: true
{% endhighlight %}

Now running our test in the terminal, `rails test test/models/narration_test.rb`:

```
Running via Spring preloader in process 9112
Run options: --seed 12134

# Running:

....

Finished in 0.044366s, 90.1591 runs/s, 157.7785 assertions/s.
4 runs, 7 assertions, 0 failures, 0 errors, 0 skips
```

In my mind, the test we just wrote only wraps around the Narration model definition. So "by definition", our model should pass these "definition" tests. Before moving out of the definition, let's address the unruliness by abstracting away some of the repetitive stuff. Let's take the local variable `narration` defined in our `valid` test, and assign in to an instance variable in a `#setup` method. Then we'll use that instance variable in subsequent tests by setting the invalid-attribute-to-be to `nil`. The `refute` and `assert_not_nil` statements then just modify their `narration` local variable to be the `@narration` instance variable. Here's my `narration_test.rb` file with changes:

{% highlight ruby %}
require 'test_helper.rb'
class NarrationTest < ActiveSupport::TestCase
  def setup
    @narration = Narration.new(title: 'How to Test Rails Models with Minitest', published: true, source_url: 'https://semaphoreci.com/community/tutorials/how-to-test-rails-models-with-minitest')
  end

  test 'valid narration' do
    assert @narration.valid?
  end

  test 'invalid without title' do
    @narration.title = nil
    refute @narration.valid?, 'saved narration without a title'
    assert_not_nil @narration.errors[:title], 'no validation error for title present'
  end

  test 'invalid without published status' do
    @narration.published = nil
    refute @narration.valid?, 'saved narration without a published status'
    assert_not_nil @narration.errors[:published], 'no validation error for published status present'
  end

  test 'invalid without source_url' do
    @narration.source_url = nil
    refute @narration.valid?, 'saved narration without a source_url'
    assert_not_nil @narration.errors[:source_url], 'no validation error for source_url present'
  end
end
{% endhighlight %}

And I ran the test again and got zero failures and zero errors. But if we add more attributes to our Narration model, we'll need to dip into this test (and possibly other tests that use this test data) to manage the growing model. Let's abstract one step further using "fixtures". A newly generated Rails app and model would have this folder/file handy for us. Let's create it manually. `test/fixtures/narrations.yml`. Note the new fixture folder and the plural narrations Yaml file. Let's abstract now into this Yaml file:

{% highlight yaml %}
valid:
  title: 'How to Test Rails Models with Minitest'
  published: true
  source_url: 'https://semaphoreci.com/community/tutorials/how-to-test-rails-models-with-minitest'
{% endhighlight %}

And now only modify the `#setup` method in our test file:

{% highlight ruby %}
def setup
  @narration = narrations(:valid)
end
{% endhighlight %}  

So we're still going to be using the `@narration` instance variable, but we're assigning it a value that pulls (by convention) from the fixture file called `narrations` with the argument `:valid` defined therein. Tests pass still. That's all for this first run at testing with Rails using the default Minitest, even though we never actually referenced "Minitest". It's "under the hood" as they say, right?
