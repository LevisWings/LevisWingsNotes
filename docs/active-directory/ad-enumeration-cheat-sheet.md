# ⚒ AD Enumeration (Cheat Sheet)

This section will show different commands to enumerate an Active Directory with multiple tools (or simply using CMD or PowerShel commandsl). The tools to use are:

* INSERT LINKS

{% hint style="warning" %}
To avoid storing your own commands in the Powershell history: `Set-PSReadlineOption -HistorySaveStyle SaveNothing`
{% endhint %}

{% hint style="info" %}
By default, **PowerView** makes calls to the Win32 API, but we can also specify another method, the WinNT: `-Method WinNT`
{% endhint %}

### Identify current user/computer domain

{% tabs %}
{% tab title="PowerShell" %}
```powershell
# Identify current user domain:
$env:USERDNSDOMAIN 
(Get-ADDomain).DNSRoot

# Identify current computer domain:
(Get-WmiObject Win32_ComputerSystem).Domain
```
{% endtab %}
{% endtabs %}

### Get Domain SID

{% tabs %}
{% tab title="PowerView & ADModule (PS)" %}
```
Get-DomainSID
```
{% endtab %}

{% tab title="PowerShell" %}
```powershell
(Get-ADDomain).DomainSID
```
{% endtab %}
{% endtabs %}

### Get Forest information

{% tabs %}
{% tab title="PowerShell" %}
```powershell
Get-ADForest
```
{% endtab %}
{% endtabs %}

### Enum Domain Trusts

{% tabs %}
{% tab title="CMD" %}
```shell
nltest /domain_trusts
```
{% endtab %}

{% tab title="PowerView (PS)" %}
```powershell
Get-DomainTrust
Get-DomainTrust -Domain <DOMAIN_NAME>
Get-DomainTrustMapping #Enumerate all trusts for the current domain and then enumerates all trusts for each domain it finds
```
{% endtab %}

{% tab title="ADModule (PS)" %}
```powershell
Get-ADTrust -Filter *
Get-ADTrust -Identity <DOMAIN_NAME>
```
{% endtab %}
{% endtabs %}

### Enum Domain Users/Computers

{% tabs %}
{% tab title="Linux" %}
```bash
# rpcclient:
enumdomusers

# ldapsearch:
ldapsearch -H ldap://<DOMAIN> -x -w '<PASSWORD>' -D "<USERNAME>@<DOMAIN>" -b "dc=<DOMAIN_PART_1>,dc=<DOMAIN_PART_2>" "(objectClass=user)" samaccountname useraccountcontrol
```
{% endtab %}

{% tab title="CMD" %}
```
net user /domain
```
{% endtab %}

{% tab title="PowerView (PS)" %}
```powershell
# Save all Domain Users to a file:
Get-DomainUser | Out-File -FilePath .\DomainUsers.txt
# Will return specific properties of a specific user:
Get-DomainUser -Identity <USERNAME> -Properties *
Get-DomainUser -Identity <USERNAME> -Properties DisplayName, MemberOf | Format-List
Get-DomainUser -Identity <USERNAME> -Properties logoncount,useraccountcontrol,samaccountname,description
Get-DomainUser -SPN # List service accounts.
Get-DomainUser -Identity <USERNAME> -SPN -Properties name,serviceprincipalaccount | fl
Get-NetUser
```
{% endtab %}

{% tab title="ADModule (PS)" %}
```powershell
Get-ADUser <USER>
Get-ADUser -Filter * -Identity <USER> -Properties *
Get-ADUser -Filter * | select SamAccountName
Get-ADObject -LDAPFilter "objectClass=User" # All users (including computers)
# Get a spesific "string" on a user's attribute
Get-ADUser -Filter 'Description -like "*wtver*"' -Properties Description | select Name, Description
```
{% endtab %}
{% endtabs %}

If we want to see the properties of a user that can lead to an attack, we can see the following resource:

INSERT LINK TO "User Properties Summary"

### Enumerate logons

{% tabs %}
{% tab title="PowerView (PS)" %}
(Only Administrator) Enumerate user logged on to a machine (i.e., users who are interactively authenticated). Uses the `NetWkstaUserEnum` function of the Windows API:

```powershell
# (Only Administrator) Enumerate user logged on to a machine:
Get-NetLoggedon -ComputerName <COMPUTER_NAME>

# With PsLoggedon64.exe we can also view the logons on a machine.
# This Sysinternals tool uses the Remote Registry, and it works if we only connect interactively:
.\PsLoggedon64.exe \\<COMPUTER>
# More information about PsLoggedon64.exe here: https://docs.microsoft.com/en-us/sysinternals/downloads/psloggedon#introduction
```
{% endtab %}
{% endtabs %}

See the **Interactive Logon**, **Network Logon/Session** sections, and for restrictions, see the section: **Restrictions - Logons/Sessions Enumeration**. These sections can be found in the following resource:

