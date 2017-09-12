---
layout: post
title:  "Tutorial Deviations"
date:   2017-09-11 18:42:00 -0500
categories: rails
published: true
---
I've been going thru Lynda's Rails 5 Essentials by Kevin Skoglund today and finished chapter 4. This Lynda course is part of a class I'm in at Dallas Makerspace on creating Rails & React apps. Here's [my repo for the simple content management system](https://github.com/chemturion/content-management-system).

I've been practicing "atomic commits", and today got up to 40. [https://github.com/chemturion/content-management-system/commits/master](https://github.com/chemturion/content-management-system/commits/master)

I also tried the challenge at the end of chapter 4 and went a step further to create the model associations in addition to the foreign key definitions in the database migrations. The default migration method is `#change`, and Skoglund has us splitting this method into `#up` and `#down`. Unfortunately I got careless in my Section migration and didn't change my `drop_table :sections` call to the `#down` method. So I had [TWO `#up` methods](https://github.com/chemturion/content-management-system/commit/4605f9791c2c23119af6e5a45b6dd200728b47c9)! So I was able to `rails db:migrate` and `rails db:migrate:status` no problem, but my schema didn't have a Sections table. That was a fun 20 minutes of troubleshooting. I can see how tests would help point out this deviation.

Kevin also created foreign keys a bit different. In his Section migration, inside the `#up` > `create_table...` he has `t.integer :page_id`, and outside of `create_table...` he has `add_index "sections", "page_id"`.

I went a different route: `t.belongs_to :page, foreign_key: true, index: true`. So we ended up with different schemas. His page_id was an integer, whereas mine resolved (after the migration) to a bigint. His t.index had an additional option at the end: `using: :btree`. I have no idea what this means, and just making the observation at this point.

**Onward!**
