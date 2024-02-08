# A2 Email Server

2 Ubuntu VMs:

```/etc/hosts
tlab1 192.168.10.10
tlab2 192.168.10.11
```

## 1. Preparation

> During this assignment you will need two hosts (`lab1` and `lab2`). Configure them in the same network, such that they can communicate to each other for mail delivery.

- **1.1 Add the IPv4 addresses and aliases of lab1 and lab2 on both computers to the /etc/hosts file.**  

    **Solution**
    Edit `/etc/hosts` fot `tlab1` & `tlab2`  
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

    **- mydestination**
    The mydestination parameter specifies what domains this machine will deliver locally, instead of forwarding to another machine.

    **- mydomain (default: see "postconf -d" output)**
    The internet domain name of this mail system. The default is to use `$myhostname` minus the first component, or "localdomain" (Postfix 2.3 and later). `$mydomain` is used as a default value for many other configuration parameters.
    Example:

    ```shell
    mydomain = domain.tld
    ```

    **- myhostname (default: see "postconf -d" output)**
    The internet hostname of this mail system. The default is to use the fully-qualified domain name (FQDN) from gethostname(), or to use the non-FQDN result from gethostname() and append ".$mydomain". $myhostname is used as a default value for many other configuration parameters.
    Example:

    ```shell
    myhostname = host.example.com
    ```

- **2.3 Why is it a really bad idea to set mynetworks broader than necessary (e.g. to 0.0.0.0/0)? 1p**
**Solution**  
**mynetworks (default: see "postconf -d" output)**
The list of "trusted" remote SMTP clients that have more privileges than "strangers".
In particular, "trusted" SMTP clients are allowed to relay mail through Postfix. See the smtpd_relay_restrictions parameter description in the postconf(5) manual.  
Configuring the mynetworks mean turning the mail server into an open relay.
Anyone on the Internet can use the server to send email without authentication. Resoursce abuse.

- **2.4 What is the idea behind the ETRN and VRFY verbs? How can a malicious party misuse the commands? 2p**
**Solution**
The **`ETRN`** command is used to request a remote mail server to start the process of sending queued mail messages to the server that issued the command.
It Allows systems that are not always connected to the internet to request mail from a queue when they connect.
    **- Spamming:** malicious party could potentially use the ETRN command to flood a server with queued messages, leading to a denial of service (DoS) or overwhelming the server with spam.
    **- Unsolicited Queue Flushing:** If not properly authenticated or restricted, malicious parties could trigger the delivery of queued messages at inconvenient times, disrupting the intended flow of email delivery and possibly leading to resource exhaustion.
    The **`VRFY`** command is used to verify if a particular user name or mailbox exists on the receiving server.
    **Spamming Preparation:** Spammers can use the VRFY command to compile lists of valid email addresses, which they can later target with unsolicited emails.
    **Information Gathering:** Malicious parties can use it to gather information about valid users on a system, which could be used for targeted attacks or social engineering.

- **2.5 Configure exim4 on lab2 to handle local emails and send all the rest to lab1. After you have configured postfix and exim4 you should be able to send mail from lab2 to lab1, but not vice versa. Use the standard debian package reconfiguration tool dpkg-reconfigure(8) to configure exim4.**
use debian config tool to configure exim4 on `tlab2`

    ```shell
    sudo dpkg-reconfigure exim4-config
    ```

  - General type: smarthost
  - Mail name: `tlab2` (or preferred name)
  - IP-addresses to listen on for incoming SMTP connections: `192.168.10.11`
  - Other destinations for which mail is accepted: `tlab2`
  - Machines to relay mail for: blank
  Keep number of DNS-queries minimal (Dial-on-Demand)?: No

Edit `/etc/exim4/update-exim4.conf.conf`

```shell
dc_eximconfig_configtype='internet'
dc_other_hostnames='tlab2'
dc_local_interfaces='127.0.0.1 ; ::1 ; 192.168.10.11'
dc_readhost=''
dc_relay_domains='tlab1'
dc_minimaldns='false'
dc_relay_nets='192.168.10.0/24'
dc_smarthost='192.168.10.10'
CFILEMODE='644'
dc_use_split_config='false'
dc_hide_mailname=''
dc_mailname_in_oh='true'
dc_localdelivery='mail_spool'
```

## 3. Sending email

> Send a message from `lab2` to `username@lab1` using `mail(1)`. Replace the `username` with your username. Read the message on lab1. See also email message headers. See incoming message information from `/var/log/mail.log` using `tail(1)`.

- **3.0 send mails using `mail`**

    ```shell
    echo 'mail content' | mail -s "Subject" vagrant@tlab1
    ```

