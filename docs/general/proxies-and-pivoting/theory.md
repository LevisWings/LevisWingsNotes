# 📓 Theory

### Pivoting

Often, during a penetration test or security assessment, everything starts with an external network, with the investigation and pentesting of machines and services available on the global network. An attempt is made to find a security hole and, if successful, a penetration is made into the local or internal network to capture as many systems as possible.

Local network traffic is non-routable, i.e. other computers that are physically connected to this network can access the local network resources, and the attacker cannot access them.

Therefore, **pivoting** is a set of techniques that allow an attacker to gain access to local resources, in essence, making traffic that is normally non-routable routable. Pivoting helps an attacker set up the working environment to use the tools as if they were on the organization's local network using the previously compromised machine as a foothold.

### Tunneling:

In the physical world, tunneling is a way to traverse terrain or boundaries that normally could not be crossed. Similarly, in networks, tunnels are a method of transporting data across a network using protocols that are not supported by that network. Tunnels work by encapsulating packets: wrapping packets inside other packets. (Packets are small pieces of data that can be reassembled at their destination to form a larger file).

Tunnels are often used in virtual private networks (VPNs). It can also establish efficient and secure connections between networks, enable the use of unsupported network protocols and, in some cases, allow users to bypass firewalls.

Tunneling is useful because it allows you to connect to any other machine on the network. Once an internal machine is compromised, it can be used as a platform for other attacks. Simply installing tools such as nmap on the compromised machine will not allow you much freedom in terms of actually scanning the network, beyond preliminary scans.

Getting a VPN connection into a network is practically the holy grail of tunneling. It is the logical equivalent of actually being connected to that network directly. As such, tools that cannot be easily run through a proxy, or on a remote system, can be run with relative ease. Most VPN protocols operate as a link-layer abstraction, such that arbitrary network protocols can be translated to the remote network. This is extremely useful for tools such as ARP spoofing, where a normal SOCKS proxy will not allow arbitrary packets to be sent. This makes it very convenient to turn on your BT5 instance (or whatever pentesting box you are using) and jump directly to the network. In addition, VPN traffic is encrypted and is likely to be completely ignored by a firewall.

Alternatively, you can use a SOCKS proxy or even a Tor hidden service to facilitate your tunneling needs. These are more limited, due to the level at which they relay traffic, but are easier to configure and easier to customize in order to hide traffic from a firewall / IDS.

### Port Forwarding

Port forwarding, sometimes called port tunneling, is the action of redirecting a network port from one network node to another. This technique can allow an external user to access a port on a private IP address (within a LAN) from the outside via a NAT enabled router.

### Local Forwarding (SSH)

Local forwarding is used to forward a port from the client machine to the server machine. Basically, the SSH client listens for connections on a configured port, and when it receives a connection, it tunnels the connection to an SSH server. The server connects to a configurated destination port, possibly on a different machine than the SSH server.

Typical uses for local port forwarding include:

* Tunneling sessions and file transfers through jump servers
* Connecting to a service on an internal network from the outside
* Connecting to a remote file share over the Internet

Quite a few organizations for all incoming SSH access through a single jump server. The server may be a standard Linux/Unix box, usually with some extra hardening, intrusion detection, and/or logging, or it may be a commercial jump server solution.

Many jump servers allow incoming port forwarding, once the connection has been authenticated. Such port forwarding is convenient, because it allows tech-savvy users to use internal resources quite transparently. For example, they may forward a port on their local machine to the corporate intranet web server, to an internal mail server's IMAP port, to a local file server's 445 and 139 ports, to a printer, to a version control repository, or to almost any other system on the internal network. Frequently, the port is tunneled to an SSH port on an internal machine.

### Remote Forwarding (SSH)

