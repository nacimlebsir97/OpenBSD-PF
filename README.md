### Contents
- [Quick intro to OpenBSD](#Quick-intro-to-OpenBSD)
- [Firewall Requirements](#Firewall-Requirements)


# Quick intro to OpenBSD

- Security Focused OS based on NetBSD.
- Founded by Theo de Raadt.
- First released in 1995.
- Open Source.
- Many builtin networking functions.
    - DNS, DHCP, WEB, EMAIL, Firewalls, Routers...
- Entire system has and continues to be designed with ground breaking security features
- OpenBSD Support many cpu architectures: AMD64, I386, ARM64, SPARC64...
- Offers 3 mains methods of installation:
	- Copy the "installXX.fs" file to USB stick
	- Burn the "installXX.iso" to CD/DVD
	- Boot over the network using the "pxeboot" file

# Firewall Requirements

- At least 2 Ethernet NICs
- Cable, DSL, Fiber, Satellite internet connection
- At least 1 Ethernet switch
- 2GB Hard drive
- 32bit or 64bit Intel AMD CPU
- Firewalling/Routing <= 100Mbps 400MHz/500MHz CPU  |  128MB/256MB RAM
- Firewalling/Routing 100mbps-1gbps + 1GHz CPU  |  1GB RAM
- Check PF Performance for more: https://www.openbsd.org/faq/pf/perf.html


---

# Basic Topology and configurations

## Interfaces configuration:

-  All interfaces are configured in **/etc/hostname.X0-9**
- The **X** portion depends on the driver in use
- The number specifies which interface to configure
- Each directive in the file is passed to ifconfig on boot
- You can make changes take effect with `sh /etc/netstart`

## **/etc/hostname.if** format

- inet specifies you are setting an IPv4 address
- Example: 192.168.1.1 is the address to configure and 255.255.255.0 is the network mask
```
inet 192.168.1.1 255.255.255.0
```
-  `!somecommand` will run that command for you 
	- example: `!route add default 192.168.1.254` to set the default gateway statically
	- example: `dhcp` to get all network configuration through DHCP

## Configure interfaces

1. Lan = em0
```
inet 192.168.1.1 255.255.255.0
```
2. Wan = em1
```
dhcp
# or static
inet 192.168.0.1 255.255.255.0
!route add default 192.168.0.254
```

---

# IP Routing

- IP Routingis the process that sends packets out the correct interface
- The destination IP Address is compared with the routing table
- If it finds a matching entry and the or the best match it sends the packet to the next hop specified in the route
- If no match is found a destination, unreachable message is sent to the sender of the packet

## Enabling IP Routing

- OpenBSD acts as a router between two or more networks only if certain kernel variable is enabled
- Otherwise OpenBSD can only send to a default gateway or its local network only
- It won't route packets that are not sourced from itself
- The file **/etc/sysctl.conf** control these kernel variables
- Check if forwarding is enabled: `sysctl -a | grep forward` and look for **net.inet.ip.forwading=0/1**
- Enable and start it:
	- Enable: `echo net.inet.ip.forwarding=1 > /etc/sysctl.conf`
	- Start: `sysctl net.inet.ip.forwarding=1`

---

# Intro to PF

- Released in 2001 as a replacement for ipfilter
- Development lead by Henning Brauer
- Filtering what ways packets can travel
- Statefull remember key details about a connection
- SRC, DST IPs, SRC, DST Ports, Protocol, Interfacem TCP seq
- NAT (Networking Address Translation)
- QoS
- AntiSpoofing
- Connection synchronization using pfsync
- Combined with CARP and PFSYNC to create high availability firewall clusters

## CARP (Common Address Redundancy Protocol)

**CARP** is a protocol designed to provide high availability for IP addresses. It's commonly used for creating a redundant or failover setup between two or more systems (often routers or firewalls). The idea is that multiple systems can share a single IP address, and one system will take over if another fails. CARP is particularly useful in scenarios where you want to ensure continuous access to a service without any downtime due to the failure of a single machine.

## PFSYNC (Packet Filter Synchronization)

**PFSYNC** is a protocol used to synchronize states between multiple systems running the Packet Filter (PF) firewall. It's designed to allow failover between two or more firewalls, ensuring that when a state (such as a connection) is established on one system, that state is shared with the backup system. This way, if the primary system fails, the backup system can immediately continue handling the traffic with minimal disruption.

---

# PF Rules

- Main rules will either be **pass/block**
- Those main rules can have sub rules
- nat-to source nat, rdr-to destination nat
- queue to classify packets for QoS policies
- Macros to store interfaces names, port numbers, IP Addresses https://www.openbsd.org/faq/pf/macros.html
- Table to store large rangers of IP Addressess


