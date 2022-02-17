# Metasploit

```bash
run autoroute -s <SUBNET IP RANGE>
run autoroute -p # It will provide us with a session.
background
use auxiliary/scanner/portscan/tcp
# We set the RHOST and execute.
run
```
