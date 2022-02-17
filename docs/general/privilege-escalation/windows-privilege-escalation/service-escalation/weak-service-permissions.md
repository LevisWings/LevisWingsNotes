# Weak Service Permissions

## Theory

The Windows registry stores entries for each service. Since registry entries can have ACLs, if the ACL is misconfigured, it may be possible to modify the configuration of a service even if we cannot modify the service directly.

## Verification

### Step #1 - Detect possible misconfigured registry

#### Automatic verification

```bash
# Look at the part of "Looking if you can modify any service registry":
winPEAS.exe quiet servicesinfo
```

![](../../../../.gitbook/assets/weak\_service\_permissions\_winpeas.png)

#### Manual verification

{% tabs %}
{% tab title="accesschk.exe" %}
```bash
accesschk.exe /accepteula "<USERNAME/GROUP>" -kvuqsw hklm\System\CurrentControlSet\services
accesschk.exe /accepteula -kvuqsw hklm\System\CurrentControlSet\services # All users.
accesschk.exe /accepteula -uvwqk <PATH>\<SERVICE_NAME>
# It has to tell us that our user/group has "KEY_ALL_ACCESS" or read/write (RW).
# If so, we can move on to the next step.
```
{% endtab %}

{% tab title="PowerShell" %}
```powershell
Get-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\* > services.txt
foreach($service in Get-Content .\services.txt | % { $_.Split('\')[-1] }){Get-Acl -Path HKLM:\System\CurrentControlSet\Services\$service | fl}
# It has to tell us that our user/group has "KEY_ALL_ACCESS" or read/write (RW).
# If so, we can move on to the next step.
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Remember all the groups we are in. For example, **NT AUTHORITY\INTERACTIVE** is a pseudo-group composed of all the users that can log into the system locally, including our user.
{% endhint %}

### Step #2 - Does it run as SYSTEM?

```powershell
sc qc <SERVICE_NAME> # It must have the START_NAME in LocalSystem.
reg query <PATH>\<SERVICE_NAME> # The ObjectName must be LocalSystem.
$possible_escalation = $services | Where-Object { ($_ObjectName -eq 'LocalSystem') -and ($_Start -eq '3') }
```

### Step #3 - Check if we can start or stop the service

{% tabs %}
{% tab title="sdshow" %}
```shell
# View permissions on a service:
whoami /user # We take as reference our SID.
# We can also take the AU (Authenticated Users) as a reference.
sc sdshow <SERVICE_NAME> 
# References:
- https://www.winhelponline.com/blog/view-edit-service-permissions-windows/ (ACE flags string)
- https://docs.microsoft.com/en-us/windows/win32/secauthz/sid-strings (SID Strings)
# Common ACE flags string:
- RP = SERVICE_START
- WP = SERVICE_STOP
- DC = SERVICE_CHANGE_CONFIG
```
{% endtab %}

{% tab title="accesschk" %}
```shell
accesschk.exe /accepteula -ucqv <USERNAME> <SERVICE_NAME>
```
{% endtab %}
{% endtabs %}

