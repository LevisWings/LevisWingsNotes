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
ssh -L <USER>@<IP> 127.0.0.1:<OPEN_PORT>:<TARGET>:<PORT> -N
ssh -L <USER>@<IP> <OPEN_PORT>:<TARGET>:<RPORT> -N # The localhost or 127.0.0.1 does not need to be specified.
ssh -L <USER>@<IP> <OPEN_PORT>:<TARGET>:<PORT> -N
ssh -L <USER>@<IP> <OPEN_PORT>:<TARGET>:<PORT> -fN # -f = No shell (background)
```
