# tcpdump

{% hint style="warning" %}
If we have the ability to use `tcpdump` on the victim machine, try to listen on both interfaces (eth0 and lo) as they may contain credentials in the traffic.
{% endhint %}

### Syntax

```bash
tcpdump -i <INTERFACE> -v
tcpdump -i any -v # Any interface.
tcpdump -i <INTERFACE> -w <FILE>.pcap
tcpdump -i <INTERFACE> <FILTER> 
tcpdump -i <INTERFACE> <FILTER> and <FILTER>
tcpdump -i <INTERFACE> <FILTER> or <FILTER>
tcpdump -n -r <FILE>.pcap | awk -F " " '{print $5}' | sort | uniq -c # Skip DNS (-n), read the .pcap file (-r) and filter by IP addresses with a counter.
```

### Filter

```bash
host <IP or HOST> # Filter to capture packets arriving or leaving a host or IP.
not host <IP or HOST> # Filter to capture packets that do not arrive or leave a host or IP.
net <SUBNET> # To capture packets from an entire subnet (/8, /16, /24, etc).
dst <IP> # Filter to capture packets that have the following IP address as destination.
src <IP> # Filter to capture packets originating from the following IP address.
tcp # Filter to capture packets that have the TCP protocol.
udp # Filter to capture packets that have the UDP protocol.
port <PORT> # Filter to capture packets coming from port X, both source and output.
src port <PORT>
dst port <PORT>
```