- **3.1 Explain shortly the incoming mail log messages. 2p**
/var/log/mail.log

    ```log
    Feb  8 16:38:22 tlab1 postfix/smtpd[28408]: connect from tlab2[192.168.10.11]
    <!-- Denial of ETRN -->
    Feb  8 16:38:22 tlab1 postfix/smtpd[28408]: discarding EHLO keywords: ETRN
    <!-- bad certificate -->
    Feb  8 16:38:22 tlab1 postfix/smtpd[28408]: warning: TLS library problem: error:0A000412:SSL routines::sslv3 alert bad certificate:../ssl/record/rec_layer_s3.c:1584:SSL alert number 42:
    <!-- lose connection after try to initiate a TLS-secured session -->
    Feb  8 16:38:22 tlab1 postfix/smtpd[28408]: lost connection after STARTTLS from tlab2[192.168.10.11]
    <!-- disconnetion -->
    Feb  8 16:38:22 tlab1 postfix/smtpd[28408]: disconnect from tlab2[192.168.10.11] ehlo=1 starttls=1 commands=2
    <!-- reconnect -->
    Feb  8 16:38:22 tlab1 postfix/smtpd[28408]: connect from tlab2[192.168.10.11]
    <!-- discard ETRN -->
    Feb  8 16:38:22 tlab1 postfix/smtpd[28408]: discarding EHLO keywords: ETRN
    <!-- recognize client -->
    Feb  8 16:38:22 tlab1 postfix/smtpd[28408]: BB9A13E853: client=tlab2[192.168.10.11]
    <!-- clean up message, give a message-id header -->
    Feb  8 16:38:22 tlab1 postfix/cleanup[28411]: BB9A13E853: message-id=<E1rY7Pq-0006vU-M0@tlab2>
    <!-- queue manager accept the file -->
    Feb  8 16:38:22 tlab1 postfix/qmgr[28356]: BB9A13E853: from=<vagrant@tlab2>, size=579, nrcpt=1 (queue active)
    <!-- disconnect -->
    Feb  8 16:38:22 tlab1 postfix/smtpd[28408]: disconnect from tlab2[192.168.10.11] ehlo=1 mail=1 rcpt=1 bdat=1 quit=1 commands=5
    <!-- sent using procmail -->
    Feb  8 16:38:22 tlab1 postfix/local[28412]: BB9A13E853: to=<vagrant@tlab1>, relay=local, delay=0.03, delays=0.01/0.01/0/0.01, dsn=2.0.0, status=sent (delivered to command: /usr/bin/procmail -a "$USER")
    Feb  8 16:38:22 tlab1 postfix/qmgr[28356]: BB9A13E853: removed
    ```

- **3.2 Explain shortly the email headers. At what point is each header added? 2p**

    ```config
    <!-- If the path can't be reached -->
    Return-Path: <vagrant@tlab2>
    <!-- original recipient with out forwarding -->
    X-Original-To: vagrant@tlab1
    <!-- final recipient -->
    Delivered-To: vagrant@tlab1
    <!-- postfix side info -->
    Received: from tlab2 (tlab2 [192.168.10.11])
            by tlab1 (Postfix) with ESMTP id BB9A13E853
            for <vagrant@tlab1>; Thu,  8 Feb 2024 16:38:22 +0000 (UTC)
    <!-- exim4 side info -->
    Received: from vagrant by tlab2 with local (Exim 4.95)
            (envelope-from <vagrant@tlab2>)
            id 1rY7Pq-0006vU-M0
            for vagrant@tlab1;
            Thu, 08 Feb 2024 16:38:22 +0000
    <!-- primary recipient -->
    To: vagrant@tlab1
    Subject: Another test normal message
    <!-- Multipurpose Internet Mail Extensions, allow text, images, etc. -->
    MIME-Version: 1.0
    Content-Type: text/plain; charset="UTF-8"
    Content-Transfer-Encoding: 8bit
    <!-- generated by the sender server -->
    Message-Id: <E1rY7Pq-0006vU-M0@tlab2>
    <!-- sender -->
    From: vagrant@tlab2
    Date: Thu, 08 Feb 2024 16:38:22 +0000
    X-UID: 2
    Status: O
    ```

## 4. Configuring procmail and spamassassin

> Next, you will configure procmail to deliver any incoming mail for spam filtering to spamassassin, and then filter to folders with self-configured rules.
>
> Procmail is configured by writing instruction sets caller recipes to a configuration file `procmailrc(5)`. Edit (create if necessary) `/etc/procmailrc` and begin by piping your arriving emails into spamassassin with the following recipe:
>
> ```config
> :0fw
> | /usr/bin/spamassassin
> ```
>
> In postfix main.cf, you have to enable procmail with mailbox_command line:
>
> ```shell
> /usr/bin/procmail -a "$USER"
> ```
>
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
> Hint: You can use file .procmailrc in user's home directory for user-specific rules.

