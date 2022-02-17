# sshuttle

**sshuttle** works as a transparent proxy through SSH. Easy to use.

### [Download](../../misc/tools.md#sshuttle)

### Use

{% tabs %}
{% tab title="Manual" %}
```bash
# If we are going to tunnel traffic on another subnet:
sshuttle -r <USER>@<IP> <SUBNET>
sshuttle -r <USER>@<IP> <SUBNET> <SUBNET>
# If we are going to tunnel traffic on the same subnet:
sshuttle -r <USER>@<IP> <SUBNET> -x <ACTUAL HOST>
# If we are going to tunnel traffic on the same subnet and other subnets:
sshuttle -r <USER>@<IP> <SUBNET> <ACTUAL SUBNET> -x <ACTUAL HOST>
```

You can also use it with an SSH private key:

```bash
sshuttle -r root@<IP> --ssh-cmd "ssh -i id_rsa" <SUBNET> -x <ACTUAL SUBNET/HOST>
```
{% endtab %}

{% tab title="Automatic" %}
Then, sshtuttle automatically creates the "iptables" rules so that one can directly contact the remote network without manual configuration.

It is also possible to let sshuttle automatically detect the subnets to be transferred based on the server's routing table. At this point, it is quite prudent to enable the first level of verbosity to be able to see the automatically created rules.

```bash
sshuttle -vNr <USER>@<IP> -x 192.168.1.0/24 # 192.168.1.0/24 = To the compromised machine segment.
```

{% hint style="info" %}
The `-x` option is used to exclude a subnet from transmission through the tunnel. If we want it to discover all of them, we simply delete this part: `-x 192.168.1.0/24` (for this example).
{% endhint %}
{% endtab %}
{% endtabs %}

### Troubleshooting

When using sshuttle, you may encounter an error that looks like this:&#x20;

```bash
client: Connected.
client_loop: send disconnect: Broken pipe
client: fatal: server died with error code 255
```

This can occur when the compromised machine you're connecting to is part of the subnet you're attempting to gain access to. For instance, if we were connecting to 172.16.0.5 and trying to forward 172.16.0.0/24, then we would be including the compromised server inside the newly forwarded subnet, thus disrupting the connection and causing the tool to die.

To get around this, we tell sshuttle to exclude the compromised server from the subnet range using the `-x` switch. To use our earlier example:&#x20;

```bash
sshuttle -r user@172.16.0.5 172.16.0.0/24 -x 172.16.0.5
```

This will allow sshuttle to create a connection without disrupting itself.
