---
date: 2016-03-20T08:56:00.000Z
lastmod: 2017-07-30T08:58:05.000Z
title: Life is Short. Run Tests in Parallel
draft: false
slug: life-is-short-run-tests-in-parallel
tags: ["Testing","Parallel"]

description: 
---

If you are doing Rails, chances are that you have [RSpec](http://rspec.info/) and [Cucumber](https://cucumber.io/) tests. They are great tools to give you the confidence that your software is working as expected. But as project grows, you may find that your tests are getting slower and slower, expecially the Cucumber tests. It not only slows down the speed of your local development, but also your Continuous Integration pipeline (Test is probably the first stage in your CI). We don't want to waste time waiting tests to finish in either local or CI. We want to develop and integrate faster. Here is how.

### 1. Install parallel_tests

[parallel_tests](https://github.com/grosser/parallel_tests) is a great gem to run Test::Unit, RSpec, Cucumber and Spinach tests parallel on multiple CPU cores.

Update your Gemfile

    group :development do
    	gem "parallel_tests"
        # other gems...
    end
    

Update config/database.yml

    test:
    	database: db/test<%= ENV['TEST_ENV_NUMBER'] %>.sqlite3
    

If you are not using sqlite3 for test db, update it accordingly.

Each CPU core will run its share of tests using its own test db. So if you have 8 CPU cores, you will see 8 sqlite3 files generated in  `db` when running tests.

Then

    bin/bundle install
    

### 2. Use Binstubs to Run Parallel Tests

If you don't know what is binstubs and why you should use it for all commands, take a look at [this](https://robots.thoughtbot.com/use-bundlers-binstubs) and [this](https://github.com/rbenv/rbenv/wiki/Understanding-binstubs).

Generate binstubs only for `parallel_tests`

    bin/bundle binstubs parallel_tests
    

It will generate 4 binstubs in `bin`

- parallel_cucumber
- parallel_rspec
- parallel_spinach
- parallel_test

You don't need to keep all of them. Only keep the ones that you need. In my case, I only use RSpec and Cucumber. So I deleted `parallel_spinach` and `parallel_test`. Add the rest to version control.

### 3. Run Tests in Parallel

Run all specs in parallel

    bin/parallel_rspec spec
    

Run all features in parallel

    bin/parallel_cucumber features
    

This will launch N browsers, where N is the number of your CPU cores.

Be amazed by how long it takes to run all tests now. Theoretically, the test running time can be reduced to `T/N`, where `T` is the time to run in serial and `N` is number of CPU cores. But that's not achievable in practice because the number of tests may not be split evenly among the processes. And even if they are split evenly, they may not take equal time to run. Besides, there are overheads to coordinate the processes and collate results. However, it's very easy and common to achieve 20% ~ 50% reduction in test running time, which can potentially save 5 ~ 10 mins in a medium size project. In a long run, it's a huge time saving.

### 4. Tips

Not all tests are equal. Some tests are for must-have features and some are for nice-to-have features. Some tests fail more often than others. To get feedback faster, it's a good practice to group  tests into categories and run them in order, e.g.

    bin/parallel_cucumber features -o '-t @smoke'
    bin/parallel_cucumber features -o '-t @flaky'
    bin/parallel_cucumber features -o '-t ~@smoke,~@flaky'
    

The above example will

1. Run all smoke features. More on [smoke tests](https://en.wikipedia.org/wiki/Smoke_testing_(software))
2. Run all flaky features
3. Run the rest

This way, it avoids running the smoke tests and flaky tests at the very end.

### Summary

It's quite easy to setup parallel tests. It not only speeds up your local development, but also CI pipeline. And the time saving is tremendous in the long run.

Life is Short. Run Tests in Parallel!
