---
layout: post
title: Windows Firewall Service Fails to Start, Office 365 MFA Problems Abound, and How They Are Related
categories: Windows
comments: true
tags:
  - Windows 10
  - Windows 1709
  - Windows 1909
  - MPSSVC
  - Windows Firewall
  - Windows Defender
  - Crashing Appx
  - Appx
  - Registry
  - Troubleshooting
  - Ivanti AV
  - Kaspersky
  - Fix
  - Office 365
---

Do you ever work on a problem that seems to have no explanation? If you've made it to this post I'm sure you have. I struggled for a couple of weeks with an issue that plagued about 1/3 of our enterprise Windows PCs. No one seemed to know what was wrong or how to even begin to troubleshoot the issue. 

That's where I come in. But first, some background.

## Background

Just like every organization should, we began to implement MFA in our Office 365 environment. As with any project like this, some pain on the change frontier can be expected. The first few days of the implementation went well. A couple of minor hiccups here and there, but nothing major to speak of. Until the team began to reach more end users. We began to encounter some issues with the MFA prompts in the desktop versions of the Office Suite of applications. Some symptoms included:

* A blank box where the MFA prompt should appear would flash on the screen and quickly disappear
* The MFA would never prompt, leaving the user not logged in. Effectively dead in the water.
* The Office application would seem to freeze, with an invisible box outside of the window context.

The MFA issues would leave users dead in the water. Each machine would take about an hour of trial and error to get the user back to a working state. We could disable MFA once we discovered issues, but this still didn't immediately fix the damage caused. Sometimes Outlook would not connect back to Exchange. Sometimes the office products would not activate at all. Either way, non-functional users is a no go.

That isn't all though. There were just the symptoms related to MFA. Little did I know that there were many more minor things that had been overlooked for a while. "Technical Debt" is a good way to put it. Below are some other oddities that were stemming from this problem.

* Outlook wouldn't allow users to preview Excel attachments, PDFs, Word docs
* Windows Security would not open from the settings menu. The window would flash open for a second and quickly close. Much like the MFA prompt window mentioned above
* Native Windows applications such as Calculator, Calendar, Photos, etc would display the same behavior as the security center and MFA prompts
* Excel documents would not open and would display strange error messages with no particular reason for them.

![Excel Error](\assets\2021-5-4\Excel Error Message - mpssvc related.png)


## So What's Going On?

Like any good sysadmin, I started Googling error messages, symptoms, the works. I'm going to paste some links below that were similar to my issue, all of which didn't resolve the problem ultimately.

<a href="https://answers.microsoft.com/en-us/windows/forum/windows_8-performance/windows-firewall-service-terminated-with-access-is/5d384fa8-a609-4aa2-9b5d-b1f9122baad1">MSFT Forum - Windows Firewall service terminated with Access is denied Error message</a>

<a href="https://www.reddit.com/r/sysadmin/comments/an2cze/adding_mfa_to_office_365_client_not_prompting_for/">Reddit - Adding MFA to Office 365 client, not prompting for modern authentication
</a>
<a href="https://techcommunity.microsoft.com/t5/office-365/not-being-prompted-for-mfa-on-outlook-365-desktop-even-with/m-p/1241681">MSFT Tech Community - Not being prompted for MFA on Outlook 365 desktop, even with Modern Auth enabled?‎</a>

<a href="https://windowsreport.com/fix-windows-defender-wont-turn-on/">FIX: Windows Defender won’t turn on Windows 10</a>

<a href="https://appuals.com/how-to-fix-windows-defender-firewall-error-code-0x6d9/">How to Fix Windows Defender Firewall Error Code 0x6d9?</a>

<a href="https://social.msdn.microsoft.com/Forums/azure/en-US/951e5d39-d3d2-472d-b2e7-64e05ea3ae96/outlook-account-setup-and-microsoftaadbrokerpluginexe?forum=WindowsAzureAD">Outlook Account Setup and Microsoft.AAD.BrokerPlugin.exe</a>

<a href="https://www.kiloroot.com/modern-authentication-issues-with-office-365-fixed-dont-just-disable-azure-active-directory-authentication-library-adal-instead-fix-it-with-this/">Modern Authentication Issues with Office 365 – FIXED – Don’t Just Disable Azure Active Directory Authentication Library (ADAL) – Instead… Fix It With This!</a>


Instead of finding the root cause of the issues, there were lots of workarounds in place. Some of these include the EnableADAL registry key being created and set to 0.
This isn't secure, nor a good practice going forward. The same for DisableADALatopWAMOverride = 0.

These reg keys were all over the place in our environment. No wonder the rollout didn't go smoothly.

### Microsoft AAD Broker Plugin, Registry Keys, and Why They're Important

