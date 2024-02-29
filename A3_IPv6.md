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

## 3. IPv6 Router Advertisement Daemon

> Instead of having to manually assign the addresses to the interfaces, this can be done by the router, so they automatically get an address assigned to them.
>
> Set up Router Advertisement Daemon on lab1 to automatically assign IPv6 addresses to VMs connected to intnet1 and intnet2.
>
> - On lab2 and lab3: Remove all static addresses from the intnet interfaces and run the interfaces down.
> - lab1: Install IPv6 Router Advertisement Daemon (radvd). Modify the content of radvd.conf file to be used in your network (If radvd.conf file does not exist create one under /etc directory). Radvd should advertise prefix fd01:2345:6789:abc1::/64 on intnet1 (enp0s8) and fd01:2345:6789:abc2::/64 on intnet2 (enp0s9). Start the router advertisement daemon (radvd).
> - Check using tcpdump that router advertisement packets are sent to enp0s8 and enp0s9 of lab1 periodically. If you can’t see any packets sent, edit the conf file.
> - Start tcpdump on lab2 and capture ICMPv6 packets. Bring the interfaces on lab2 and lab3 up. Stop capturing packets after receiving first few ICMPv6 packets. Make sure the addresses that are assigned to the interfaces are received from the router advertisement.
> - Ping lab3 from lab2 using the IPv6 address allocated by radvd. You should get a return packet for each ping you have sent. If not, recheck your network configuration.

On `lab2` and `lab3`:

Remove all static IPv6 addresses from the interfaces.

```bash
sudo ip -6 addr flush dev enp0s8
```

Bring the interfaces down.

```bash
sudo ip link set enp0s8 down
```

On `lab1`:

Install `radvd` if it's not already installed.

```bash
sudo apt update
sudo apt install radvd
```

Create or edit the `radvd.conf` file located in the `/etc` directory.

```bash
sudo nano /etc/radvd.conf
```

Insert the following configuration into the `radvd.conf` file:

```plaintext
interface enp0s8 {
    AdvSendAdvert on;
    prefix fd01:2345:6789:abc1::/64 {
        AdvOnLink on;
        AdvAutonomous on;
        AdvRouterAddr on;
    };
};

interface enp0s9 {
    AdvSendAdvert on;
    prefix fd01:2345:6789:abc2::/64 {
        AdvOnLink on;
        AdvAutonomous on;
        AdvRouterAddr on;
    };
};
```

Start the `radvd` service.

```bash
sudo systemctl start radvd
```

Confirm that `radvd` is running and sending router advertisements.

```bash
sudo tcpdump -i enp0s8 -vv icmp6
sudo tcpdump -i enp0s9 -vv icmp6
```

On `lab2` and `lab3`:

Bring the interfaces back up.

```bash
sudo ip link set enp0s8 up
```

Start capturing ICMPv6 packets with `tcpdump`.

```bash
sudo tcpdump -i enp0s8 -vv icmp6
```

Wait to receive the router advertisements and confirm that an IPv6 address within the advertised prefix is assigned to the interface.

### 3.1 Explain your modifications to radvd.conf. Which options are mandatory?

- `AdvSendAdvert on;` (mandatory): This option enables the sending of router advertisements on the specified interface. When set to on, radvd will periodically send out router advertisement messages, which are used by clients to configure their IPv6 addresses and other settings.
- `AdvOnLink on;` (Common): This option indicates that the advertised prefix is on-link, meaning that addresses within this prefix are directly reachable on the attached link (network segment). It tells the client that hosts on this prefix are reachable without going through a router.
- `AdvAutonomous on;` (Common): This option enables Stateless Address Autoconfiguration (SLAAC) for the prefix. It allows clients to automatically configure their own IPv6 addresses using the advertised prefix combined with their interface identifier (derived from the MAC address or generated through privacy extensions).
- `AdvRouterAddr on;` (Optional): This option indicates that the address of the interface from which the advertisement is sent should be used as the default router address for the advertised prefix. However, this option is not typically used in radvd configurations and is not part of the standard Router Advertisement specification. In most cases, the router's address is inferred by clients from the source address of the received advertisement.

### 3.2 Analyze captured packets and explain what happens when you set up the interface on lab2

The provided `tcpdump` output captures an IPv6 Router Advertisement (RA) packet sent from `tlab1` to the `ip6-allnodes` multicast address, which is a special IPv6 address that all IPv6-enabled devices on the local network segment listen to.

- **Source and Destination**: The packet is from `tlab1` and is sent to `ip6-allnodes`, indicating it's a Router Advertisement meant for all IPv6 nodes on the local network.

- **ICMPv6 Router Advertisement**: The packet is an ICMPv6 message with a type indicating it's a Router Advertisement. RAs are used by routers to advertise their presence along with various IPv6 network parameters.

- **Hop Limit**: Set to 64, which is a common default value for the hop limit in IPv6 packets.

- **Flags**: The RA contains flags that are set to `[none]`, indicating no specific flags are set. Commonly used flags in RAs include the Managed Address Configuration (M) and Other Configuration (O) flags, but neither is set here.

- **Router Lifetime**: Set to 1800 seconds (30 minutes), indicating how long the receiver should consider this router as a default router.