- **4.0 procmail configuration with spamassassin**

    modify `/etc/procmailrc`

    ```conf
    # 0 means locking
    # f filter
    # w Procmail should wait for the filter program to complete and use its exit status to determine if the recipe was successful.
    :0fw
    | /usr/bin/spamc

    # :0: the second : means a lock to the destination mailbox ensure safety transfer
    :0:
    * ^X-Spam-Status: Yes
    .spam/
    ```

    create and modify `~/.procmailrc`

    ```config
    <!-- c means using a carbon copy of the mail -->
    # Save messages with [cs-e4160] in the subject to a folder
    :0 c:
    * ^Subject:.*\[cs-e4160\]
    .cs-e4160/

    # Forward messages with [cs-e4160] in the subject
    :0 c
    * ^Subject:.*\[cs-e4160\]
    ! testuser1@tlab1
    ```

- **4.1 How can you automatically filter spam messages to a different folder using procmail? Demonstrate by sending a message that gets flagged as spam.**
**Solution**
    from vagrant@tlab2 send email using `mail` command

    ```shell
    echo "XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X TESTTEST" | mail -s "TEST SPAM
    " vagrant@tlab1
    ```

- **4.2 Demonstrate the filter rules created for messages with `[cs-e4160]` in the subject field by sending a message from lab2 to `<user>@lab1` using the header.**
**Solution**
    from vagrant@tlab2 send email using `mail` command

    ```shell
    echo "test content" | mail -s "[cs-e4160]
    " vagrant@tlab1
    ```

    The mail would be found in `~/.cs-e4160/new`

- **4.3 Explain briefly the additional email headers (compared to step 3.2).**
    extra header

    ```config
    X-Spam-Checker-Version: SpamAssassin 3.4.6 (2021-04-09) on tlab1
    <!-- low spam level -->
    X-Spam-Level:
    <!-- ALL_TRUSTED: pass through trust server only -->
    <!-- TO_MALFORMED:format of the 'To' field -->
    X-Spam-Status: No, score=-0.9 required=5.0 tests=ALL_TRUSTED,TO_MALFORMED
            autolearn=no autolearn_force=no version=3.4.6
    ```

    /var/log/mail.log

  ```log
    <!-- post fix pickup from vagrant, assign internal id -->
    Feb  8 16:20:06 tlab1 postfix/pickup[28355]: E98073E854: uid=1000 from=<vagrant>
    
    <!-- post clean up, assign message-id -->
    Feb  8 16:20:06 tlab1 postfix/cleanup[28370]: E98073E854: message-id=<20240208162006.E98073E854@tlab1>

    <!-- queue manager take message(450 bytes) to delivery -->
    Feb  8 16:20:06 tlab1 postfix/qmgr[28356]: E98073E854: from=<vagrant@tlab1>, size=450, nrcpt=1 (queue active)

    <!-- spamassassin connection -->
    Feb  8 16:20:06 tlab1 spamd[20330]: spamd: connection from ::1 [::1]:52454 to port 783, fd 5

    <!-- spamassassin change user id to root -->
    Feb  8 16:20:06 tlab1 spamd[20330]: spamd: setuid to root succeeded

    <!-- fallback to nobody -->
    Feb  8 16:20:06 tlab1 spamd[20330]: spamd: still running as root: user not specified with -u, not found, or set to root, falling back to nobody

    <!-- message processed by spamassassin -->
    Feb  8 16:20:06 tlab1 spamd[20330]: spamd: processing message <20240208162006.E98073E854@tlab1> for root:65534

    <!-- spamassassin give result (0.1/0.5) -->
    Feb  8 16:20:07 tlab1 spamd[20330]: spamd: clean message (0.1/5.0) for root:65534 in 0.0 seconds, 557 bytes.

    <!-- spamassassin detailed info -->
    Feb  8 16:20:07 tlab1 spamd[20330]: spamd: result: . 0 - NO_RELAYS,TO_MALFORMED scantime=0.0,size=557,user=root,uid=65534,required_score=5.0,rhost=::1,raddr=::1,rport=52454,mid=<20240208162006.E98073E854@tlab1>,autolearn=no autolearn_force=no

    <!--  postfix local delivery agent has successfully delivered the mail to root@tlab1, using local relay -->
    Feb  8 16:20:07 tlab1 postfix/local[28371]: E98073E854: to=<root@tlab1>, orig_to=<root>, relay=local, delay=0.05, delays=0.01/0/0/0.05, dsn=2.0.0, status=sent (delivered to command: /usr/bin/procmail -a "$USER")

    <!-- remove from queue -->
    Feb  8 16:20:07 tlab1 postfix/qmgr[28356]: E98073E854: removed
    ```

## 5. E-mail servers and DNS

> Configuring a DNS server is the task for the B-path next week. Information about mail servers is also stored in the DNS system. To get a surface level understanding, study the topic from the internet.

**5.1 What information is stored in MX records in the DNS system? 1p**

**5.2 Explain briefly two ways to make redundant email servers using multiple email servers and the DNS system. Name at least two reasons why you would have multiple email servers for a single domain? 2p**