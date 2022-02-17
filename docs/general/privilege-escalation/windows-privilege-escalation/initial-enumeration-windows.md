# Initial Enumeration (Windows)

## Network Enumeration

```shell
# List all network interfaces, IP, and DNS:
ipconfig # ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
# List current routing table (tabla de enrutamiento):
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
# List the ARP table:
arp -a #Muestra la Dirección de Internet (IPs), la Dirección Física (MACs) y el tipo (Estático o Dinámico).
# List all network shares:
net share
# List DNS:
ipconfig /displaydns
# SNMP Configuration:
reg query HKLM\SYSTEM\CurrentControlSet\Services\SNMP /s
Get-ChildItem -path HKLM:\SYSTEM\CurrentControlSet\Services\SNMP -Recurse
```

List all current connections:

```shell
# With netstat:
netstat -n # It shows everything numerically and not the names.
netstat -a # Displays all connections and listening ports by TCP and UDP, with IPv4 and IPv6 addresses.
netstat -b # Show programs. It requires privileges and only works on Windows.
netstat -nato # Windows ("-o" shows the process ID).
# With PowerShell:
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,LinkLayerAddress,State
```

## AV, AppLocker and Firewall Enumeration

```shell
# Check the Windows Defender service, whether it is activated or not, etc.:
sc query windefend
Get-MpComputerStatus
```

```shell
# AppLocker rules list:
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
$a = Get-ApplockerPolicy -effective; $a.rulecollections
# Test AppLocker policy:
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```

```shell
# List firewall state and current configuration:
netsh advfirewall show currentprofile # List firewall current profile in Windows.
netsh advfirewall firewall show rule name=all # (Lots of output) List firewall rules.
netsh advfirewall firewall dump # All.
netsh firewall show state # Firewall status.
netsh firewall show config # Firewall configuration.
netsh advfirewall show domain # If enabled in the domain.
netsh advfirewall show private # If enabled for private connections.
netsh advfirewall show public # If enabled for public connections.
# List firewall's blocked ports:
$f=New-object -comObject HNetCfg.FwPolicy2;$f.rules | where {$_.action -eq "0"} | select name,applicationname,localports
```

## User Enumeration

```shell
# Whoami, privs, groups:
whoami
echo %username% # whoami alternative (1)
$env:UserName # whoami alternative (2)
whoami /priv # View the privileges we have available. If the privilege appears in the list, that user has it, even if it appears as disabled.
whoami /groups # See which groups we are in
tree /F C:\Users\<USERNAME> # Ver las carpetas y archivos en ese directorio
```

```shell
# Which users are in the system? Any old user profiles that have not been cleaned up?
net users # Get all users. Also works with: net user
dir /b /ad "C:\Users\"
dir /b /ad "C:\Documents and Settings\" # Windows XP and below
Get-LocalUser | ft Name,Enabled,LastLogon,Description
Get-ChildItem C:\Users -Force | select Name
qwinsta # Is anyone else logged in?
query user # Same as qwinsta
net user <USERNAME> # To see more information about that user.

# Groups:
net localgroup # Get All Groups (A veces no funciona).
Get-LocalGroup | ft Name # Get All Groups
net localgroup <LOCALGROUP> # Details About a Group
Get-LocalGroupMember <LOCALGROUP> | ft Name, PrincipalSource, ObjectClass

# Misc:
net accounts # Get Password Policy & Other Account Information

# History PowerShell
(Get-PSReadlineOption).HistorySavePath # Get the Powershell history path of the current users.
Get-ChildItem C:\Users\*\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt # Check the Powershell history of all users
```

{% hint style="success" %}
By running `net user` or `net users`, and `whoami`, we can confirm if we are a local user or a service account. If `whoami` says something like "iss/application" and the `net user` result does NOT show our user, then we know that we are not a user, but a service account.
{% endhint %}

## System Enumeration

### General, system architecture

```shell
# General:
dir <PATH> /a # View hidden files and directories of the PATH we specify.
systeminfo # System information.
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
Get-WmiObject -Class Win32_OperatingSystem
Get-WmiObject -Class Win32_OperatingSystem | select Version,BuildNumber
hostname
echo %windir%
# System architecture:
echo %PROCESSOR_ARCHITECTURE%
wmic os get osarchitecture
```

### PATH

See if the PATH changes, if there is a custom path before `C:\Windows\System32`, if you can write to that directory, etc.

```shell
# Environment variables:
set
Get-ChildItem Env: | ft Key,Value
```

Remember, when running a program, Windows looks for that program in the CWD (Current Working Directory) first, and then from the PATH going from left to right. This means that if the custom path is placed on the left (before `C:\Windows\System32`), it is much more dangerous than on the right. If this scenario occurs, we can try [PATH Hijacking](path-hijacking-windows.md).

Also check if the HOMEDRIVE is a shared resource.

### Connected drives (as logical disks)

```shell
net use
wmic logicaldisk # Enumerate logical disks.
wmic logicaldisk get caption,description,providername # Filter disks by caption (C/D/E/F,ETC), description and vendor. Much nicer.
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
powershell -c "Get-PSDrive | where {$_.Provider -like \"Microsoft.PowerShell.Core\FileSystem\"}| ft Name,Root" # CMD
```

### Unmounted disks

```shell
# Enumeration of unmounted disks:
mountvol
```

### Others/Advanced

```shell
# This can be useful at times, as some third-party drivers, even from reputable companies, contain more holes than a Swiss cheese. This is only possible because exploiting ring 0 is outside of most people's experience:
DRIVERQUERY
driverquery /v /fo cvs | ConvertFrom-CSV | Select-Object 'Display Name', 'Start Mode', Path
Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"}
```

## HosFix(s)

