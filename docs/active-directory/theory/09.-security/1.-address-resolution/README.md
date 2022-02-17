# 📓 1. Address resolution

From a security perspective, the resolution of address is quite relevant, since if a user/program can be tricked into connecting to an erroneous machine, therefore many attacks can be performed like:

* Person-in-The-Middle (PitM) : This allows an attacker to intercept the communications of the victim and read/manipulate information (if it is not properly encrypted/signed) sent or received by the victim.
* [NTLM Relay](https://en.hackndo.com/ntlm-relay/): An attacker can use an NTLM authentication from the victim and redirect it to a desired server in order to get access to it.
* [NTLM crack](https://0xdf.gitlab.io/2019/01/13/getting-net-ntlm-hases-from-windows.html): Even if you are not able to relay the NTLM authentication, you can try to crack the NTLM hash and recover the user password.

But, what addresses need to be resolved? There are three types of addresses that are used by machines:

### MAC address

The [MAC](https://en.wikipedia.org/wiki/MAC\_address) (Media Control Access) is the address that uniquely identifies each computer in the world (concretely each computer network card). The MAC address is the one used to send messages in the Ethernet protocol, in the Link layer, that **communicates the computers in the same network**. Each network card has an associated unique MAC address that allows to identify it in a network. Usually the MAC address remains constant, but [it can be changed](https://www.howtogeek.com/192173/HOW-AND-WHY-TO-CHANGE-YOUR-MAC-ADDRESS-ON-WINDOWS-LINUX-AND-MAC/).

A MAC address is composed by 6 bytes like `01:df:67:89:a4:87`, where the first 3 bytes indicates the [MAC vendor](https://gist.github.com/aallan/b4bb86db86079509e6159810ae9bd3e4) and the last 3 are an unique identification for each network card of that vendor.

### IP address

The [IP address](https://en.wikipedia.org/wiki/IP\_address) is the one used by the IP protocol, in the Internet layer, that **allows communication between computer of different networks**. Unlike MAC addresses, the IPs are not configured in the network cards but need to be set by an external entity, by using a protocol like [DHCP](https://zer1t0.gitlab.io/posts/attacking\_ad/#dhcp) or setting an static IP address. So a computer can change its IP address at any time.

While traversing the networks, the IP addresses need to be mapped to MAC addresses to allow communication inside of the different networks that route the packets. For this purpose, the [ARP](https://zer1t0.gitlab.io/posts/attacking\_ad/#arp) protocol is used.

There are two versions of IP addresses, IPv4, that are composed by 4 bytes (32 bits) like `23.78.167.99`,and IPv6, composed by 16 bytes like `2001:db8:85a3:8d3:1319:8a2e:370:7348`. Usually IPv4 is used.

### Hostnames

Since IP addresses are hard to remember, computers are also assigned a name that is more human-friendly like `pepe-machine`, known as hostname. However, since computers need the IP address to communicate, it is possible to associate the hostnames to IP addresses by using protocols like **DNS**, **NetBIOS**, **LLMNR** or **mDNS**.

Therefore, the following processes are vital for a computer to being able to find the correct address to communicate:

### Hostname-IP resolution

The computers need to be able to map the hostname of a machine to its correct IP address. For this purpose there are two strategies:

1. Asking to a central server for the hostname resolution, which is the approach used by **DNS**. If an attacker can become the central server, it can map the hostnames to choose addresses.
2. Sending a broadcast request with the hostname to peers asking to the computer with the given hostname to identify itself. This approach is used by protocols like **NetBIOS**, **LLMNR** or **mDNS**, where any computer in the network can respond to the request, so an attacker could listen waiting for requests and respond them identify itself as the target computer.

### IP-MAC resolution

Once the IP is identified, the computers need to know to which computer (network card) belongs that IP, so they ask for its MAC. For this they use the **ARP** protocol that works by sending a broadcast request to the internal network and waiting to the correct host to identify itself. The attacker could respond to the request identifying itself as the target to receive the connection.

### IP configuration

In order to use an IP and being able to find the central server to resolve IPs, computers need to be configured. This configuration can be done manually for each computer or it can be done by using a protocol like **DHCP**, where a server provides with configuration options to the computers of the network.

However, since computers don't have anything configured when they talk with the DHCP server, they need to look blindly for it by sending broadcast requests that any other machine can respond. So there is an opportunity for an attacker to misconfigure it by responding to these requests and providing to the client fake configuration parameters, usually pointing to a DNS server controlled by the attacker.

So, let's see how address resolution can be attacked in Active Directory and other computer networks.
