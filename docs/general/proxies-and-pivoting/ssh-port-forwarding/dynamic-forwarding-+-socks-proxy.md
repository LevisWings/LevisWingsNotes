# Dynamic Forwarding (+ SOCKS Proxy)

Connections from various services will be transferred through the SSH client, then through the SSH server and finally to various target machines.

Type of Dynamic Forwarding -> Forward Connection

## Method

* With Dynamic Port Forwarding a local port is opened on the pivot machine. SSH will listen on this port and behave as a SOCKS proxy server (SOCKS4 or SOCKS5).
* This will allow the auditor to use the SSH server on the pivot machine as a SOCKS proxy server with local binding.

```bash
# Both are the same, since if 127.0.0.1 is not specified, by default it will be 127.0.0.1.
ssh <USERNAME>@<RHOST o IP> -D 127.0.0.1:<LPORT> -N
ssh <USERNAME>@<RHOST o IP> -D <LPORT> -N
```

* Done. The auditor can then make requests to all networks accessible from the pivot machine through the proxy. This can be done with proxychains or another proxy.

### Proxychains configuration

```bash
socks5 127.0.0.1 <LPORT> # The LPORT is where we set up dynamic forwarding.
```

* Now in all the commands to execute, you must go at the beginning with: `proxychains`.

### Curl with proxy

```bash
curl --head http://<VICTIM IP o PIVOT IP> --proxy socks5://127.0.0.1:<LPORT> # The LPORT is where we set up dynamic forwarding.
```

### BurpSuite configuration

* Configure SOCKS Proxy: [Use SOCKS Proxy](broken-reference)
