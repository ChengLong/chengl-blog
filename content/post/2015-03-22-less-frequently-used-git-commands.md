---
date: 2015-03-22T08:30:00.000Z
lastmod: 2017-07-30T08:34:26.000Z
title: Less frequently used git commands
draft: false
slug: less-frequently-used-git-commands
tags: ["git"]
image: /content/images/2017/07/Git-Logo-1788C.png
description: 
---

From time to time, I find myself or my peers not using git correctly or efficiently. This is largely due to the fact that some git commands are less well known and used. Below are a few commands that could be helpful but less frequently used.

#### git bisect

Suppose you find that commit `c100` is failing your tests. But it was running fine at commit `c67`. You want to know which commit in the range (`c67`, `c100`] introduced the bug.

Of course you can *manually* checkout each commit starting from `c68` and run tests to see if it's broken. But that's *`O(n)`* and tedious. You can do it in *`O(lg(n))`* by using `git bisect`.

    git bisect start
    git bisect good c67
    git bisect bad c100
    

Now git will bisects and checks out a revision for you. You have to answer either `git bisect good` or `git bisect bad`. This process repeats until it finds the *first* bad commit.

This is good but *not* optimal. You can automate this.

    git bisect start
    git bisect good c67
    git bisect bad c100
    git bisect run rspec spec/your/failing/spec.rb
    

Git will take care of the whole process and tell you the *first* bad commit. Note that `rspec spec/your/failing/spec.rb` should be the command to determine whether the commit is good or bad. It can be any command that *"exit with code 0 if the current source code is good, and exit with a code between 1 and 127 (inclusive), except 125, if the current source code is bad"*. Take a look at [this](http://git-scm.com/docs/git-bisect#_bisect_run)

#### git cherry-pick

Put simply, this command let you replay any *existing* commit to wherever you are right now.

Suppose you have this tree

    c1 - c2 - c3 - c4 - c5 - c6 [master]
    	  \
          c7 - c8 - c9 [feature]
    

You only want commit `c8` to go into master, you do

    git checkout master
    git cherry-pick c8
    

This results in

    c1 - c2 - c3 - c4 - c5 - c6 - c8' [master]
    	  \
          c7 - c8 - c9 [feature]
    

Note that commit `c8'` and `c8` have exactly the same change. But they have different SHA.

#### git filter-branch

Suppose someone accidentally checked in a SSH key that can login to your production server. You want to remove the key completely from git

    git filter-branch --force --index-filter \
    'git rm --cached --ignore-unmatch id_rsa' \
    --prune-empty --tag-name-filter cat -- --all
    

The above command will remove the file `id_rsa` from the entire history of every branch and tag. And prune empty commits, if any.

Since this will rewrite history, you have to force push to all branches and tags

    git push origin --force --all
    git push origin --force --tags
    

If your repo is large, `git filter-branch` may take a long time. Consider using [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/). It's much simpler and faster. To do the same thing using `bfg`

    bfg --delete-files id_rsa
    

#### git format-patch

When the build is broken, you can't simply push your commits. But your colleague needs a few commits from you urgently. What do you do? Instead of creating a temporary branch, you can use `git format-patch`.

This command put your commit in an emailable format, i.e. a patch. You can then email/AirDrop/Thumbdrive it to your colleague. For example:

- `git format-patch -2` creates 2 patches from last 2 commits
- you email/AirDrop/Thumbdrive these 2 patches to your colleague
- your colleague applies the patches `git apply [patch-name].patch`

#### git reflog

This command is especially useful after you do `git reset --hard HEAD~3` and realize that it's a mistake.

Fortunately, git doesn't actually discard your commits even if you do a hard reset. Hard reset merely moves HEAD to a different state. You commmits are not lost. All movements of HEAD are recorded in reflog.

    git reflog
    

You will see something like this

    a4f7ad2 HEAD@{0}: reset: moving to HEAD~3
    dcb037d HEAD@{1}: checkout: moving from 44d871f62adaa5a70a to master
    44d871f HEAD@{2}: checkout: moving from test to 44d871f62adaa5a70a4f5
    ...
    
    

It shows the history of how HEAD has changed. This includes actions such as `commit`, `rebase`, `checkout`, `reset`, etc. You can see that the first line
`a4f7ad2 HEAD@{0}: reset: moving to HEAD~3` is the hard reset. To recover from that mistake, simply do:

    git reset --hard HEAD@{1}
    

Note that entries in reflog are *not* stored forever. By default, they will expire in 90 days.
