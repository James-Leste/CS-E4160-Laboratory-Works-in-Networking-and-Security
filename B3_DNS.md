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

### 2.1 Explain the configuration you used.

#### install `bind9` for `ns1`

```shell
sudo apt update
sudo apt install bind9
```

#### Edit the main configuration file, /etc/bind/named.conf:

```shell
sudo vim /etc/bind/named.conf
```

#### Make the following changes

- options section:

    Uncomment the listen-on option and add the IP address of your virtual machine.
    Uncomment the allow-query option and specify the network range of your clients (e.g., 192.168.1.0/24).
- zone section:

    Comment out the example zone (zone "localhost" { ... }).
    You don't need a zone definition for a caching-only server.

#### Configure Firewall

Open port 53

#### Restart and Verify:

```shell
sudo systemctl restart named
```

- Test the caching functionality by pinging a website from another machine on the network. The first ping might take longer as the server retrieves the information. Subsequent pings using the same domain should be faster as the result is cached.

### 2.2 What is a recursive query? How does it differ from an iterative query?