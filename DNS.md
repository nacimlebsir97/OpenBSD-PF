This Unbound DNS configuration transforms an OpenBSD system into a reliable local DNS resolver for the internal network. It securely resolves domain names, performs caching to speed up lookups, and supports DNSSEC to validate responses. By keeping DNS traffic local and under your control, it enhances both privacy and performance.

# Intro
- DNS (Domain Name System)
- Uses UDP and TCP port 53
- Resolves human-readable domain names (e.g., example.com) to IP addresses (e.g., 93.184.216.34)
- Unbound is a lightweight, open-source resolver developed with security in mind
- Acts as a recursive resolver by querying the DNS hierarchy directly (root ➜ TLD ➜ authoritative server)
- Caching improves efficiency by storing previous query results
- DNSSEC support ensures integrity and authenticity of DNS responses
- Frequently used on firewalls, routers, and servers to reduce reliance on external DNS providers
- Supports advanced features like ad-blocking, domain overrides, and query forwarding

## Why It's Important
Running Unbound locally provides increased privacy, security, and speed by avoiding third-party DNS services. It reduces external traffic via caching, supports custom domain resolution, and enables domain blocking for ad or threat filtering. With DNSSEC enabled, it also guards against DNS spoofing and ensures trustworthy responses.

# Contents
- [README](README.md)
- [PF](PF.md)
- [DHCP](DHCP.md)
- [Unbound Configuration](#unbound-configuration)
- [Domain Overrides](#domain-overrides)
- [Monitor some queries](#monitor-some-queries)


# Unbound Configuration

1. Back up the configuration file since unbound is part of OpenBSD and not a package to install:
```
mv /var/unbound/etc/unbound.conf /var/unbound/etc/unbound.conf.bak
```

2. Create a new one and set the configuration in:
```
server:
		interface: 0.0.0.0        # listens on all interfaces
		
		# Allow specific subnets to query this DNS server
		access-control: 192.168.1.0/24 allow
		access-control: 127.0.0.1 allow
		access-control: ::1 allow

		# Deny any other IPv4/IPv6 address
		access-control: 0.0.0.0/0 deny
		access-control: ::/0 deny

# Forward all unresolved DNS queries to these upstream DNS servers
forward-zone:

		name: "."    # which means forward any top level domains, to the next line dns addresses 

		forward-addr: 9.9.9.9
		forward-addr: 1.1.1.1
		forward-first: yes
```

3. Check the file for syntax errors: `unbound-checkconf`
4. Enable and start the unbound service: `rcctl enable unbound && rcctl start unbound`

## Domain Overrides:

- In the server section
```
# It's more effective to use `always_nxdomain` or `always_null` for blocking instead of resolving to localhost (127.0.0.1), which can still be reachable if services are running locally.
# always_null drops the request silently without pointing it to 127.0.0.1
# Just an example

local-zone: "somebadsite.com" static
local-data: "somebadsite.com IN A 127.0.0.1"
local-data-ptr: "127.0.0.1 somebadsite.com"
```


### With a blacklist file:

Update **unbound.conf** (In the server section) and add: `include: "/path/to/unbound.blacklist.conf"`

```
# blacklist.conf

server:
local-zone: "0--0.ml." always_null
local-zone: "0-0.fr." always_null
local-zone: "0-000.store." always_null
local-zone: "0-dishpurchasingcorporation.hb-api.tt.omtrdc.net." always_null
```


# Monitor some queries
1. Check with netstat: `netstat -an | less`
2. Check with tcpdump: 
	- On the Server: `tcpdump -ni bge0 udp port 53`
	- On the client: `dig @192.168.1.1 www.github.com`
