# A2 Email Server

2 Ubuntu VMs:
tlab1 192.168.10.10
tlab2 192.168.10.11

## 1. Preparation

> During this assignment you will need two hosts (lab1 and lab2). Configure them in the same network, such that they can communicate to each other for mail delivery.


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
> - myhostname (from /etc/hosts)
>
> - mydestination
>
> - mynetworks (localhost and virtual machines IP block)
>
> Disable ETRN and VRFY commands. Remember to reload postfix service /etc/init.d/postfix every time you edit main.cf.

- 2.0.1 Installing postfix, procmail, spamassassin
    **Solution**
    ```bash
    sudo apt-get update
    sudo apt-get install postfix -y
    sudo apt-get install procmial -y
    sudo apt-get install spamassassin spamc -y
    ```
    **Note:** Upon installing postfix, the installation wizard
- 2.1 Configure the postfix configuration file main.cf to fill the requirements above. 1p  
  
    **Solution**


