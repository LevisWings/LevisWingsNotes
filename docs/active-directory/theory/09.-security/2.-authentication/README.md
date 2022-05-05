# 📓 2. Authentication

A critical point to understand many of the Active Directory attacks is to understand how authentication works in Active Directory. But before digging into the technical details, let's make a quick summary.

In Active Directory there are two network authentication protocols available: **NTLM** (INSERT LINK TO NTLM) and **Kerberos** (INSERT LINK TO KERBEROS). Any of them can be used to authenticate domain users, but Kerberos is the preferred option for this case, however only NTLM can be used to authenticate local computer users.

Since any of them can be used, how the client and server agree on the authentication protocol to be used? They use a negotiation mechanism called **SPNEGO** (INSERT LINK TO SPNEGO). With SPNEGO, they can indicate the protocols that are acceptable for them.

![SPNEGO negotiation](../../../../.gitbook/assets/spnego\_negotiation.png)

The protocols negotiated with SPNEGO must be compatible with the [GSS-API](https://zer1t0.gitlab.io/posts/attacking\_ad/#gss-api) programming interface, allowing the client and server programs to use them in a transparent way.

But it also must be taken into account that authentication protocols are not only to being used to remote logons, but also for local logons, since the computers need to authenticate the domain users against the Domain Controllers (usually by asking a Kerberos ticket). There are different [logon types](https://zer1t0.gitlab.io/posts/attacking\_ad/#logon-types) in Windows machines and they should be considered by a pentester, since many of them cache the user credentials in the lsass process or store the passwords in the LSA Secrets.

So let's review these topics.
