# B3 DNS

## 1. Preparation

> You will need four virtual machines for this exercise. Begin with assigning suitable host names in /etc/hosts.  

VM setup:

```config
192.168.10.10 lab1
192.168.10.11 lab2
192.168.10.12 lab3
192.168.10.13 lab4
```

## 2. Caching-only `nameserver`

> Setup `ns1` to function as a caching-only nameserver. It will not be authoritative for any domain, and will only resolve queries on behalf of the clients, and cache the results.
>
> Configure the nameserver to forward all queries for which it does not have a cached answer to Google's public nameserver (`8.8.8.8`). Only allow queries and recursion from local network.
>
> Start your nameserver and watch the logfile `/var/log/syslog` for any error messages. Check that you can resolve addresses through your own nameserver from the client machine. You can use `dig(1)` to do the lookups.

### 2.1 Explain the configuration you used

#### install `bind9` for `ns1`

```shell
sudo apt update
sudo apt install bind9
```

#### Edit the main configuration file, /etc/bind/named.conf.local

```shell
sudo vim /etc/bind/named.conf.local
```

#### Make the following changes

```javascript
options {
        directory "/var/cache/bind";
        forwarders {
            8.8.8.8; // Google's DNS server
            8.8.4.4; // Secondary Google's DNS server
        };
        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

#### Test the configuration

```shell
sudo named-checkconf
```

If nothing is output, the configuration file is good.

#### Restart the service

```shell
sudo systemctl restart bind9
```

#### Configure the DNS Resolver

Set the Ubuntu VM itself to use the local caching-only nameserver by editing the `resolv.conf` file.

```shell
sudo nano /etc/resolv.conf
```

add this line to the file

```config
nameserver 127.0.0.1
```

#### Use the `dig` command

```shell
dig example.com
```

Check the output for the resolved IP address and additional information. The first query might take a bit longer; subsequent queries for the same domain should be faster due to caching.

```dig
; <<>> DiG 9.18.18-0ubuntu0.22.04.2-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44381
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 793f61bcd27e09b80100000065dd91acfdbb931f14b6d120 (good)
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             300     IN      A       142.250.74.78

;; Query time: 44 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Tue Feb 27 07:39:24 UTC 2024
;; MSG SIZE  rcvd: 83
```

### 2.2 What is a recursive query? How does it differ from an iterative query?

In the context of Domain Name System (DNS) operations, the terms "recursive query" and "iterative query" refer to different methods by which a DNS resolver can obtain the IP address associated with a domain name. Understanding the distinction between these two types of queries is essential for grasping how DNS lookups function.

#### Recursive Query

A recursive query in DNS is one where the DNS client (such as a user's computer or a local DNS resolver) requests a DNS server to resolve a domain name fully. The client expects the server to handle all the necessary steps to resolve the name and return the final answer. If the server doesn't have the answer in its cache, it will query other DNS servers on behalf of the client until it finds the authoritative source for the domain name, fetching the corresponding IP address.

- **Client Expectation:** The client expects a definitive answer; it does not expect to be referred to another DNS server.
- **Server Responsibility:** The DNS server takes on the responsibility of completing the entire lookup process. If it does not have the answer, it will perform additional queries to other DNS servers until it finds the information or determines that it cannot be found.
- **Use Case:** Recursive queries are commonly used by most client-side DNS resolvers, where simplicity and completeness of the response are more critical than the control over the lookup process.

#### Iterative Query

An iterative query, on the other hand, involves the DNS client making a query to a DNS server, which then returns the best answer it can. If the server does not have the final answer, it will return a referral to another DNS server that is closer to the authoritative source for the domain. The client is then responsible for querying the referred DNS server. This process repeats, with the client making successive queries to different DNS servers, until it reaches an authoritative DNS server that can provide the final answer or until the query fails.

- **Client Responsibility:** The client takes a more active role, handling each step of the resolution process and deciding which server to query next based on the referrals received.
- **Server Role:** The DNS server provides the best information it has, which might be the final answer or a referral to another server that is closer to the authoritative source.
- **Use Case:** Iterative queries are typically used between DNS servers. They allow for more control over the lookup process and can reduce the load on each individual server since each server does not have to perform the full resolution process.

#### Key Differences

- **Responsibility:** In a recursive query, the resolver is responsible for performing the entire lookup sequence, while in an iterative query, the client (which could be a resolver itself) takes on the responsibility of following through the sequence of DNS servers.
- **Process:** Recursive queries involve a single request from the client to the resolver, expecting a complete answer. Iterative queries involve a series of requests between the client and potentially multiple DNS servers, with each server providing partial information or referrals.
- **Efficiency and Load:** Recursive queries can simplify the process for the client but put more load on the recursive server. Iterative queries distribute the load across multiple servers but require the client to manage multiple connections and handle potential complexity in the resolution path.

## 3. Create your own tld .insec

> Next, you setup your first own domain. This shows that, starting from the top level domains like .com or .fi, all layers of the DNS can also be configured by yourself, to create your very own private internet.
>
> Configure ns1 to be the primary master for .insec domain. For that you will need to create zone definitions, reverse mappings, and to let your server know it will be authoritative for the zone. Create a zone file for .insec with the following information:
>
> Primary name server for the zone is ns1.insec
Contact address should be hostmaster@insec
Use short refresh and retry time limits of 60 seconds
Put your machine's ip in ns1.insec's A record
Similarly create a reverse mapping zone c.b.a.in-addr.arpa, where a, b and c are the first three numbers of the virtual machine's current IP address (i.e. IP = a.b.c.xxx -> c.b.a.in-addr.arpa).
>
> Add a master zone entry for .insec and c.b.a.in-addr.arpa (see above) in named.conf. Reload bind's configuration files and watch the log for errors. Try to resolve ns1.insec from your client.

### 3.1 Explain your configuration

#### Create the Forward Zone File for `.insec`

Create and edit `/etc/bind/db.insec`

```bash
$TTL    3600
@       IN      SOA     ns1.insec. hostmaster.insec. (
                        2024022701 ; Serial
                        60         ; Refresh
                        60         ; Retry
                        604800     ; Expire
                        86400 )    ; Negative Cache TTL
