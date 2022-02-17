# SSH (PTH)

The SSH protocol (also known as Secure Shell) is a method for secure remote login from one computer to another.

SSH is the standard way to log in on UNIX and Linux systems. All methods of execution for Windows are not available. Typically, the only real way to remotely access these systems is through this protocol. In larger organizations, I have often found that Linux systems are tied to Active Directory. If you have a loss of domain groups, look them up to see if there are any with unix/linux in the name. This is a good sign that UNIX/Linux systems may be joined to the domain.

You can't pass the hash to SSH, but you can connect with a Kerberos ticket ([that you can get it in multiple ways](./)). First, try to connect using SSH and enable verbose messages.

```bash
ssh -o GSSAPIAuthentication=yes <USERNAME>@<DOMAIN> -vv

debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic,password
debug1: Next authentication method: gssapi-with-mic
debug1: Unspecified GSS failure.  Minor code may provide more information
No Kerberos credentials available (default cache: FILE:/tmp/krb5cc_1045)
```

You will probably see that it is waiting for the **krb5** ticket somewhere else (depends on your UID) so move your ticket there instead.

```bash
cp user.ccache /tmp/krb5cc_1045
```

Once you have your ticket in the right place, try logging in again. If it doesn't work, consider also using `-l` to specify the username.

```bash
ssh -o GSSAPIAuthentication=yes <USERNAME>@<DOMAIN>
```
