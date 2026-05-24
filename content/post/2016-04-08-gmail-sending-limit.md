---
date: 2016-04-08T08:58:00.000Z
lastmod: 2017-07-30T09:00:25.000Z
title: Gmail Sending Limit
draft: false
slug: gmail-sending-limit
tags: ["Gmail","Limit"]

description: 
---

Many people use Gmail as personal emails. Companies use [Google Apps for Work](https://apps.google.com/). Everyone is happy using it. Not until you hit its limits.

I recently learnt [its limits](https://support.google.com/a/answer/166852?hl=en) (shown below) in a hard way.

![Screen-Shot-2016-04-08-at-3-33-16-PM](/content/images/2017/07/Screen-Shot-2016-04-08-at-3-33-16-PM.png)

I will hightlight the most restrictive constraints from the above:

- You can send max 2000 emails per day from one account, **500** for trial accounts
- 3000 unique recipients per day, **500** for trial accounts

From what I understand, normal Gmail accounts have the same limits as trial accounts. What are trial accounts? When you sign up [Google Apps for Work](https://apps.google.com/), all users are in trial period for 30 days. And it's free during this period. After trial, you have to pay ~$5/user/month.

The catch is **What if you want to send > 500 emails during trial period?** This is the problem that I had.

According to [Google](https://support.google.com/a/answer/166852?hl=en),

> At the end of your free trial period, your sending limits will be automatically increased when your domain is cumulatively billed for at least $30 USD (or the same amount in your currency). To expedite this process, you can manually prepay this amount as suggested here. It will take up to 48 hours to upgrade your sending limits after you submit the manual payment. Note: while you're still in your trial period, your sending limits will not be increased.

So I signed up Google Apps and setup Email with my custom domain. Made manual payment of $31. After ~550 emails, it stopped with following erorr

    Daily user sending quota exceeded. s197sm16092990pfs.62 - gsmtp
    

I contacted online chat support. Surprisingly, I got reply immediately. It turned out that even though I had made the payment of $31, it **DOES NOT** automatically end my trial period. So the limit of 500 emails still applies. The solution is  Google support staff sent me a new Google Apps contract via email. Unpon agreeing to its T&C, it effectively expired my trial period and changed my account to paid subscription. I was told that it may take 48 hours for the new limit of 2000 emails/day to be effective.

So far, contacting Google support seems to be the only way to upgrade from trial limits within trial period. I haven't found any setting that I can do it on my own, which is weird. If customers don't want to enjoy the free month, they should have the capability to do so.
