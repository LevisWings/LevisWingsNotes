# Display Filters

### Protocols

Simply type in the protocol name:&#x20;

```bash
icmp, arp, http, dns, udp, etc.
dns and http # DNS and HTTP
```

### Fields

{% embed url="https://www.wireshark.org/docs/dfref" %}

```bash
http.host # Extract "Host" header.
ftp.request.command # FTP request command
```

### Search by word

**Contains** searches all the headers of a frame (all the layers of the TCP/IP model) for a word that is the same (**case sensitive**):&#x20;

```bash
frame contains <WORD>
```

**Matches** is the same as contains but is more flexible, since it is **case insensitive**:

```bash
frame matches <WORD>
```

### Observations

```bash
http # Filter by protocol: Only captures application layer traffic.
tcp.port == 80 # Filter by port: Application layer and transport layer traffic.
```

### Simple examples

```bash
ip.addr == 192.168.0.1 # Filter packets that include the IP.
!ip.addr == 192.168.0.1 # Filter packets that do not include the IP. If we do: ip.addr != 192.18.0.1 it may seem the same but it is not.
ipv6.addr == 2406:da::3f42:487a # Filter packets that include IPv6.
ip.src == 192.168.0.1 # Filter out packets that include the source IP.
ip.dst == 192.168.0.1 # Filter packets that include the destination IP.
ip.host == www.cnn.com # Filter packets that include the hostname.
ip.addr == 192.168.0.0/24 # Filter packets that include subnet IPs.
!ip.addr == 192.168.0.0/24 # Filter packets that do not include IPs from the subnet.
eth.addr # To filter packets according to their MAC address.
eth.src # Source MAC.
eth.dst # Destination MAC.
```

### Advanced examples

```bash
tcp.src.port != 80 # Filter out packets that do not contain TCP source port 80
udp.port == 53 # Filter only for UDP traffic from port 53
frame.time_relative > 1 # Filter out time greater than 1s from first frame
tcp.window_size < 1460 # Filter TCP window longer than 1460 bytes
dns.count.answers >= 10 # Filter DNS responses with more than 10 IP addresses
ip.ttl <= 10 # Filter out packets with TTL values less than equal 10
http contains "GET" # Filter HTTP packets containing GET command
net <SUBNET>
```

### Tricks (for tshark)

An easy way to identify **field names** (`-T fields -e <FIELD NAME>`) in Wireshark is to navigate to the Packet Details in the capture, highlight the interesting field, then view the bottom left corner.

![](../.gitbook/assets/field\_name\_wireshark.png)
