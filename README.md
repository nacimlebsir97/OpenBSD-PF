# Intro to OpenBSD

  OpenBSD is a security-focused, open-source operating system originally forked from NetBSD and founded by Theo de Raadt in 1995. Renowned for its rigorous code auditing and pioneering security features, OpenBSD is widely respected in the cybersecurity and networking communities. It includes a comprehensive suite of built-in networking tools such as DNS, DHCP, web and email servers, firewalls, and routing capabilities. Supporting a wide range of CPU architectures, including AMD64, i386, ARM64, and SPARC64. OpenBSD offers flexible installation methods, including USB, CD/DVD, and PXE boot over the network. Its minimalist design and security-first approach make it ideal for firewalls, servers, and embedded systems.

# Intro to PF

  PF (Packet Filter) is OpenBSD’s built-in firewall system, introduced in 2001 as a secure and flexible replacement for IPFilter, with development led by Henning Brauer. It provides advanced packet filtering capabilities, allowing administrators to control how traffic flows through the network based on criteria like source/destination IP, ports, protocol, interface, and TCP sequence numbers. PF is a stateful firewall, meaning it tracks connection states to make intelligent decisions about traffic. In addition to filtering, PF supports Network Address Translation (NAT), Quality of Service (QoS), anti-spoofing, and connection synchronization via pfsync. When combined with CARP (Common Address Redundancy Protocol), PF enables the deployment of high-availability firewall clusters, making it a powerful tool for robust and secure network infrastructure.

## CARP (Common Address Redundancy Protocol)

  CARP is a protocol designed to provide high availability for IP addresses. It's commonly used for creating a redundant or failover setup between two or more systems (often routers or firewalls). The idea is that multiple systems can share a single IP address, and one system will take over if another fails. CARP is particularly useful in scenarios where you want to ensure continuous access to a service without any downtime due to the failure of a single machine.

## PFSYNC (Packet Filter Synchronization)

  PFSYNC is a protocol used to synchronize states between multiple systems running the Packet Filter (PF) firewall. It's designed to allow failover between two or more firewalls, ensuring that when a state (such as a connection) is established on one system, that state is shared with the backup system. This way, if the primary system fails, the backup system can immediately continue handling the traffic with minimal disruption.


# Contents
- [Firewall Requirements](PF.md#firewall-requirements)
- [Interfaces](PF.md#interfaces)
- [IP Routing](PF.md#ip-routing)
- [PF Rules](PF.md#pf-rules)
- [PF Configuration File](PF.md#pf-configuration-file)
- [PF Commands](PF,md#pf-commands)
- [Monitoring](PF.md#monitoring)
- [Bonus](PF.md#bonus)
- [Vlans](PF.md##vlans)
- [CARP](PF.md#carp)
- [PFSYNC](PF.md#pfsync)
- [Port Forwarding](PF.md##port-forwarding)
