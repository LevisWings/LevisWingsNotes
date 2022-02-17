# Enumeration (methodology)

## Introduction

As always, enumeration is the key to success. Our first step when attempting to pivot through a network is to get an idea of what's around us.

There are five possible ways to enumerate a network through a compromised host:

* Using material found on the machine. The hosts file or ARP cache, for example.
* Using pre-installed tools.
* Using statically compiled tools.
* Using scripting techniques.
* Using local tools through a proxy.

These are written in the order of preference. Using local tools through a proxy is incredibly slow, so should only be used as a last resort.

## Using material found on the machine

{% tabs %}
{% tab title="Linux" %}
```bash
/etc/hosts
/etc/resolv.conf
```
{% endtab %}

{% tab title="Windows" %}
```shell
C:\Windows\System32\drivers\etc\hosts
```
{% endtab %}
{% endtabs %}

## Using pre-installed tools

{% tabs %}
{% tab title="Linux" %}
```bash
ifconfig
hostname -I # Also: hostname -i
arp -a
netstat -nat
ps aux
nmcli dev show # Command as an alternative for reading the resolv.conf file.
# Nmap:
nmap -sn <IP RANGE> # Discover hosts
nmap -PE -sn 10.10.68.220/24 # Discover hosts
nmap -p- -sT -n -Pn -iL hosts.txt # Discover ports
# arp-scan:
arp-scan -I <INTERFACE> --localhost
arp-scan <IP RANGE>
```
{% endtab %}

{% tab title="Windows" %}
```shell
ipconfig # Also: ipconfig /all
```
{% endtab %}
{% endtabs %}

## Using statically compiled tools

If there are no useful tools already installed on the system, and the rudimentary scripts are not working, then it's possible to get static copies of many tools. These are versions of the tool that have been compiled in such a way as to not require any dependencies from the box. In other words, they could theoretically work on any target, assuming the correct OS and architecture.

### Download

{% content-ref url="../../misc/tools.md" %}
[tools.md](../../misc/tools.md)
{% endcontent-ref %}

#### Resources (with static binaries):

{% embed url="https://github.com/andrew-d/static-binaries" %}

{% embed url="https://github.com/ernw/static-toolbox/releases" %}

{% embed url="https://sourceforge.net/projects/unix-utils/files" %}

### Practice

{% tabs %}
{% tab title="Linux" %}
With a static nmap binary, we can discover hosts on a given subnet and then discover all their open ports:

```bash
nmap -sn <SUBNET>
nmap -p- -sT -n -Pn -iL hosts.txt
```
{% endtab %}
{% endtabs %}

{% content-ref url="socat.md" %}
[socat.md](socat.md)
{% endcontent-ref %}

## Using scripting techniques

{% tabs %}
{% tab title="Linux" %}
### Host discovery

We can use scripting techniques in the Bash language to discover hosts. In the following script, we need the ping command:

{% hint style="danger" %}
It's worth noting as well that you may encounter hosts which have firewalls blocking ICMP pings (Windows boxes frequently do this, for example).
{% endhint %}

```bash
#!/bin/bash
for i in $(seq 1 254); do
    timeout 1 bash -c "ping -c 1 .$i" &>/dev/null && echo -e "[+] Host: .$i - ACTIVE" &
done; wait
```

If you don't have the ping binary, you can look further into a technique to employ Host Discovery + Port scan.

### Port discovery

After getting the hosts discovered, it is time to discover their open ports.

{% code title="portDiscovery.sh" %}
```bash
#!/bin/bash

hosts=("<HOST>" "<HOST>")

for host in ${hosts[@]}; do
	echo -e "\n[+] Host: $host\n"
	for port in $(seq 1 1024); do
		timeout 1 bash -c "echo '' > /dev/tcp/$host/$port" &>/dev/null && echo -e "\t[*] Port $port - OPEN" &
	done; wait
done
```
{% endcode %}

Brief explanation:

* We know that by doing `echo '' > /dev/tcp/$host/$port`, we are sending empty content over TCP to a host port. If the computer and port is open, nothing happens, but we have a command that was successfully executed.
* If we execute the above command on a closed port, then it will wait for a long time until it throws an error. But this long time can be avoided by using the `timeout` command.

We can also use oneliners (for example, with nc).

```bash
# Top ports:
top10=(20 21 22 23 25 80 110 139 443 445 3389); for i in "${top10[@]}"; do nc -w 1 <IP> $i && echo "Port $i is open" || echo "Port $i is closed or filtered"; done
top10=(20 21 22 23 25 80 110 139 443 445 3389); for i in "${top10[@]}"; do (echo > /dev/tcp/<IP>/"$i") > /dev/null 2>&1 && echo "Port $i is open" || echo "Port $i is closed"; done
# All ports:
for i in $(seq 1 65535); do nc -w 1 <IP> $i && echo "Port $i is open"; done
```

### Hosts and ports discovery

If we do not have the ping command, we can do a port scan on a specific subnet. Then, if it detects any port, it is because there is an active host.

```bash
#!/bin/bash
subnet="192.168.30"
top10=(20 21 22 23 25 80 110 139 443 445 3389)
for host in {1..255}; do
    for port in "${top10[@]}"; do
        (echo > /dev/tcp/"${subnet}.${host}/${port}") > /dev/null 2>&1 && echo "Host ${subnet}.${host} has ${port} open" || echo "Host ${subnet}.${host} has ${port} closed"
    done
done
```
{% endtab %}

{% tab title="Windows" %}
### Port discovery

{% embed url="https://github.com/MuirlandOracle/C-Sharp-Port-Scan" %}

{% embed url="https://github.com/MuirlandOracle/CPP-Port-Scanner" %}
{% endtab %}
{% endtabs %}

## Using local tools through a proxy

{% content-ref url="chisel.md" %}
[chisel.md](chisel.md)
{% endcontent-ref %}

{% content-ref url="sshuttle.md" %}
[sshuttle.md](sshuttle.md)
{% endcontent-ref %}
