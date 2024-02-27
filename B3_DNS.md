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