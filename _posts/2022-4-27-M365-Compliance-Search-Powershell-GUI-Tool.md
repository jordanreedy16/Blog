---
layout: post
categories:
 - Microsoft 365
 - Windows
 - Powershell
 - Development
title: |
     My First Powershell GUI - Email Deletion in M365 Purview
     (Formerly Microsoft Compliance Center)
comments: true
tags: 
 - Microsoft 365
 - Windows
 - Powershell
 - Development
 - Microsoft Purview
 - Microsoft Compliance
 - Exchange Online Powershell
 - EXO V2 Module
 - Powershell
 - Development
---

<div class="message">
 I went to school for traditional computer science. I've only recently started to utilize the many hours of coding I endured in college, but it turns out the real value garnered is being able to learn about functionality rather than simply copy/pasting snippets.

 That being said, this is my first attempt at creating a GUI interface for a powershell script I've created, but also it is only my 3rd or 4th script that is actually useful (and complete lol).</div>
<br />

All of the source code can be found on my Github, here -> <a href="https://github.com/jordanreedy16/M365-Email-Deletion-GUI">M365 Email Deletion GUI</a>

GUI and Terminal View

![GUI and Terminal](\assets\2022-4-27\GUI-Screenshot-w-terminal.png)

## Down and dirty overview
I'm a system administrator, so I prefer the things I write be efficient and usable rather than pretty. I'll leave that to my dev counterparts who can create much more robust and beautiful code than I. Rather, I like to use my time developing for learning how the underlying systems interact and just how much juice I can squeeze out of given program, API, etc.

#### From a high level, here's what this bad boy does:
1. Connects you into the Security & Compliance Center Powershell. This is a module in Exchage Online Powershell, which you'll need to have imported to use this application.
2. Takes in information, in my case a ticket number, from email address, to email address, the date received, the subject, if it had an attachment, and what that attachment name is, or the extension of one. All this info helps to improve search accuarcy, as we don't want to be deleting large swaths of email.
3. Review the information and search for the emails in question. It'll return how many it found. Compare that to how many emails you expect to delete.
4. Delete the emails. This is limited to 25 in order to prevent accidental mass deletion. I think Microsoft also has a 1,000 email limit in deletion, but less is better.
5. Success!

### Required Module

<a href="https://docs.microsoft.com/en-us/powershell/exchange/exchange-online-powershell-v2?view=exchange-ps#install-the-exo-v2-module
">Exchange Online Powershell (EXO V2 Module) Install Instructions</a>

You'll probably need to tell Powershell to accept remotely signed scripts
```powershell
Set-ExecutionPolicy RemoteSigned
```

Simply put, run this command in your Powershell window.
```powershell
Install-Module -Name ExchangeOnlineManagement
```

## Usage

### Create the search
So in the background, all this really does is grab the information from you and creates a search out of it. If you've ever used the web portal, you kind of know how this goes. Here's the portal: <a href="https://compliance.microsoft.com/homepage">Microsoft Compliance Center (now Microsoft Purview)</a>

When I do this, I do deletions by ticket number. That way if I need to reference back from our internal ticketing system, I can.

Here is an example:

![Example1](\assets\2022-4-27\Example-Email-Redacted.png)

When you click the search button, it will create the search in M365 Compliance.

![Example1-SearchCreated](\assets\2022-4-27\Search-Created.png)

<br />

Here is the created search in M365
![Example1-SearchCreated](\assets\2022-4-27\Content-Search-View.png)

### Run the search
Now to run the search, all you'll need to do is click "Start Search".
I'm not a good programmer, so the window freezes while the search occurs. A side effect of running a powershell GUI application all running in one thread. 
(I know I could use runspaces, but I just couldn't figure it out this time. Maybe on the next one, or if I get the desire to rework this application.)

Here is the search inside M365 Compliance.
![Example1-SearchStarting](\assets\2022-4-27\Content-Search-Starting.png)


The GUI is frozen at this point. A dialog box will pop up once finished.
![Example1-SearchStarting](\assets\2022-4-27\GUI-Search-Frozen.png)

Now wait a while. It is sometimes done in a couple minutes, other times it takes about 5 minutes.

When it is done and finds matching emails, it will prompt for confirmation before deletion.
![Example1-SearchCompleted](\assets\2022-4-27\Attention-Dialog-Deletion.png)

Here is how it looks it M365
![Example1-SearchCompleted](\assets\2022-4-27\Content-Search-Completed.png)

Now that we have items returned, you can either confirm the emails found inside the M365 Compliance Center, or just delete them if you're confident.

Clicke the "Review" button. The email will eventually load. It takes a couple minutes.
![Example1-SearchReview](\assets\2022-4-27\Content-Search-Review.png)

There it is! That's the email I want to delete.

![Example1-SearchSample](\assets\2022-4-27\Content-Search-Samples.png)

That's about all there is to it. If you have any questions or comments let me know below. Also, there is always room for improvement so if you have any contructive feedback it is always welcome.