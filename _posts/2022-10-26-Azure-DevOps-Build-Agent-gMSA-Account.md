---
layout: post
category: 
- DevOps
- Sysadmin
title: Use a Group Managed Service Account for Azure DevOps VSTS Agents
comments: true
summary: Set up Group Managed Service Accounts with Azure DevOps Self-Host Build Agents
---

# Azure DevOps and gMSAs
I realized I'm much more likely to make a short form post. After telling myself "I'll do it next week" for weeks now, I'm just going to write up a little tip I've found success with in Azure DevOps.

Some background, at my organization we use the On Premise version, which gives me some nice control over build virtual machines. Additionally, the VSTS agent sometimes needs to access things on our network shares. There is no accountability if this operation is performed by a BUILTIN Local Network Service account, which the agent uses by default.

Instead, we are going to use a Group Managed Service Account (gMSA) to accomplish our task. This will allow you a couple of things:

* Access control to shares and folders on the network
* Accountability - Know which resource is accessing which object

This post will take you through setting up a gMSA account first, then show you how to set up the VSTS agent with said account.

## Let's get started

### Get Personal Access Token (PAT)


* Go to your Azure DevOps Instance > Select your profile from upper right corner > Choose Security

![Security](\assets\2022-10-26\Security-Setting.png)


* Select New Token > Fill out information in column (We're choosing full access for now)


![Generate Token](\assets\2022-10-26\Generate-Token.png)


* Copy the token to somewhere safe, you'll never see it again. If you lose it you'll have to create a new token.

### Download and configure agent

* Go to Collection Settings > Agent Pools > New Pool or Add to Existing (based on goal)


* Click the pool to open, then choose New Agent


![New Agent](\assets\2022-10-26\New-Agent.png)


* Follow the instructions that pop up on the desired server or PC. We are doing the default install for now.
	- URL: https://ems-azdevops.emscorporate.com/DefaultCollection/
    - You'll need to specify the collection if not in the default
    - Specify Pool Name: In this case, "Example Pool"
    - See below powershell screenshot for example
    - Most defaults are fine.


* Choose to run it as a service. Also just run as NT authority right now

![Default Install](\assets\2022-10-26\Default-Install.png)
![Example](\assets\2022-10-26\Example.png)



* All done. Agent is configured

### Set up gMSA for use with Azure DevOps agent service

By default during installation, it is registered with the NT Network Service

![Default Service Log On](\assets\2022-10-26\Default-Service-Logon.png)


* All of this will be done with Powershell. You must have DA or similar
	- You're going to want to get the DN for the computer you are associating this account with.
    - Replace my domain information with your own. If you copy and paste this code it will not work.
    - Be sure to select the distinguishedName (DN) from Active Directory for this. More info can be found at the link below:
    - <a href="https://learn.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview">Group Managed Service Accounts Overview</a>


``` Powershell
New-ADServiceAccount SVCExampleAcc -DNSHostName SVCExampleAcc.exampleDomain.com

Set-ADServiceAccount -Identity SVCExampleAcc -PrincipalsAllowedToRetrieveManagedPassword "CN=WIN10TESTVM,OU=Win10,OU=Desktops,OU=Computers,DC=fdc,DC=prv"

# Test it
Get-ADServiceAccount -Identity SVCExampleAcc -Properties PrincipalsAllowedToRetrieveManagedPassword

```

* If you need to run a command prompt to try things as your service account, use psexec.exe from windows internals.


``` CMD
psexec.exe -u domain\svcaccname cmd.exe
```


* When it prompts for password, just leave it blank. Once in, type ``` Whoami ``` to confirm identity.


Below is us logging into a command session as the service account created earlier in this guide

![Example psexec](\assets\2022-10-26\Example-psexec.png)

Below is an example of the operation on a PC that doesn't have the retrieve password attribute

![Failed login](\assets\2022-10-26\Failed-Login.png)


* To change the Azure Pipleline account, add the account you created to the service


![gMSA Service](\assets\2022-10-26\gMSA-Service.png)


* Leave the password field empty and click apply.


![Completed](\assets\2022-10-26\Completed.png)


* Restart the service and it should function.

