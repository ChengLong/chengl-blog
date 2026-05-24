---
date: 2017-07-20T09:32:00.000Z
lastmod: 2017-07-30T16:14:36.000Z
title: Multirepo vs Monorepo
draft: false
slug: multirepo-vs-monorepo
tags: ["monorepo","multirepo"]

description: 
---

Here's a conversion I keep hearing recently:

> A: Let's put all our projects in one repo.
> 
> B: Why?
> 
> A: Because Google and Facebook do monorepo.

Whenever I hear this, I'm very tempted to show this picture.

![JohnFrumCrossTanna1967-1](/content/images/2017/07/JohnFrumCrossTanna1967-1.jpg)

Finding the origin of this picture is left as an exercise for the readers. On a more serious note, I want to write down my thoughts on multirepo vs monorepo.

## What is multirepo?

One project one repository. Each project is an independent working unit. It can be a mobile app, frontend app, backend service or standalone CLI app.

- Each project has full autonomy to manage its evolution and deployment. There should be little to no coupling between projects. If projects depend on each other, the coupling between projects is API contracts, nothing else.
- Each project manages dependencies on its own. Common library is in a repo of itself. Projects that depend on it can use any version of that library that they deem fit. It can be argued that sharing code is also introducing coupling. And it may result in long tail of maintenance of old libraries. Anyway, [Managing dependencies is hard.](https://medium.com/netflix-techblog/towards-true-continuous-integration-distributed-repositories-and-dependencies-2a2e3108c051)
- Engineering teams are decoupled and can work on different projects in parallel without stepping on each other's toes.
- [Deployment Pipeline](https://martinfowler.com/bliki/DeploymentPipeline.html) can be easily setup for each project.
- Access control can be applied at project level.

This repo structure is how most open source projects are run. And it's also probably what most developers are familiar with. Besides, it plays nicely with [microservices architecture](https://martinfowler.com/articles/microservices.html).

## What is monorepo?

One monolithic repo that contains everything. Literally, everything.

- All projects (regardless they are related or not) and their dependent libraries, including 3rd party code that are not written by you nor your colleagues, live in one single repo.
- There is one and only one verison of each dependency in the entire repo, which is the latest (*HEAD* in git terminology). Whenever a dependency needs to be updated, the update should be done for all projects depend on it and make sure that all projects still work. So the repo should always be in a consistent state. At any commit, all projects should work.
- [Cross-project changes is easier.](http://danluu.com/monorepo/) Large scale refactoring is easier and can be done in one single atomic commit.
- Extensive code sharing.
- Everyone can see all code.

## How to choose?

If you are in a two-man startup, close this page now and keep working on your monolith. The choice between multirepo and monorepo is irrelevant to you. This question is only relevant when your company is operating at scale, i.e. >100 developers.

Given the distinct characteristics of multirepo and monorepo, how to choose one over another? I think there are two main factors to consider, tooling and culture.

### Tooling

In monorepo, running build is not as trivial as multirepo. You probably don't want to run tests and builds for all projects since that's just unnecessarily wasting time and computing resources. So the first thing to figure out is, given a change with one or more commits, which project(s) should build and what tests should run. And in order to figure this out, it's necessary to have acyclic directed graph (DAG) of dependencies for all projects. When a change is submitted, it's checked against the DAG of dependencies to see which projects are affected. All affected projects are possible to break, so tests are run only for these affected projects and their transitive dependents. Good news is that Google has open sourced their build tool [bazel](https://bazel.build/) and Facebook has something similar called [buck](https://buckbuild.com/). While in multirepo, this problem doesn't exist because there is no need to figure out which project to build. Whenever a change happens to a project, that project's [deployment pipeline](https://martinfowler.com/bliki/DeploymentPipeline.html) is triggered.

Source code version control is another tooling challenge imposed by monorepo. It's well known that git is bad at scaling. So is mercurial. [Quoting Linus Torvalds](http://git.net/ml/git/2009-05/msg00051.html)

> Git fundamnetally never really looks at less than the whole repo. Even if you limit things a bit (ie check out just a portion, or have the history go back just a bit), git ends up still always caring about the whole thing, and carrying the knowledge around.
> 
> So git scales really badly if you force it to look at everything as one *huge* repository. I don't think that part is really fixable, although we can probably improve on it.

Although [sparse checkout](https://schacon.github.io/git/git-read-tree.html#_sparse_checkout) and [shallow clone](https://schacon.github.io/git/git-clone) may alleviate the scaling problem, it's not a sustainable solution to large organizations. [Some anecdote](http://permalink.gmane.org/gmane.comp.version-control.git/189776) suggests that the practical limit of git is 15GB of `.git` directory. This is probably why [Microsoft invented GVFS](https://blogs.msdn.microsoft.com/bharry/2017/02/03/scaling-git-and-some-back-story/), [Facebook chose to patch Mercurial](https://code.facebook.com/posts/218678814984400/scaling-mercurial-at-facebook/) and [Google builds Piper](https://cacm.acm.org/magazines/2016/7/204032-why-google-stores-billions-of-lines-of-code-in-a-single-repository/fulltext). The point is, if your organization decides to go monorepo, think carefully about what version control to use.

Large scale refactoring in monorepo doesn't come for free. It requires [dedicated](http://www.hyrumwright.org/papers/icsm2013.pdf)[tooling](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/41876.pdf) support.

Setting up deployment pipeline is complicated in monorepo. In multirepo, it's straightforward to have one project one pipeline. But in monorepo, one possible way is to have the first stage to figure out relevant projects and then trigger child pipelines for each relevant project. And each child pipeline may trigger other pipelines according to the DAG dependency graph. From what I see, the only off-the-shelf Continuous Delivery (CD) tool in the market that supports pipeline [fan-in and fan-out](https://www.gocd.org/getting-started/part-3/#fan_out_fan_in) is [GoCD](https://www.gocd.org/). Other CD solutions in the market have very simple pipeline modeling. They are designed for multirepo, not monorepo. For example, there isn't an elegant solution for [monorepo in GitLab](https://gitlab.com/gitlab-org/gitlab-ce/issues/18157) after one year, neither is [Travis CI](https://github.com/travis-ci/travis-ci/issues/3540).

In short, for multirepo to work, open source and commercial tools in the market are most likely sufficient. But for monorepo, depending on the scale, it may require high tooling investment.

### Culture

Multirepo and monorepo not only have different tooling requirments, but also varying engineering culture and philosophy. Multirepo values decoupling and engineering velocity, while monorepo favours standardization and consistency. It's all trade-offs. Whichever approach a company takes is a reflection of the company's culture. Netflix favours [Freedom & Responsbility](https://www.slideshare.net/reed2001/culture-1798664/2-Netflix_CultureFreedom_Responsibility2) so it prefers mutlirepo. And Google values consistency and code quality so it prefers monorepo. What's important here is to pick one approach that fits your organization's engineering culture, rather than fitting your organization's engineering culture to a certain repo structure.

## Conclusion

Choosing multirepo or monorepo is not trivial. There is no single absolute right or wrong answer. Companies like Amazon and Netflix are living evidence that multirepo at large scale works. On the other hand, companies like Google and Facebook are living evidence that monorepo at large scale also works. Each approach has its own set of principles and practices to follow. Each approach also has its own challenges. Deciding between the two boils down to tooling and culture. Whichever approach an organization takes should be backed up by a list of solid reasons why one is preferred over the other in that organization, not *Because Google and Facebook do monorepo.* That's cargo cult engineering. And [You Are Not Google](https://blog.bradfieldcs.com/you-are-not-google-84912cf44afb).
