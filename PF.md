# Contents
- [README](README.md)
- [Firewall Requirements](#firewall-requirements)
- [Interfaces](#interfaces)
- [IP Routing](#ip-routing)
- [PF Rules](#pf-rules)
- [PF Configuration File](#pf-configuration-file)
- [PF Commands](#pf-commands)
- [Monitoring](#monitoring)
- [Bonus](#bonus)
- [Vlans](#vlans)
- [CARP](#carp)
- [PFSYNC](#pfsync)
- [Port Forwarding](#port-forwarding)

  This PF (Packet Filter) configuration turns an OpenBSD system into a simple but effective stateful firewall and NAT router, allowing all traffic from the LAN to access the internet, while blocking all unsolicited inbound connections from the WAN. Only responses to internal requests are permitted back in, ensuring basic network protection and isolation. The setup includes NAT, state tracking, and interface-based rules for a secure default-deny stance.

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

# Interfaces configuration:

- All interfaces are configured in **/etc/hostname.X0-9**
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

# PF Rules

- Main rules will either be **pass/block**
- Those main rules can have sub rules
- nat-to source nat, rdr-to destination nat
- queue to classify packets for QoS policies
- Macros to store interfaces names, port numbers, IP Addresses https://www.openbsd.org/faq/pf/macros.html
- Table to store large rangers of IP Addressess

## /etc/pf.conf format

- Rules are stored in /etc/pf.conf
- Rules are processed from top to bottom and last match wins
- Processing logic can be bypassed with the **quick** keyword
- rules examples:
	- `pass out on em1 from 192.168.0.0/24 to any nat-to (em1) keep state`
	- `pass in on em1 inet proto tcp from any to tp 203.0.113.254 port 80 rdr-to 192.168.0.3 port 80`

---

# PF Configuration File

```
# Set macros of internal & external interfaces
  LAN="em0"
  WAN="em1"

# Skip filtering on loopback
  set skip on lo0

# Set block policy
  set block-policy drop              # drop is better than reject if you don't want send any info back, reject is better for troubleshooting

# Drop all the traffic
  block drop all

# Allow LAN traffic
  pass in on $LAN from $LAN:network to any keep state

# Pass the LAN traffic to the internet
  pass out on $WAN from $LAN:network to any nat-to ($WAN) keep state

# Allow the firewall to talk
  pass out on $WAN from $WAN:network to any keep state
```

---

# PF Commands

- Check file for valid syntax: `pfctl -nf /etc/pf.conf`
- Load rules into pf: `pfctl -f /etc/pf.conf`
- Enable PF: `pfctl -e`
- Disable PF: `pfctl -d`
- Check status: `pfctl -sr` or `pfctl -s info`
- Show Statistics: `pfctl -si`
- Show states: `pfctl -ss`

---

# Monitoring

- `systat vmstat` - Shows RAM/CPU usage and Interrupts in realtime
- `systat ifstat` - Shows packets per second on all interfaces in realtime
- `systat netstat` - Shows connections sourced from the firewall in realtime
- `netstat -an` - Shows active/listening network connections
- `pftop` - Shows pf state table contents in realtime
- `iftop` - Shows bandwidth of connections through the firewall
- `vnstat` Keeps historic information amount of data usage on all interfaces
- `tcpdump` Shows any packet information you need in realtime

---

# Bonus

## Vlans

- Allows one or more physical interfaces to be separated into multiple IP Networks

### Configuration

1. Edit **/etc/hostname.vlan10**
```
inet 192.168.10.1 255.255.255.0 10 vlandev em0
```
2. Once we restart networking we will have a new vlan10 interface

### Integrate to PF
```
OFFICE=vlan10

pass in on $OFFICE from $OFFICE:network to any keep state
pass out on $WAN from $OFFICE:network to any nat-to ($WAN) keep state
```


## CARP

- Allows two or more hosts to share a virtual IP Address that can switch between the groups of host for failover

### Configuration

- In this case em0 (which is the LAN interface) is 192.168.1.10
1. Edit **/etc/hostname.carp10**
```
inet 192.168.1.1 255.255.255.0 192.168.1.255 vhid 10 pass carppass carpdev em0
```
2. Once reloading networking, notice the status of carp10 interface (from backup to master)

#### Configure the backup node to be backup not the master

```
inet 192.168.1.1 255.255.255.0 192.168.1.255 vhid 10 advskew 100 pass carppass carpdev em0
```

### Integrate to PF

- Just in case needed to destroy the interface and recreate it again: `ifconfig carp10 destroy`
```
pass proto carp
```


## PFSYNC

- Allows two or more firewalls to share state table information for transparent connection failover

### Configuration

1. Edit **/etc/hostname.pfsync0**
```
up syncdev em0
```
2. Restart networking

### Pf integration

```
pass proto pfsync
```

## Port forwarding

- Also known as destination NAT, allows hosts on a public IP Address space to access hosts on a private one

### PF rule

- Example: forward external (WAN) connections to an internal HTTP server on port 80.
```
pass in on $WAN inet proto tcp from any to any port 80 rdr-to 192.168.1.100 port 80 keep state
pass out on $LAN inet proto tcp from any to 192.168.1.100 port 80 keep state
```

### Explanations
- Rule 1 (Inbound NAT / Port Forwarding): The rule redirects traffic hitting your public IP on port 80 to the internal web server.
- Rule 2 (Allowing Outbound Response): The rule allows the traffic from your NAT firewall to reach the internal server and receive responses.

#### State table
- When PF passes a packet with the `keep state` keyword (or by default, since most `pass` rules implicitly use it), it records that connection in its state table.
- Without the state table, you'd need to manually write both directions (`in` and `out`) of every connection, which is painful and error-prone.