;
@       IN      NS      ns1.insec.
ns1     IN      A       192.168.10.10
```

#### Create the Reverse Zone File

Assume ns1 ip address is 192.168.10.xxx
Create and edit `/etc/bind/db.10.168.192`

```bash
$TTL    3600
@       IN      SOA     ns1.insec. hostmaster.insec. (
                        2024022702 ; Serial
                        60         ; Refresh
                        60         ; Retry
                        604800     ; Expire
                        86400 )    ; Negative Cache TTL
;
@       IN      NS      ns1.insec.
xxx     IN      PTR     ns1.insec.
```

change the xxx accroding to 192.168.10.xxx

#### Configure the Master Zones in `named.conf`

Open and edit `/etc/bind/named.conf.local`

```shell
zone "insec" {
        type master;
        file "/etc/bind/db.insec";
};

zone "10.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.10.168.192";
};
```

#### Check and restart

```shell
sudo named-checkconf
sudo named-checkzone insec /etc/bind/db.insec
sudo named-checkzone 10.168.192.in-addr.arpa /etc/bind/db.10.168.192

sudo systemctl reload bind9
```

#### Test

```shell
dig ns1.insec @192.168.10.10
```

### 3.2 Provide the output of dig(1) for a successful query

```shell
vagrant@lab1:/etc/bind$ dig ns1.insec @192.168.10.10

; <<>> DiG 9.18.18-0ubuntu0.22.04.2-Ubuntu <<>> ns1.insec @192.168.10.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50692
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b410b27832e82b710100000065dd9acf90d64d85837825b0 (good)
;; QUESTION SECTION:
;ns1.insec.                     IN      A

;; ANSWER SECTION:
ns1.insec.              3600    IN      A       192.168.10.10

;; Query time: 0 msec
;; SERVER: 192.168.10.10#53(192.168.10.10) (UDP)
;; WHEN: Tue Feb 27 08:18:23 UTC 2024
;; MSG SIZE  rcvd: 82
```

### 3.3 How would you add an IPv6 address entry to a zone file?

## 4. Create a slave server for `.insec`

> Configure ns2 to work as a slave for .insec domain. Use a similar configuration as for the master, but don't create zone files.
>
> On the master server, add an entry (A, PTR and NS -records) for your slave server. Don't forget to increment the serial number for the zone. Also allow zone transfers to your slave.
>
> Reload configuration files in both machines and watch the logs. Verify that the zone files get transferred to the slave. Try to resolve machines in the .insec domain through both servers.

### 4.2 Explain the changes you made

#### Master server configuration

In `/etc/bind/db.insec`, add

```config
@       IN      NS      ns2.insec.
ns2     IN      A       192.168.10.11
```

In `/etc/bind/named.conf.local`, add

```config
zone "insec" {
        type master;
        file "/etc/bind/db.insec";
        allow-transfer { 192.168.10.11; };
};
```

`allow-transfer` will allow zone file transfer

In `/etc/bind/db.10.168.192`, add

```config
11      IN      PTR     ns2.insec.
```

Always remember to increment the serial number in zone files after modifications.

#### Slave server configuration

In `/etc/bind/named.conf.local`, add zone definition

```config
zone "insec" {
        type slave;
        file "slaves/db.insec";
        masters { 192.168.10.10; }; // IP of ns1
};