INSERT LINK TO: Logon Types

### Enumerate sessions

{% tabs %}
{% tab title="Linux" %}
```bash
# rpcclient:
debug 10
netsessenum
debug 0 # To set the debug by default.
```
{% endtab %}

{% tab title="PowerView (PS)" %}
Enumerate Sessions (Network Sessions/Network logons) information for a machine. It uses the `NetSessionEnum` function of the Windows API. The **CName** parameter will tell us where you have connected from:

```powershell
# Enumerate Sessions (Network Sessions/Network logons) information for a machine:
Get-NetSession -ComputerName <COMPUTER_NAME>

# Enumerate domain machines of the current/specified domain where specific users are logged into:
Find-DomainUserLocation -Domain <DomainName> | Select-Object UserName, SessionFromName
```
{% endtab %}
{% endtabs %}

### Enum Groups and Group Members

{% tabs %}
{% tab title="Linux" %}
```bash
# rpcclient:
enumalsgroups builtin # Default groups.
queryaliasmem builtin <RID> # Display the SIDs of the members that are part of that group (from the RID).
lookupsids <SID>
```
{% endtab %}

{% tab title="CMD" %}
```shell
net groups /domain
net group "<GROUP>" /domain # Group members.
```
{% endtab %}

{% tab title="PowerView (PS)" %}
```powershell
Get-DomainGroup
Gte-DomainGroup -Domain "<DOMAIN>"
# Return members of Specific Group (eg. Domain Admins & Enterprise Admins):
Get-DomainGroup -Identity '<GROUP_NAME>' | Select-Object -ExpandProperty Member
Get-DomainGroupMember -Identity '<GROUP_NAME>' | Select-Object MemberDistinguishedName
# Enumerate the local groups on the local (or remote) machine. Requires local admin rights on the remote machine:
Get-NetLocalGroup | Select-Object GroupName
Get-NetLocalGroup -Computer <COMPUTER>
#Enumerates members of a specific local group on the local (or remote) machine. Also requires local admin rights on the remote machine:
Get-NetLocalGroupMember -GroupName Administrators | Select-Object MemberName, IsGroup, IsDomain
Get-NetLocalGroupMember -Computer <COMPUTER>
#Return all GPOs in a domain that modify local group memberships through Restricted Groups or Group Policy Preferences:
Get-DomainGPOLocalGroup | Select-Object GPODisplayName, GroupName
```
{% endtab %}

{% tab title="ADModule (PS)" %}
```powershell
Get-ADGroup -Filter * | select SamAccountName
# Return members of Specific Group (eg. Domain Admins & Enterprise Admins):
Get-ADGroup "<GROUP_NAME>" -Properties members,memberof
```
{% endtab %}
{% endtabs %}

If we want to see the groups that can lead to an attack, we can see the following resource:

LINK TO: Groups Summary

### Domain Controllers Discovery

{% tabs %}
{% tab title="Linux" %}
```bash
# nslookup (No authentication):
# DNS query asking for the LDAP servers of the domain (which are the DCs):
nslookup -q=srv _ldap._tcp.dc._msdcs.<DOMAIN> 

# nmap:
# We can also know if it is a DC if it has ports 88 (Kerberos), 389 (LDAP), 5985 (WinRM), etc.:
nmap <IP_DC> -Pn -sV -p-
```
{% endtab %}

{% tab title="CMD" %}
```shell
# nltest:
nltest /dclist:<DOMAIN> # Authentication required.
```
{% endtab %}
{% endtabs %}

### Windows Domain Computers Discovery

{% tabs %}
{% tab title="Linux" %}
```bash
# ldapsearch:
# Requires credentials. Query the domain database via LDAP:
ldapsearch -H ldap://<DOMAIN> -x -LLL -W -D "<USERNAME>@<DOMAIN>" -b "dc=<DOMAIN>,dc=<DOMAIN>" "(objectclass=computer)" DNSHostName OperatingSystem userAccountControl SamAccountName

# nbtscan:
nbtscan 192.168.100.0/24 # No credentials. NetBIOS scan.

# ntlm-info:
ntlm-info smb 192.168.100.0/24 # No credentials. Retrieve host information from NTLM.

# Finally, you can also scan other ports such as 135 (RCP) or 139 (NetBIOS session service) with nmap:
```
{% endtab %}
{% endtabs %}

### Windows Domain Computers Enumeration

{% tabs %}
{% tab title="PowerView (PS)" %}
```powershell
Get-DomainComputer -Properties OperatingSystem, Name, DnsHostName | Sort-Object -Property DnsHostName
Get-DomainComputer -Ping -Properties OperatingSystem, Name, DnsHostName | Sort-Object -Property DnsHostName # Enumerate Live machines 
Get-DomainComputer -Properties samaccountname,useraccountcontrol,operatingsystem,description
Get-DomainComputer <COMPUTER> -Properties name,serviceprincipalname,operatingsystem | fl # View the services that the computer has up.
```
{% endtab %}

