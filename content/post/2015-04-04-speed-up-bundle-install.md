---
date: 2015-04-04T08:36:00.000Z
lastmod: 2017-07-30T08:37:55.000Z
title: Speed up bundle install
draft: false
slug: speed-up-bundle-install
tags: ["bundler","Ruby"]

description: 
---

**TL;DR** Next time, when you need `bundle install`, do `bundle install --jobs X`, where **X** is the number of your machine cores. It will save you huge amount of time.

[Some blog](http://archlever.blogspot.sg/2013/09/lies-damned-lies-and-truths-backed-by.html) suggests that setting the number of jobs to **X-1** is statistically better than **X**. But that is not true anymore, at least with bundler >= 1.7.12. To prove this, I tested it on a large Rails project ([discourse](https://github.com/discourse/discourse)) on my Mid 2012 MBP with 2.6 GHz Intel Core i7. Below is my test result:

- `bundle install` takes **13 mins and 45 seconds**
- `bundle install --jobs 7` takes **3 mins and 51 seconds**
- `bundle install --jobs 8` takes **3 mins and 47 seconds**

Although the difference between 7 and 8 jobs is not big, using more jobs is still faster. [Other benchmark](http://blog.mroth.info/blog/2014/10/02/rubygems-bundler-they-took-our-jobs/) has similar result.

To save some typing, I would recommend configing this option globally on your machine, e.g. `bundle config --global jobs X`. After this, you only need to do `bundle install`

Life is too short. Speed up `bundle install`.