- **Reachable Time and Retrans Timer**: Both are set to 0ms, indicating that the sender has not specified particular values for these parameters. The reachable time specifies how long a neighbor is considered reachable after a reachability confirmation event, and the retrans timer specifies the time between retransmitted Neighbor Solicitation messages.

- **Prefix Info Option**: This part of the RA specifies the IPv6 prefix that is being advertised (`fd01:2345:6789:abc1::/64`) for automatic address configuration. The flags within this option (`onlink`, `auto`, `router`) indicate that the prefix is on-link, addresses within the prefix can be auto-configured, and the sender is a router. The valid time is set to 86400 seconds (1 day), and the preferred time is set to 14400 seconds (4 hours).

- **Source Link-Address Option**: Contains the link-layer address of the sender (`08:00:27:28:20:7c`). This is used by receiving nodes to update their Neighbor Discovery cache with the link-layer address of the router.

**What Happens on `lab2` Upon Setup**:

When the interface on `lab2` is set up and it receives this Router Advertisement:

1. **Router Discovery**: `lab2` learns about the presence of a router (`tlab1`) on its network segment.

2. **Prefix Information**: `lab2` receives the prefix `fd01:2345:6789:abc1::/64` and understands that it can auto-configure an IPv6 address within this prefix using SLAAC (Stateless Address Autoconfiguration).

3. **Address Configuration**: Based on the prefix information in the RA, `lab2` will generate an IPv6 address for its interface. This is typically done by appending the interface's EUI-64 (an identifier derived from the MAC address) to the prefix, or by using a privacy extension if configured.

4. **Router as Default Gateway**: The Router Lifetime value tells `lab2` that it can use `tlab1` as a default gateway for a specified time (1800 seconds in this case).

5. **Neighbor Cache Update**: The source link-layer address option allows `lab2` to update its Neighbor Cache with the MAC address of `tlab1`, facilitating layer 2 communications.

After these steps, `lab2` will have an IPv6 address within the advertised prefix and will be able to communicate with other IPv6 nodes on the network and beyond, using `tlab1` as its default gateway.

### 3.3 How is the host-specific part of the address determined in this case?

Stateless Address Autoconfiguration (SLAAC).

EUI-64 Method: This is a common method where the interface identifier is derived from the network interface's MAC (Media Access Control) address. The MAC address is 48 bits, and to convert it to a 64-bit EUI-64 format, the MAC address is split in half, the hexadecimal value 0xFFFE is inserted in the middle, and the Universal/Local (U/L) bit (the 7th bit of the first byte) is flipped. This process generates a unique 64-bit identifier for the interface.

### 3.4 Show and explain the output of a traceroute(1) from lab2 to lab3

```plaintext
vagrant@tlab2:~$ traceroute fd01:2345:6789:abc2:a00:27ff:feb2:f6c0
traceroute to fd01:2345:6789:abc2:a00:27ff:feb2:f6c0 (fd01:2345:6789:abc2:a00:27ff:feb2:f6c0), 30 hops max, 80 byte packets
 1  fd01:2345:6789:abc1:a00:27ff:fe28:207c (fd01:2345:6789:abc1:a00:27ff:fe28:207c)  0.783 ms  0.429 ms  0.852 ms
 2  fd01:2345:6789:abc2:a00:27ff:feb2:f6c0 (fd01:2345:6789:abc2:a00:27ff:feb2:f6c0)  1.519 ms  1.251 ms  1.416 ms
```

2 hops

first to lab1 as a router
second to lab3

## 4. Configure IPv6 over IPv4

> Ideally, IPv6 should be run natively wherever possible, with IPv6 devices communicating with each other directly over IPv6 networks. However, the move from IPv4 to IPv6 will happen over time. The Internet Engineering Task Force (IETF) has developed several transition techniques to accommodate a variety of IPv4-to-IPv6 scenarios. One type of IPv4–to–IPv6 transition mechanism is translation including NAT64, Mapping of Address and Port (MAP), IPv6 Rapid Deployment (6rd), etc.
>
> In this part of the assignment the goal is to demonstrate two ipv6 only nodes communicating with each other and the global internet through an ipv4 link. You will need to spin up another VM, lab4 for this part of the assignment to setup the network shown below, which has two IPv6 only nodes and two nodes with both IPv6 and IPv4 capabilities but only an IPv4 link connecting them to each other
>
> 1. Reset the networking on lab1, lab2 and lab3 back to default.
>
> 2. Create a new VM named lab4. Lab4 should have a NAT adapter for you to be able to ssh into and administer it, so set up port forwarding accordingly
>
> 3. On lab3 and lab4, add a network adapter of type internal network and name it intnet3
>
> 4. On lab2 and lab4, disable all static IPv4 addresses on the intent adapters. Create an IPv6 link between lab2 and lab1 assigning static addresses from the fd01:2345:6789:abc1::/64 subnet, similarly create an IPv6 link between lab3 and lab4 assigning addresses from the subnet fd01:2345:6789:abc2::/64.
>
> 5. Between lab1 and lab3 setup an IPv4 link with static addresses from 192.168.0.0/16
> 6. Make sure only lab3 has internet access. Configure your routing so that lab3 is used as the internet gateway

