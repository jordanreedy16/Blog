---
layout: post
categories: Linux Security
title: How to Install the RSA PAM Agent on Ubuntu 20.04 (64-Bit)
---

<div class="message">
  Are you setting up multi-factor authentication with RSA? Ubuntu is now a supported OS, so let's do it!
</div>

As of PAM Agent version 8.1.2, installing the RSA SecurID authentication agent on Ubuntu versions 18.04 and 20.04 are supported. The installation is similar to that of SUSE linux. We recently had a requirement to add multifactor authentication to all of our servrs, so I installed this on all of them recently. It works very well so far. 

## Prepare to Install

I'm only going to install the agent on a single server at a time, as our environment isn't that big. You can do a bulk install if you want, but I won't cover that here.
We'll also be using the UDP protocol for authentication, not the REST protocol.

There are some requirements we need to get in order before we begin the install process.

Required files:
* sdconf.rec - This file is specifies how the agent communicates with the primary and replica appliance instances by IP address.

    In the console go to Access -> Authentication Agents -> Generate Configuration File

    ![Generate sdconf.rec](\assets\2021-5-6\generate_sdconf.PNG)

    ![Download sdconf.rec](\assets\2021-5-6\save_sdconf_fromzip.PNG)

* Server.cer - This is the certificate from the authentication manager server that allows agent auto-registration

    In the console go to Access -> Authentication Agents -> Download Server Certificate File
        
    ![Server Certificate](\assets\2021-5-6\server_certificate.PNG)

* Add a new Authentication Agent to RSA Security Console

    In the console go to Access -> Authentication Agents -> Add New

    Add the IP address or DNS name of the server here

    ![Add Agent](\assets\2021-5-6\add_agent.PNG)


## Installation and Configuration

Now that we have all of the files we need to continue, let's get to installing.

First thing we need to do is transfer the files to the server you're working on. Use whatever file transfer process you like, personally I've been using WinSCP. Pick your poison.

1. We'll need to make the directory where the configuration files we downloaded earlier will live. We'll be using default locations for this guide. Directory name is /var/ace

    ```bash
    mkdir /var/ace
    ```


2. We need to move sdconf.rec and server.cer to this folder.

    ```bash
    mv sdconf.rec server.cer /var/ace
    ```


3. Create a file named sdopts.rec

    ```bash
    vi /var/ace/sdopts.rec
    ```

    Add line: CLIENT_IP=x.x.x.x
    Replace x.x.x.x with the IP of the server you're installing on. In our case, CLIENT_IP=10.15.10.150

    Save the file

4. Now let's edit the sshd_config file. Below are the parameters we need to change:

     ```bash
    vi /etc/ssh/sshd_config
    ```

    UsePAM - Yes
    PasswordAuthentication No
    ChallengeResponseAuthentication Yes

     ```bash
    systemctl restart sshd.service
    ```


5. Now we actually get to install the agent!

     ```bash
    tar -xvf PAM-Agent_v8.1.3.139.04_19_21_01_39_13.tar
    cd PAM-Agent_v8.1.3.139.04_19_21_01_39_13
    sudo ./install_pam.sh
    ```
    I use all of the defaults going forward. 

    * Accept the EULA. 
    * Enter 0 to choose UDP protocol. 
    * Hit enter to use /var/ace as the default directory. 
    * Hit Enter to use /opt as the PAM agent install directory


6. Once the agent is installed, we need to add an option in the /etc/pam.d directory. We use MFA on SSH, so we'll be configuring the SSH file in the pam.d directory

    We want to add the line at the end of the sshd file! I had to do a bunch of trial and error on this one.

    ```bash
    cd /etc/pam.d
    sudo vi sshd
    ```
    Add line:

    ```bash
    auth required pam_securid.so
    ```

With all that configuration done, you should be good to go. The good news is that we've only configured MFA for SSH, so if you lock yourself out you can always access it via console and troubleshoot. If you do lock yourself out, just remove the "auth required pam_securid.so" line you added in step 6. 

