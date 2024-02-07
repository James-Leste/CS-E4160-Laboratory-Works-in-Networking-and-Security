# A2 Email Server

2 Ubuntu VMs:
```/etc/hosts
tlab1 192.168.10.10
tlab2 192.168.10.11
```


## 1. Preparation

> During this assignment you will need two hosts (`lab1` and `lab2`). Configure them in the same network, such that they can communicate to each other for mail delivery.


- 1.1 Add the IPv4 addresses and aliases of lab1 and lab2 on both computers to the **/etc/hosts** file.

    **Solution**
    Edit /etc/hosts fot tlab1 & tlab2   
    syntax: `[ip address] [alias]`
    ```/etc/hosts
    192.168.10.10 tlab1
    192.168.10.11 tlab2
    ```
    Use `ping tlab1` and `ping tlab2` to test if the aliases can be resolve to correct ip addresses.

## 2. Installing software and Configuring postfix and exim4

> As a first step, install all the software that will be used in the assignment. Verify that the following packages are installed:
> 
> **lab1:** postfix, procmail, spamassassin  
> **lab2:** exim4
>
> **Postfix** is the MTA used on lab1 for delivering the mail. **Exim4** is the MTA used on lab2. **Procmail** is used as the MDA on lab1. **Spamassassin**, as the name suggests, is a tool used for spam detection.
>
> Installing **mailutils** on lab1 can help with handling incoming mail. Then, you should configure postfix to deliver mail from lab2 to lab1.
> 
> **Edit main configuration file for postfix (main.cf, postconf(5)) on lab1.** You must change, at least, the following fields:
>
> - myhostname (from `/etc/hosts`)
>
> - mydestination
>
> - mynetworks (localhost and virtual machines IP block)
>
> Disable ETRN and VRFY commands. Remember to reload postfix service `/etc/init.d/postfix` every time you edit main.cf.

- **2.0.1 Installing postfix, procmail, spamassassin on tlab1**
  
    **Solution**
    ```shell
    sudo apt-get update
    sudo apt-get install postfix -y
    sudo apt-get install procmial -y
    sudo apt-get install spamassassin spamc -y
    ```
    **Note:** Upon installing postfix, the installation wizard will prompt some configuration options.  
    1. General mail configuration type: Internet with smarthost
    2. System mail: `tlab1`; (This will form the email address: `username@tlab1`)
    3. SMTP relay host: default
   
    If you want to change some of these configurations later, the following command can be used:
    ```shell
    sudo dpkg-reconfigure postfix
    ```
- **2.0.2 Installing exim4 on tlab2**
  
    **Solution**
    ```shell
    sudo apt-get update
    sudo apt-get install exim4 -y
    ```
    **Note:** Differently, during installation exim4 won't prompt any windows for configuration. exim4 can be configured later using:
    ```shell
    sudo dpkg-reconfigure exim4-config
    ```

- **2.1 Configure the postfix configuration file `main.cf` to fill the requirements above. 1p** 
  
    **Solution**  
    edit **/etc/postfix/main.cf**, change following configurations
    ```main.cf
    myhostname: tlab1
    mydestination: $myhostname, tlab1, localhost.localdomain, localhost
    mynetworks: 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 192.168.10.0/24

    smtpd_discard_ehlo_keywords = etrn
    disable_vrfy_command = yes
    ```
- **2.2 What is purpose of the `main.cf` setting "mydestination"? 1p**  
    **Solution**
    ```
    Hi
    ```

- **2.3 Why is it a really bad idea to set mynetworks broader than necessary (e.g. to 0.0.0.0/0)? 1p**
    **Solution**  
    ```

    ```

- **2.4 What is the idea behind the ETRN and VRFY verbs? How can a malicious party misuse the commands? 2p**

- **2.5 Configure exim4 on lab2 to handle local emails and send all the rest to lab1. After you have configured postfix and exim4 you should be able to send mail from lab2 to lab1, but not vice versa. Use the standard debian package reconfiguration tool dpkg-reconfigure(8) to configure exim4.**
    ```
    ```

## 3. Sending email
> Send a message from `lab2` to `username@lab1` using `mail(1)`. Replace the `username` with your username. Read the message on lab1. See also email message headers. See incoming message information from `/var/log/mail.log` using `tail(1)`.

- **3.0 send mails using `mail`**
    ```shell
    echo 'mail content' | mail -s "Subject" vagrant@tlab1
    ```

- **3.1 Explain shortly the incoming mail log messages. 2p**

- **3.2 Explain shortly the email headers. At what point is each header added? 2p**

## 4. Configuring procmail and spamassassin
> Next, you will configure procmail to deliver any incoming mail for spam filtering to spamassassin, and then filter to folders with self-configured rules.
>
> Procmail is configured by writing instruction sets caller recipes to a configuration file `procmailrc(5)`. Edit (create if necessary) `/etc/procmailrc` and begin by piping your arriving emails into spamassassin with the following recipe:
>
> ```config
> :0fw
> | /usr/bin/spamassassin
> ```
> In postfix main.cf, you have to enable procmail with mailbox_command line:
> ```
> /usr/bin/procmail -a "$USER"
> ```
> Remember to reload postfix configuration after editing it.
>
> You may need to start the spamassassin daemon after enabling it in the configuration file `/etc/default/spamassassin`.
>
> Send an email message from `lab2` to `username@lab1`. Read the message on lab1. See email headers. If you do not see spamassassin headers there is something wrong, go back to previous step and see `/var/log/mail.log`.
> Write additional procmail recipes to:
>
> - Automatically filter spam messages to a different folder.
>
> - Add a filter for your user to automatically save a copy of a message with header `[cs-e4160]` in the subject field to a different folder.
>
> - Forward a copy of the message with the same `[cs-e4160]` header to `testuser1@lab1` (create user if necessary).
>
>  Hint: You can use file .procmailrc in user's home directory for user-specific rules.
- **4.0**
modify `/etc/procmailrc`
```config
:0fw
| /usr/bin/spamc

:0:
* ^X-Spam-Status: Yes
.spam/
```
create and modify `~/.procmailrc`
```config
# Save messages with [cs-e4160] in the subject to a folder
:0 c:
* ^Subject:.*\[cs-e4160\]
.cs-e4160/

# Forward messages with [cs-e4160] in the subject
:0 c
* ^Subject:.*\[cs-e4160\]
! testuser1@tlab1
```






