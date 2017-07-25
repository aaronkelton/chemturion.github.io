---
layout: post
title:  "Misleading Errors in Rails - Missing the Haml gem"
date:   2017-07-25 10:00:00 -0500
categories: rails haml beginner error
published: true
---
I've been getting multiple exposure to Rails from different sources in an attempt to saturate my understanding. I'm going through the book, Agile Web Development with Rails 5, by Sam Ruby et al. Unfortunately my app doesn't always behave according the author's expectations. I recently had to just overwrite my files with the source files, which turned all my Haml files back to ERB. As I continued through the section on "Capturing an Order", I first ran `rake haml:replace_erbs`. But when I finished updating my code per the tutorial and read, "We're ready to play with our form. Add some stuff to your cart, and then click the Checkout button. You should see something like the following screenshot." I saw this instead in my browser:

```
ActionController::UnknownFormat in StoreController#index
StoreController#index is missing a template for this request format and variant. request.formats: ["text/html"] request.variant: [] NOTE! For XHR/Ajax or API requests, this action would normally respond with 204 No Content: an empty white screen. Since you're loading it in a web browser, we assume that you expected to actually render a template, not nothing, so we're showing an error to be extra-clear. If you expect 204 No Content, carry on. That's what you'll get from an XHR or API request. Give it a shot.
```

All the Stack Overflow and tutorials suggested that I was missing the `app/views/store/index.html.haml` file, or that I needed to explicitly include in my `store#index` a `respond_to` block because of my terminal's output:

```
Started GET "/" for ::1 at 2017-07-25 10:20:45 -0500
  ActiveRecord::SchemaMigration Load (0.3ms)  SELECT "schema_migrations".* FROM "schema_migrations"
Processing by StoreController#index as HTML
  Cart Load (0.2ms)  SELECT  "carts".* FROM "carts" WHERE "carts"."id" = ? LIMIT ?  [["id", 18], ["LIMIT", 1]]
Completed 406 Not Acceptable in 91ms (ActiveRecord: 0.6ms)
```

The `respond_to` suggestions were a good clue, though. I suspected that this was a Haml issue, so I changed my filename suffix back to `.erb` for the `store/index`, and here's my browser's output:

{% highlight haml %}
%p#notice= notice
%h1 Your Pragmatic Catalog
- cache @products do
  - @products.each do |product|
    - cache product do
      .entry
        = image_tag(product.image_url)
        %h3= product.title
        = sanitize(product.description)
        .price_line
          %span.price= number_to_currency(product.price)
          = button_to 'Add to Cart', line_items_path(product_id: product), remote: true
{% endhighlight %}

So it literally spit out the Haml code. Good— I just concluded that Rails wasn't rendering my Haml file. So what's the complicated fix? I needed to add `gem 'haml'` to my Gemfile. I like these exercises in playing detective; it helps me not take the error messages literally, but instead better understand how they were generated in the first place.
