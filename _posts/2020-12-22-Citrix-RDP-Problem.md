---
layout: post
title: A Simple Citrix Problem
---

Spent some time yesterday working on a Citrix issue that I wasn't sure I could fix. It turns out it was a lot more simple than I thought. I just got a bit flustered when working on it and needed to sleep on it. I'm going to outline what I did this morning to get it fixed up.

Issue: RDP sessions from Citrix Storefront weren't launching like they used to. I'm fairly new to this company, so this is my first time using Citrix. Additionally, I'm unsure how long it had been since they used these specific RDP session links. When clicking on the RDP link box, it either didn't load at all with no explanation, or it popped up this warning box with literally no error information. See below:

![Warning Screenshot](\assets\2020-12-22\XenApp-Warning.png)

After searching the internet I only found on slightly helpful piece of information, even though it was a bit dated. <a href="https://support.citrix.com/article/CTX127051">How to Publish a Remote Desktop with a Custom Configuration: CTX127051</a>

This page went through the creation of and RDP file and how to publish it using Storefront. This help article helped, but not fully.  I'll show you the adjustment I had to make that wasn't that straightforward, at least for me.

Here's the old configuration. As you can see the command and arguments are all on one line.

![Old Configuration](\assets\2020-12-22\CitrixOldPathConfiguration.PNG)






Here's the new configuration. The command needed to be broken out from the single line like it used to be.

![New Configuration](\assets\2020-12-22\CitrixNewPathConfiguration.PNG)








And that's all it took! I believe that it happened because the Citrix instance was upgraded and those links had not been used since. Either way, it's back up and working again. On to the next thing! I hope if someone comes across this it helps.