zone "10.168.192.in-addr.arpa" {
        type slave;
        file "slaves/db.10.168.192";
        masters { 192.168.10.10; }; // IP of ns1
};
```

Check configuration file syntax

```shell
sudo named-checkconf
sudo named-checkzone insec /etc/bind/db.insec
sudo named-checkzone c.b.a.in-addr.arpa /etc/bind/db.c.b.a
```

Restart the service

```shell
sudo systemctl reload bind9
```

### 4.1 Demonstrate the successful zone file transfer

To achieve zone file transfer, the desination should be specify first.

#### Go to slave server `/etc/bind/named.conf.options`

The path at the top of the file is the destination path

```config
options {
        directory "/var/cache/bind";
        ...
}
```

#### Go to slave server `/etc/bind/named.conf.local`

See the zone definition file location, the path is relative to the path in `/etc/bind/named.conf.options`

```config
zone "insec" {
        type slave;
        file "slaves/db.insec";
        masters { 192.168.10.10; }; // IP of ns1
};
```

As a result, the zone file would be transferred to `/var/cache/bind/slaves/`

In case there is no `/var/cache/bind/slaves/` folder, this file should be created.

```shell
sudo mkdir /var/cache/bind/slave
sudo chown bind:bind /var/cache/bind/slave
sudo chmod 775 /var/cache/bind/slave
```

#### Verification

Go to `/var/cache/bind/slaves/` to see if there are two zone files in this case.

### Provide the output of dig(1) for a successful query from the slave server. Are there any differences to the queries from the master?

```shell
vagrant@lab2:/var/cache/bind/slaves$ dig ns2.insec @192.168.10.11

; <<>> DiG 9.18.18-0ubuntu0.22.04.2-Ubuntu <<>> ns2.insec @192.168.10.11
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11744
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 5ea136cb3e6ddf580100000065de15eb552f437f7d621c4b (good)
;; QUESTION SECTION:
;ns2.insec.                     IN      A

;; ANSWER SECTION:
ns2.insec.              86400   IN      A       192.168.10.11

;; Query time: 0 msec
;; SERVER: 192.168.10.11#53(192.168.10.11) (UDP)
;; WHEN: Tue Feb 27 17:03:39 UTC 2024
;; MSG SIZE  rcvd: 82
```

## 5. Create a subdomain `.not.insec.`

### 5.1 Explain the changes you made

#### In ns1 `/etc/bind/db.insec`, Delegate `.not.insec` Subdomain on `ns1`

Add

```config
not     IN      NS      ns2.insec.
not     IN      NS      ns3.insec.
ns2     IN      A       192.168.10.11
ns3     IN      A       192.168.10.12
```

Increment serial number and reload, ensure the log doesn't show any errors.

#### Configure `ns2` as Master for `.not.insec`

create `/etc/bind/db.not.insec` with

```config
$TTL 86400
@     IN  SOA ns2.insec. hostmaster.insec. (
            1          ; Serial
            86400      ; Refresh
            7200       ; Retry
            3600000    ; Expire
            86400 )    ; Minimum
@     IN  NS  ns2.insec.
@     IN  NS  ns3.insec.
ns2   IN  A   192.168.10.11
ns3   IN  A   192.168.10.12
```

update `/etc/bind/named.conf.local`

```config
zone "not.insec" {
    type master;
    file "/etc/bind/db.not.insec";
};
```

reload and verify

#### Configure `ns3` as Slave for `.not.insec`

In `/etc/bind/named.conf.local`

```config
zone "not.insec" {
    type slave;
    file "slaves/db.not.insec";
    masters { 192.168.10.11; }; // IP of ns2
};
```

Create `/var/cache/bind/slave/` directory and grant permission, then restart bind9.

### 5.2 Provide the output of dig(1) for successful queries from all the three name servers

```shell
dig @192.168.10.10 ns1.not.insec 1n 2n 3n
dig @192.168.10.11 ns1.not.insec 1n 2n 3n
dig @192.168.10.12 ns1.not.insec 1n 2n 3n

dig @192.168.10.10 ns2.not.insec 1n 2n 3n
dig @192.168.10.11 ns2.not.insec 1y 2y 3y
dig @192.168.10.12 ns2.not.insec 1y 2y 3y

dig @192.168.10.10 ns3.not.insec 1n 2n 3n
dig @192.168.10.11 ns3.not.insec 1y 2y 3y
dig @192.168.10.12 ns3.not.insec 1y 2y 3y
```

## 6. Implement transaction signatures

### 6.1 Explain the changes you made. Show the successful and the unsuccessful zone transfer in the log

#### Generate key on whichever server(no difference)

command

```shell
tsig-keygen -a HMAC-SHA256 keyname
```

output

```shell
key "keyname" {
        algorithm hmac-sha256;
        secret "jTC6zqv+16ysRSR+fjKlFKlqHRB1Wh8f6D3EA+61cPE=";
};
```

#### In `ns2` (master for `not.insec`)

create a file `/etc/bind/tsig.key` and paste the key content.

In `/etc/bind/named.conf.local`, add line

```shell
include "/etc/bind/tsig.key";
```

add

```shell
zone "not.insec" {
    type master;
    file "/etc/bind/db.not.insec";
    allow-transfer { key "keyname"; };
    also-notify { 192.168.10.12 key "keyname"; };
};
```

#### In `ns3` file `/etc/bind/named.conf.local`

```shell
zone "not.insec" {
    type slave;
    file "slaves/db.not.insec";
    masters { 192.168.10.11 key "keyname"; }; // IP of ns2
};
```

#### Reload and testing

Use this to send a unauthenticated transfer

```shell
dig @192.168.10.11 axfr not.insec
```
