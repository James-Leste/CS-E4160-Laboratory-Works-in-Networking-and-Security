# A3 IPv6

## 1. IPv6 addressing

> IPv6 addresses may look more foreign to many people, due to being more complex than IPv4 addresses, even though they have been in use for a long time. You can think in your case, which is the more familiar loopback address: 127.0.0.1 or ::1 (0:0:0:0:0:0:0:1 or even more verbose 0000:0000:0000:0000:0000:0000:0000:0001). The following picture shows the categories of different IPv6 address types, so first study it and familiarize yourself with the use of them. Then answer some questions related to some of them.

### 1.1 In Unique Local IPv6 Unicast Address space. how does a device know whether the IPv6 address it just created for itself is unique?

In the Unique Local IPv6 Unicast Address space, a device can ensure the uniqueness of an address it has generated using the Duplicate Address Detection (DAD) mechanism. This is part of the Neighbor Discovery Protocol (NDP) for IPv6, defined in RFC 4862. When a device generates an address, it performs DAD by sending a Neighbor Solicitation message to check if any other device is using the same address. If there's no response, the address is considered unique and can be assigned to the interface.


### 1.2 Explain 3 methods of dynamically allocating IPv6 global unicast addresses?

## 2. Build two IPv6 networks with a router

> To prepare for creating the final network, you will first familiarize yourself with routing messages between two IPv6 networks.
>
> You will set up lab1 to act as a router. This means that lab1 will route traffic from one network to another. In practice this is done using routing tables, but before that you must allow certain things that are not allowed by default. Use the following sysctl commands (note that the last one will avoid messing up enp0s3 interface. You should do the last one on all of your VMs to prevent problems with misconfiguration.):
>
> ```shell
> sudo sysctl -w net.ipv6.conf.default.forwarding=1
> sudo sysctl -w net.ipv6.conf.all.forwarding=1
> sudo sysctl -w net.ipv6.conf.enp0s3.accept_ra=0
> ```
>
> Assign static IPv6 addresses from the subnets fd01:2345:6789:abc1::/64 and fd01:2345:6789:abc2::/64 to your virtual machines. On lab2 and lab3 add IPv6 route to the other network using lab1 as a gateway. Make sure that you can ping lab1 from lab2 and lab3, then ensure that IPv6 routing works on lab1 by pinging lab3 from lab2. You can also try traceroute to see the route taken by the packets.
>
> You can do the configurations using ip(8). Editing /etc/network/interfaces is a bad idea as it can mess radvd in the next part. The addresses should be assigned to intnet interfaces, not the NAT Network.

### 2.1 What do the above sysctl commands do?

- `sudo sysctl -w net.ipv6.conf.default.forwarding=1`: This command enables IPv6 forwarding for all interfaces that will be created in the future (the default interfaces). This is necessary for lab1 to forward IPv6 packets, acting as a router.

- `sudo sysctl -w net.ipv6.conf.all.forwarding=1`: This command enables IPv6 forwarding for all existing interfaces. It's similar to the first command but applies to all currently active interfaces.

- `sudo sysctl -w net.ipv6.conf.enp0s3.accept_ra=0`: This command disables the acceptance of IPv6 Router Advertisements on the enp0s3 interface. Router Advertisements are usually used for automatic configuration of addresses and other configuration settings. Disabling this on a router is often recommended because the router itself should not be auto-configured by other routers; it should be statically configured or use a different method for configuration.

### 2.2 The subnets used belong to Unique Local IPv6 Unicast Address space. Explain what this means and what is the format of such addresses

Unique Local Addresses (ULAs) are IPv6 addresses intended for local communications within a site or between a limited set of sites. They are similar to IPv4's private address ranges (like 10.x.x.x, 172.16.x.x-172.31.x.x, 192.168.x.x). The format for ULAs is fd00::/8, where the first 8 bits are always 1111 1101 or fd in hexadecimal. The next 40 bits are randomly generated to ensure uniqueness, followed by a subnet ID and an interface identifier.

### 2.3 List all commands that you used to add static addresses to lab1, lab2 and lab3. Explain one of the add address commands

```shell
lab1
fd01:2345:6789:abc1::1/64
fd01:2345:6789:abc2::1/64

lab2
fd01:2345:6789:abc1::2/64
lab3
fd01:2345:6789:abc2::2/64
```

lab1, assign IPv6 addresses from each subnet

```shell
sudo ip -6 addr add fd01:2345:6789:abc1::1/64 dev enp0s8
sudo ip -6 addr add fd01:2345:6789:abc2::1/64 dev enp0s8
```

lab2, assign an IPv6 address from the first subnet and set lab1 as the gateway for the second subnet.

```shell
sudo ip -6 addr add fd01:2345:6789:abc1::2/64 dev enp0s8
sudo ip -6 route add fd01:2345:6789:abc2::/64 via fd01:2345:6789:abc1::1
```

lab3, assign an IPv6 address from the second subnet and set lab1 as the gateway for the first subnet.

```shell
sudo ip -6 addr add fd01:2345:6789:abc2::2/64 dev enp0s8
sudo ip -6 route add fd01:2345:6789:abc1::/64 via fd01:2345:6789:abc2::1
```

### 2.4 Show the command that you used to add the route to lab3 on lab2, and explain it

```shell
sudo ip -6 route add fd01:2345:6789:abc2::/64 via fd01:2345:6789:abc1::1
```

### 2.5 Show enp0s8 interface information from lab2, as well as the IPv6 routing table. Explain the IPv6 information from the interface and the routing table. What does a double colon (::) indicate?

routing table

```shell
::1 dev lo proto kernel metric 256 pref medium
fd01:2345:6789:abc1::/64 dev enp0s8 proto kernel metric 256 pref medium
fd01:2345:6789:abc2::/64 via fd01:2345:6789:abc1::1 dev enp0s8 metric 1024 pref medium
fe80::/64 dev enp0s3 proto kernel metric 256 pref medium
fe80::/64 dev enp0s8 proto kernel metric 256 pref medium
```

- `pref medium` is the preference of the route which may be used to choose between routes of equal metric.
- `metric 256` is the routing metric for the route; lower values have a higher priority when multiple routes to the same destination exist.

interface information

```shell
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet6 fd01:2345:6789:abc1::2/64 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe4a:47a0/64 scope link
       valid_lft forever preferred_lft forever
```

- BROADCAST indicates that the interface accepts broadcast packets.
- MULTICAST indicates that the interface supports multicast.
- UP means that the network interface is enabled.
- LOWER_UP indicates that the interface is physically up (i.e., the cable is plugged in and the interface is receiving signals)
- mtu 1500: This denotes the Maximum Transmission Unit of the interface, which is the size of the largest packet that can be sent on the network link. 1500 bytes is typical for Ethernet.
- valid_lft forever preferred_lft forever again indicates an infinite lifetime for this address.

### 2.6 Start `tcpdump` to capture ICMPv6 packets on each machine. From lab2, ping the lab1 and lab3 IPv6 addresses using ping6(8).. You should get a return packet for each ping you have sent. If not, recheck your network configuration. Show the headers of a successful ping return packet. Show ping6 output as well as tcpdump output.

run on lab1, use lab2 to ping lab1 and lab3, and observe the terminal

```shell
sudo tcpdump -i enp0s8 icmp6
```

## 3. 