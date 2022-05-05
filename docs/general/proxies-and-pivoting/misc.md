# Misc

## Add static routes (Linux)

{% tabs %}
{% tab title="ip route" %}
### Add routes

```bash
ip route add <network_ip>/<cidr> via <gateway_ip> # Via gateway_ip
ip route add <network_ip>/<cidr> dev <iface> # Via iface
```

### Remove routes

```bash
ip route del <network_ip>/<cidr>
```
{% endtab %}

{% tab title="route" %}
### Add routes

```bash
route add -net <network_ip>/<cidr> gw <gateway_ip>
```
{% endtab %}
{% endtabs %}

## OpenVPN fix MTU

{% embed url="https://www.thegeekpub.com/271035/openvpn-mtu-finding-the-correct-settings" %}
Reference
{% endembed %}

#### Solution #1

The first thing you need to do to fix your OpenVPN MTU problem is to figure out what your largest MTU actually is. You can do this using the ping command. `ping -f` tells ping not to fragment the packet under any circumstances. `ping -l` tells ping the packet size to use.

```bash
ping -l <IP> -l <PACKET_SIZE> # Try: 1500, 1450
```

If we were successful with a 1450 byte packet to get the data across the tunnel, then we can add the following OpenVPN MTU setting (must be added on both client and server):

```
tun-mtu 1450
```

#### Solution #2

A lot of people will suggest setting MSSFIX to 40 bytes smaller and also including it in your statement, as some devices don’t properly learn MTU settings. MSSFIX is a workaround used when MTU path discovery is broken for some reason.

```bash
1450 - 40 = 1410
# OpenVPN configuration:
mssfix 1410
```
