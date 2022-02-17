# Network Enumeration

```bash
# Hostname:
hostname -I # Also: hostname -i
cat /etc/hostname
# Interfaces:
ifconfig # The ifconfig utility is used to assign or view an address to a network interface and/or configure network interface parameters.
ip a # Ip is a utility for displaying or manipulating routing, network devices, interfaces and tunnels.
netstat -ie
cat /etc/network
# Show IPs and ports from file system:
awk '/32 host/ { print f } {f=$2}' <<< "$(</proc/net/fib_trie)" | sort -u # To view the local hosts on the network.
cat /proc/net/tcp # To view the machine ports (in hexadecimal format).
# Neighbours:
route # Also, in other versions, it can be: routel
ip route
ip n
ip neigh
cat /proc/net/route # It will be in hexadecimal format, but we can parse it as shown below.
cat /proc/net/arp
# ARP:
arp -a
arp -e
# List current conections:
netstat # Displays the network status.
netstat -natu # Linux (-u show UDP).
netstat -tulpn # It does not resolve names, shows TCP and UDP sockets, listening connections (listening or waiting for a connection) and shows the PID or program name of the sockets.
ss # Another utility to investigate sockets, ports.
ss -antlp
ss -nlt
# Hosts:
cat /etc/hosts
# DNS:
cat /etc/resolv.conf
```

### IPTables

On some occasions we may not have the ability to open a shell search on a port. This may be due to firewall rules being applied. If we have the ability to read system files, we can see the rules in the following files in the `/etc/iptables/` directory:

```bash
/etc/iptables/rules.v4
/etc/iptables/rules.v6
/etc/iptables/rules
/etc/iptables/iptables.rules
/etc/iptables/ip6tables.rules
# Other commands:
iptables -L 2>/dev/null
cat /etc/iptables/*
```

### /proc/net/route

```bash
wget https://gist.githubusercontent.com/incebellipipo/6c8657fe1c898ff64a42cddfa6dea6e0/raw/9872d19808dfa9f37819ddb90d3884df1355ee8d/routingtableparser.cpp
which g++ # If the victim machine does not have g++, we compile it on our machine
g++ routingtableparser.cpp -std=c++11 -static -o routingtableparser
./routingtableparser
```

### /proc/net/tcp

These /proc interfaces provide information about currently active TCP connections, and are implemented by tcp4\_seq\_show() in net/ipv4/tcp\_ipv4.c and tcp6\_seq\_show() in net/ipv6/tcp\_ipv6.c, respectively.

It will first list all listening TCP sockets, and next list all established TCP connections. To list its contents, we can simply do: `cat /proc/net/tcp[6]`

![](../../../../.gitbook/assets/proc\_net\_tcp.png)

The format will be in hexadecimal, but we can decode it with the following script:

{% code title="netTCP2Ports.sh" %}
```bash
#!/bin/bash

if [ $1 ]; then
    for port in $(cat $1 | awk '{print $2}' | awk '{print $2}' FS=":" | sort -u); do
        echo "[*] Open port: $(echo "obase=10; ibase=16; $port" | bc)"
    done
else
    echo -e "\\nUse: netTCP2Ports <file>\\n"
fi
```
{% endcode %}

If we only have the hexadecimal numbers (on each line), we can use the following oneliner:

```bash
for hex in $(cat content.txt); do echo "[*] Port: $(echo "obase=10; ibase=16; $hex" | bc)/tcp"; done
```
