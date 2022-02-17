# Metasploit

```bash
msfdb run # Start the database.
msfconsole 
# Search modules:
search <NAME>
# If we want to use a module, there are 2 ways to do it:
use <NUMBER> # Select by number.
use <PATH> # Select by path.
# View available options of an exploit:
show options
# Set options:
set RHOSTS <IP>
set RPORT <PORT>
# Select a target, which can be different OS or different architectures:
show targets
set target <NUMBER>
# Set the payload:
show payloads
set payload <PAYLOAD>
# Set the payload options:
set LHOST <LHOST>
set LPORT <LPORT>
# Execute/launch the exploit:
run
exploit
```