That last link by kiloroot is the one that really helped me discover the root cause. This didn't ultimately end up being the issue, but pointed me in the right direction. Here's a quote of some more symptoms:

<cite>A colleague of mine recently solved one of the biggest pain points I have dealt with regarding Office365 – that is, Microsoft’s seemingly hit-or-miss modern authentication.</cite>


>Symptoms look like this:
> 1. Outlook client can’t connect and/or authenticate for end-users
> 2. Turning on Azure MFA for an end-user ruins their life (and yours) because all office applications, teams, etc. break.
> 3. Admins have an impending sense of “dread” when setting up systems for new users because 80% of the time they are going to spend hours sorting out the above issues.
> 4. You call Microsoft Support complaining of these issues and they are eventually stumped and tell you to rebuild the desktop/laptop from scratch… great for end-users that deal with this issue 1 year into the job and rather like their systems as-is… -or- MS Support tells you to pop a registry key into the end-user’s system which just disables Modern Authentication all together – which may fix Outlook but leaves many many other things broken…

> If any of that sounds familiar, I highly recommend you read the article he published on linked-in…. this is THE SILVER BULLET to end your Microsoft Authentication woes:<a href="https://www.linkedin.com/pulse/solving-modern-authentication-issues-office-365-chris-leet/"> Solving Modern Authentication Issues with Office 365</a>


This is a good resource that is already written so I don't have to re-invent the wheel. I'd really recommend reading Chris's Linkedin article covering the AAD broker plugin, as it was exactly what was going on with us. Unfortunately, this still didn't solve our problem. So let's check that out now. We're almost there, I promise!

## Windows Firewall, AppX packages, and Pesky Registry Permissions

At this point it has been two weeks of work with nothing to show for it. Our rollout has stopped in its tracks, with the fear of managers breathing fire if employees are down while trying to implement this security feature. Word has spread and it isn't looking good. I'm at the end of my ideas.

Behold, Windows Internals. I decided to go and buy the first edition. I'm mad I can't solve this on my own and want to know more, so why not?

This ended up being a good decision. I was able to see how AppContainers worked in Windows and how those related to AppX packages such as the new windows apps like Calculator, the AAD Broker Plugin, and even Windows Defender.

### Sysinternals - In the Weeds

Where do I even start to look?

#### Process Explorer

I started off by using the search feature in process explorer to find the MPSSVC.dll and registry keys that were active with this process.

![Process Explorer Search](\assets\2021-5-4\ProcExpSearch.PNG)


Then I looked at the registry key path. The \Parameters\AppCs was of interest to me, after reading how App Containers really dominate the built in Windows features.

![Process Explorer MPSSVC Reg Keys](\assets\2021-5-4\ProcExpmpssvcAppCs.PNG)

The nice thing about Process Explorer is you can find a dll -> Properties -> Explore. It'll take you right to the registry key that controls that service! Woah!

![Process Explorer MPSSVC Reg Keys Explore](\assets\2021-5-4\ProcExpMPSSVCRegKeyProp.PNG)

So let's go there. Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\mpssvc\Parameters\AppCs

![Registry MPSSVC](\assets\2021-5-4\RegMPSSVCAppCs.PNG)

Lo and Behold - This is exactly where the problem is coming from. But we can't see the permissions for the AppCs area on our regular account? Gotta go deeper.

![Registry Can't View Permissions](\assets\2021-5-4\RegMPSSVCAppCsPermissionDenied.PNG)

We need to utilize another systinteranls tool - psexec - to run regedit.exe and SYSTEM.

![PSEXEC](\assets\2021-5-4\PSEXECRunRegAsSys.PNG)

And now you can see it. mpssvc doesn't have read or write access to AppCs at all! This is bad.

![Registry MPSSVC No Permissions](\assets\2021-5-4\ImproperPermissionsMPSSVC.PNG)




## The Fix

Luckily we were able to get a script working for this issue so we could push it to all the PCs that were having that issue. Once we pushed it out all went well with the rollout. As of this writing, we have fully implemented MFA for our Office 365 environment. I'm glad that's over. 

I'll post the script and some helpful links below:

### The Script

Big shoutout to andrewtchilds on Github for the script:

<script src="https://gist.github.com/andrewtchilds/64de720eaf315d6d247d21b0f48d3920.js"></script>

### What Caused This?

This is the most important question to answer. From what I can tell, this had been occuring in minor ways before I arrived to this position. It seems to have stemmed from an issue with Kaspersky antivirus that Ivanti (formerly LANDesk) had. From what I can tell, there was only one obscure article that covered the issue. 

<a href="https://forums.ivanti.com/s/question/0D71B000005GGZPSA4/detail?language=en_US">Landesk EPS - LDAV Kaspersky and Windows 10 Fall update 1709</a>

