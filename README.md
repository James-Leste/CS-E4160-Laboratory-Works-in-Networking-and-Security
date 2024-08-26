# CS-E4160-Laboratory-Works-in-Networking-and-Security Catalog

- A1 & B1: Setting up and Networking tools

## Networking Part

### A2_Email_Server

> In this exercise you will learn how to setup an email server with filtering rules and spam detection. Consider that from now on you'll have to do extensive self-research to be able to successfully complete the assignments.  

[Link](https://github.com/James-Leste/CS-E4160-Laboratory-Works-in-Networking-and-Security/blob/main/A2_Email_Server.md)

### A3_IPv6

> In this exercise you will familiarize yourself with Internet Protocol version 6 (IPv6). The main task is to build a small network and assign addresses and routes automatically with router advertisements. You will also create a connection between two IPv6 networks over an IPv4 network.

[Link](https://github.com/James-Leste/CS-E4160-Laboratory-Works-in-Networking-and-Security/blob/main/A3_IPv6.md)

### A4_Encrypted_Filesystems

> In this exercise you will simulate encryption of an external memory (such as USB memory stick) using a file as the storage media. Simulation is used primarily because in many cases you have no physical access to the server machines (in addition, the servers are virtual). Two different schemes will be used: encrypted loopback device with dm_crypt and encryption layer for an existing filesystem with gocryptfs. However, we will begin by familiarizing with GPG and encrypting single files.

[Link](https://github.com/James-Leste/CS-E4160-Laboratory-Works-in-Networking-and-Security/blob/main/A4_Encrypted_Filesystems.md)

### A5_Firewall

> This assignment introduces you to some firewalling basics. It includes packet filtering using Linux Nftables. You will first setup a router which will work as a firewall between the other two machines. The firewall will then be extended with a web proxy.

[Link](https://github.com/James-Leste/CS-E4160-Laboratory-Works-in-Networking-and-Security/blob/main/A5_Firewall.md)

## Security Part

### B2_Web_Server

> In this exercise, you will introduce yourself to some basic features of Apache web server and its plugins. In addition to that you will set up a Node.js server for serving a webpage, and configure an nginx server as a reverse proxy for the servers. Take into account that from now on you'll have to do extensive self-research to be able to successfully complete the assignments. You will need three virtual machines to complete this assignment. The final web server network configuration will look like the image below(see link).

[Link](https://github.com/James-Leste/CS-E4160-Laboratory-Works-in-Networking-and-Security/blob/main/B2_Web_Server.md)

### B3_DNS

> In this exercise you will set up a simple caching-only nameserver, implement your own .insec -domain, complete with a slave server - and finally a subdomain .not.insec, enhanced with DNSSEC. You will also try out Pi-hole - a DNS sinkhole, which can be used to stop DNS-queries for blacklisted domains.

[Link](https://github.com/James-Leste/CS-E4160-Laboratory-Works-in-Networking-and-Security/blob/main/B3_DNS.md)

### B4_Network_Filesystems

> Network filesystems create a way to access files on another computer as if they are located on your computer. A basic approach to accessing remote files would be to download them, edit them and then upload the edited versions to the server. Mounting the files as a directory on your computer makes it easier to manage and use the files and synchronize changes between your computer and the remote server. Data integrity loss due to device failure can be very problematic. To prevent such data loss, redundancy and integrity mechanisms can be integrated into file systems.

[Link](https://github.com/James-Leste/CS-E4160-Laboratory-Works-in-Networking-and-Security/blob/main/B4_Network_Filesystems.md)

### B5_VPN

> This assignment introduces you to the Virtual Private Network (VPN) concept. You will use OpenVPN and all three VMs to establish a VPN in practice by creating and examining a host-to-net VPN scenario. A roadwarrior host (lab3, RW) establishes a secure tunnel to a gateway (lab1, GW). Traffic can flow from the roadwarrior through the gateway to a Storage server (lab2, SS)  and back. Hosts on the right-side local link can not eavesdrop or modify the traffic flowing inside the tunnel.

[Link](https://github.com/James-Leste/CS-E4160-Laboratory-Works-in-Networking-and-Security/blob/main/B5_VPN.md)

## Learning Outcome

From this course, I learnt to proficiently set up and configure various networking and security tools, demonstrating practical knowledge and hands-on experience in managing key network services. I have:

- Successfully configured an email server with advanced filtering and spam detection mechanisms.
- Understood and implemented IPv6 networking, including automatic address assignment and interconnecting IPv6 networks over IPv4.
- Simulated and deploy encrypted filesystems, utilizing encryption technologies like dm_crypt and gocryptfs to secure external memory.
- Designed and configured a firewall using Linux Nftables, enhancing network security by implementing packet filtering and proxy services.
- Set up web servers using Apache, Node.js, and nginx as a reverse proxy, while managing virtual machines to establish complex web server networks.
- Implemented and managed DNS services, including caching-only nameservers, DNSSEC-enhanced subdomains, and DNS sinkholes to block unwanted domains.
- Established network filesystems, enabling remote access to files with built-in redundancy and data integrity mechanisms.
- Deployed and configured Virtual Private Networks (VPNs) using OpenVPN to secure network communication between different hosts within a simulated network.
