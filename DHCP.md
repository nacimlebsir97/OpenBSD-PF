This DHCP configuration enables an OpenBSD system to act as a local DHCP server for the internal network. It automatically assigns IP addresses and essential network settings, such as subnet mask, gateway, and DNS to connected LAN clients. By managing IP leases internally, it simplifies network setup, ensures consistent addressing, and reduces manual configuration on each device.

# Intro

- DHCP (Dynamic Host Configuration Protocol)
- Uses UDP port 67 server and 68 for the client
- As a client we ask the DHCP server for the proper network information
- As a server we assign the proper network information to the client by giving that client a lease.
- Main parameters server assigns include **Network ID**, **Subnet Mask**, **Default gateway**, **DNS Servers**, **Domain name**
	- Optional parameters can include **TFTP server**, **NTP Server**...
- IP Addresses are configured in a pool/range
- When the server gets a request it takes an address from the pool and links it with a lease
- Leases expire after a time configured on the server
- Then those addresses are realses back to the pool
- The client can reqiest the lease it had before it expires

# DHCP Configuration

1. Configuration file: **/etc/dhcpd.conf**
```
option domain-name "homenetwork.local";
option domain-name servers 9.9.9.9, 1.1.1.1;    # Can be changed to OpenBSD host IP if it also acts as a DNS server

subnet 192.168.1.0 netmask 255.255.255.0 {
	range 192.168.1.100 192.168.1.200;
	option router 192.168.1.1;
}
```

2. Test the file: `dhcpd -n`
3. Enable the service: `rcctl enable dhcpd`
4. Each pool is only used if an interface is active with an IP Address in that pool subnet 
