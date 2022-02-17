# Local Forwarding

## Theory

Creating a forward (or "local") SSH tunnel can be done from our attacking box when we have SSH access to the target. As such, this technique is much more commonly used against Unix hosts. Linux servers, in particular, commonly have SSH active and open. That said, Microsoft (relatively) recently brought out their own implementation of the OpenSSH server, native to Windows, so this technique may begin to get more popular in this regard if the feature were to gain more traction.

Type of Local Forwarding -> Forward Connection

## Practice

On our computer, we enable the SSH service:

```bash
systemctl start ssh
service ssh start
```

We run the following command on our machine:

```bash
ssh -L <USER>@<IP> 127.0.0.1:<LPORT>:<VICTIM IP or PIVOT IP>:<RPORT> -N
ssh -L <USER>@<IP> <LPORT>:<VICTIM IP or PIVOT IP>:<RPORT> -N # The localhost or 127.0.0.1 does not need to be specified.
ssh -L <USER>@<PIVOT IP> <LPORT>:<IP>:<RPORT> -N # The <PIVOT IP> is the machine that we are going to do a remote forwarding of a certain port. Example: ssh root@172.0.0.3 -N -L 4000:127.0.0.1:4000
# VICTIM IP will be 127.0.0.1, but if we want to see a port/service of another machine that is visible from the compromised victim machine, then we indicate the PIVOT IP.
ssh -L <USER>@<PIVOT IP> <LPORT>:<IP>:<RPORT> -fN # -f = No shell (background)
```