{% tab title="ADModule (PS)" %}
```powershell
Get-ADComputer -Filter * -Properties *
```
{% endtab %}
{% endtabs %}

### Enum Group Policies

{% tabs %}
{% tab title="PowerView (PS)" %}
GPO properties on a computer:

* `Displayname` = Name of the GPO .
* `Name` = Identifier of the GPO.
* `gpcfilesyspath` = Path of the GPO.

```powershell
Get-DomainGPO
# Enumerate all GPOs to a specific computer:
Get-DomainGPO -ComputerIdentity <COMPUTER_NAME>
Get-DomainGPO -Properties DisplayName displayname,name,gpcfilesyspath
# We can go to the "Enum OUs" section to determine to which objects these GPOs apply...

# Get users that are part of a Machine's local Admin group
Get-DomainGPOComputerLocalGroupMapping -ComputerName <COMPUTER_NAME>
```
{% endtab %}
{% endtabs %}

More information in the following resource:

LINK TO: Group Policy

### Enum OUs

{% tabs %}
{% tab title="PowerView (PS)" %}
```powershell
Get-DomainOU -Properties Name
# Which objects belong to a particular GPO?:
Get-DomainOU -GPLink <NAME GPO> # Data about the OU that has that GPO applied to all its objects.
Get-DomainOU -GPLink <NAME GPO> | % {Get-DomainComputer -SearchBase $_.distinguishedname -Properties samaccountname} # See the GPO computers.
```
{% endtab %}
{% endtabs %}

### Enum ACLs

Remember that the `ObjectAceType` is important if the **ActiveDirectoryRights** (i.e. the right one) is `ExtendedRight`.

{% tabs %}
{% tab title="PowerView (PS)" %}
```powershell
Get-DomainObjectAcl
Get-DomainObjectAcl -Identity <AccountName> # Returns the ACLs associated with the specified account.
Get-DomaiObjectAcl -Identity <AccountName> -ResolveGUIDs
Get-DomainObjectAcl -Identity <AccountName> | select ActiveDirectoryRights,SecurityIdentifier,AceType
Get-DomainObjectAcl -Identity <AccountName> | select SecurityIdentifier,AceType,ActiveDirectoryRights, @{name='Whois';expression= {Convert-SidToName $_.SecurityIdentifier }}, ObjectAceType | fl # To obtain the object which has these permissions, without the need to convert each SID to a name.
Get-DomainObjectAcl -Identity <AccountName> | select SecurityIdentifier,AceType,ActiveDirectoryRights, @{name='Whois';expression= {Convert-SidToName $_.SecurityIdentifier }}, ObjectAceType | Where Whois -eq "<DOMAIN>\<USER>"
# Displays all ACLs where the SecurityIdentifier is greater than 1000:
Invoke-ACLScanner
Get-DomainObjectAcl <AccountName> | ? { ($_.SecurityIdentifier -match '^S-1-5-.*-[1-9]\d{3,}$') } | select SecurityIdentifier,AceType,ActiveDirectoryRights, @{name='Whois';expression= {Convert-SidToName $_.SecurityIdentifier }}, ObjectAceType # Recommended
Get-DomainComputer | Get-DomainObjectAcl | ? { ($_.SecurityIdentifier -match '^S-1-5-.*-[1-9]\d{3,}$') } | select SecurityIdentifier,AceType,ActiveDirectoryRights, @{name='Whois';expression= {Convert-SidToName $_.SecurityIdentifier }}, ObjectAceType # Computer type objects.
## Search for interesting ACEs
Find-InterestingDomainAcl -ResolveGUIDs
## Check the ACLs associated with a specified path (e.g smb share)
Get-PathAcl -Path "\\Path\Of\A\Share"
```
{% endtab %}

{% tab title="ADModule (PS)" %}
```powershell
(Get-ACL "AD:$((Get-ADUser <AccountName>).distinguishedname)").access
(Get-ACL "AD:$((Get-ADUser <AccountName>).distinguishedname)").access | where IdentityReference -eq <DOMAIN>\<USER>
(Get-Acl "AD:$(Get-ADUser <AccountName>)").Owner
```
{% endtab %}
{% endtabs %}

### Enum Domain Shares

{% tabs %}
{% tab title="PowerView (PS)" %}
```
Find-DomainShare
Find-DomainShare -Verbose
```
{% endtab %}
{% endtabs %}

### Misc

{% tabs %}
{% tab title="PowerView (PS)" %}
```powershell
Convert-SidToName <SID> # Convert the SID to a name.
ConvertFrom-UACValue <useraccountcontrol VALUE> # Useful when querying with ldapsearch.
```
{% endtab %}

{% tab title="ADModule (PS)" %}
```powershell
cd AD: # Browse AD objects as if it were a file system.
```
{% endtab %}
{% endtabs %}
