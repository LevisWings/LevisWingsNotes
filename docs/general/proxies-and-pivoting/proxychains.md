# proxychains

Proxychains is a tool that allows network traffic to pass through one or more SOCKS proxies. This is useful when a tool cannot pass through a proxy natively.

Setting up proxychains is simple, just edit the `/etc/proxychains.conf` file and add the list of SOCKS proxy(s) through which you want to connect.

### SOCKS proxy

```bash
socks4 127.0.0.1 <PORT SOCKS Proxy>
socks5 127.0.0.1 <PORT SOCKS Proxy> # Recommended
```

Now for all commands, you have to add at the beginning, the command `proxychains` :

```bash
proxychains <COMMAND>
proxychains -q <COMMAND>
```

### HTTP proxy

If we want to use an HTTP proxy, (such as Squid HTTP Proxy) it would be as follows:

```bash
http <PROXY IP> <PROXY PORT>
```
