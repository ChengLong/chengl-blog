---
date: 2016-12-04T09:23:00.000Z
lastmod: 2017-07-30T09:24:30.000Z
title: Speed Up SSH
draft: false
slug: speed-up-ssh
tags: ["ssh"]

description: 
---

Recently I worked on a project where executing remote commands is very slow to start. This is expected because an SSH connection has to be established first.

    $ time ssh server exit
    ssh server exit  0.04s user 0.01s system 0% cpu 7.901 total
    

It took nearly **8** seconds to ssh into `server`.

That's slow!

After a bit of googling, there is an elegant way to speed this up. Simply put this at the bottom of your `~/.ssh/config`.

    Host *
      Compression yes
      ControlMaster auto
      ControlPath ~/.ssh/sockets/%r@%h:%p
      ControlPersist 4h
      ServerAliveInterval 60
    

What this does is to *try* to share the master connection via a socket with multiple ssh sessions. It falls back to creating a new one if one does not already exist. The socket file is defined by `ControlPath`. `ControlPersist 4h` specifies that keep the master connection in background for four hours after the client connection is closed. `ServerAliveInterval 60` means that the client will wait for 60s to send a message to the server (if no data has been received during this period) to keep the connection alive.

Create sockets directory

    $ mkdir ~/.ssh/sockets
    

Running `time ssh server exit` again won't see any difference. But look into `~/.ssh/sockets`, a socket file is created.

    srw-------  1 user group    0 Dec  4 22:12 user@***.***.***.***:22
    

Try again

    $ time ssh server exit
    ssh server exit  0.01s user 0.00s system 40% cpu 0.027 total
    

Now it takes less than **0.5%** of **8** seconds.

What's interesting is that, this config not only speeds up SSH connections, but also `git`, if you're authenticating with SSH.