If we do not see the HosFix(s) with the `systeminfo` command, we can go to the directory: `C:\Windows\SoftwareDistribution\Download`. There you will find only the wsus updates, but not when the patches were installed.

The file `C:\Windows\WindowsUpdate.log` contains logs of when the patches were installed.

If the hotfixes are still not displayed, we can query them with WMI using the WMI-Command binary with QFE (Quick Fix Engineering) to display the patches:

```shell
wmic qfe # Displays the implemented system patches.
wmic qfe get Caption, Description, HotFixID, InstalledOn
Get-HotFix | ft -AutoSize
```

## Software, Processes, Services and Tasks

### Software/Programs

```shell
# What software is installed:
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
dir /a "C:\ProgramData"
reg query HKEY_LOCAL_MACHINE\SOFTWARE

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)', 'C:\ProgramData' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name

# WMI can also be used to display installed software.
wmic product get name
wmic product get name, version, vendor
Get-WmiObject -Class Win32_Product | select Name, Version
```

{% hint style="info" %}
Is FileZilla/Putty/etc installed? Run LaZagne to check if stored credentials for those applications are installed. Also, some programs may be installed and running as a service that is vulnerable.
{% endhint %}

### Services

What are the processes/services running on the system, are there any internal services not exposed? If so, can we open it?

```shell
net start
tasklist /svc
sc query
sc queryex type=service state=all
reg query HKLM\SYSTEM\CurrentControlSet\Services
Get-Service
wmic service list
wmic service get name,pathname,displayname,startmode
```

### Process

One of the best places to look for privilege escalation are the processes running on the system. Even if a process is not running as an administrator, it can still result in additional privileges. The most common example is discovering a web server such as IIS or XAMPP running on the box, placing an aspx/php shell on the box, and getting a shell as the user running the web server. Generally, this is not an administrator but will often have the SeImpersonate token, allowing Rogue/Juicy/Lonely Potato to provide SYSTEM permissions.

#### Get processes

```powershell
wmic process get ProcessId,Description,ParentProcessId,CommandLine # Recommended.
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
tasklist
tasklist /v # More detailed.
```

The following line returns the owner of the process without administrator rights, if something is blank under the owner it is probably running as SYSTEM, NETWORK SERVICE or LOCAL SERVICE:

```shell
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize
powershell -c "Get-WmiObject -Query 'Select * from Win32_Process' | where {$_.Name -notlike 'svchost*'} | Select Name, Handle, @{Label='Owner';Expression={$_.GetOwner().User}} | ft -AutoSize"
Get-WmiObject win32_service | Select-Object Name,State, PathName | Where-Object {$_.State -like 'Running'}
```

#### Kill process

```shell
# By ID/PID
taskkill /F /PID <ID/PID>
```

### Scheduled tasks

What scheduled tasks are there, has anything custom been implemented?

```shell
schtasks /query /fo LIST 2>nul | findstr TaskName
schtasks /query /fo LIST /TN "<TASKNAME_PATH>" /v
dir C:\windows\tasks

Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State
Get-ScheduledTask | select TaskName,State
```

By default, we can only see the tasks created by our user and the default scheduled tasks that every Windows operating system has. Unfortunately, we cannot see the scheduled tasks created by other users (such as administrators) because they are stored in `C:\Windows\System32\Tasks`, to which standard users do not have read access. It is not uncommon for system administrators to go against security practices and perform actions such as providing read or write access to a folder normally reserved for administrators only. We may (albeit rarely) encounter a scheduled task running as an administrator configured with weak file/folder permissions for any number of reasons.

Translated with www.DeepL.com/Translator (free version)

## Permissions

### Enum permissions

```shell
# Full permissions for the "Everyone" or "Users" group in the folders?:
icacls "C:\Program Files\*" 2>nul | findstr "(F)" | findstr "<GROUP NAME>" # Replace by a group (Everyone, BUILTIN\Users)
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(F)" | findstr "<GROUP NAME>" # Replace by a group (Everyone, BUILTIN\Users)

# Modify permissions for all or users on folders?
icacls "C:\Program Files\*" 2>nul | findstr "(M)" | findstr "<GROUP NAME>" # Replace by a group (Everyone, BUILTIN\Users)
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(M)" | findstr "<GROUP NAME>" # Replace by a group (Everyone, BUILTIN\Users)

Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match '<GROUP NAME>'} } catch {}} # Replace by a group (Everyone, BUILTIN\Users)
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\sModify"}

# You can also load accesschk from Sysinternals to check for writable folders and files:
accesschk.exe -qwsu "Everyone" *
accesschk.exe -qwsu "Authenticated Users" *
accesschk.exe -qwsu "Users" *
```

### Modify permissions

```shell
# Assign "Full Control" (F) permission to a file:
icacls <FILE> /GRANT <USERNAME>:F # If nothing happens, type "Y" and then Enter.
cacls <FILE> /GRANT <USERNAME>:F
```

## .NET Framework version

```shell
# CMD:
reg query "HKLM\SOFTWARE\Microsoft\Net Framework Setup\NDP" /s # Lists all .NET versions
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP" # Also lists all .NET versions
reg query "HKLM\SOFTWARE\Microsoft\Net Framework Setup\NDP\v<NUM>" /s # In <NUM>, we put the version (1,2,3 or 4) to know the exact version.
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v<NUM>\full" /v version # Same as above.
cd %systemroot%\Microsoft.NET\Framework # To search for it manually.
dir /b /ad /o-n %systemroot%\Microsoft.NET\Framework\v?.*
# PowerShell:
Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP' -Recurse | Get-ItemProperty -Name version -EA 0 | Where { $_.PSChildName -Match '^(?!S)\p{L}'} | Select PSChildName, version
```
