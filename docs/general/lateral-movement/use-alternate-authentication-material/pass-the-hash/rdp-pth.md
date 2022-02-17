# RDP (PTH)

Often, during a penetration test, you may want to access software installed on a user's system that is only available through a graphical user interface (GUI). This may be a password manager that can be easily exported through the GUI, or other software that can perform actions that would be impossible/unwieldy to use otherwise. You may want to pass an NT hash of a user that could not be cracked and take their session, but as we said, by default you cannot.

However, there is a way to perform Pass-The-Hash, and that is that this is only possible when the system has [Restricted Admin mode for RDP](https://docs.microsoft.com/en-us/archive/blogs/kfalde/restricted-admin-mode-for-rdp-in-windows-8-1-2012-r2) enabled, introduced in Windows 8.1/2012 R2. If this is not enabled and you try to do PTH, you will get an error stating that "Account restrictions are preventing this user from logging in." (Account restrictions are preventing this user from logging in.). When Restricted Admin mode is enabled, simple credentials are not sent, so it is possible to perform a Pass-The-Hash/Key/Ticket to establish an RDP connection. Restricted Admin mode is disabled by default. The good news is that, if you have some level of administrator access to the system and access to SMB/WinRM/etc, you can enable this feature remotely.

```bash
crackmapexec smb <IP> -u <USERNAME> -H <HASH> -x 'reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f'
```

Once the registry key is set or you know previously that it is already set, you can [pass the hash with xfreerdp](https://www.kali.org/blog/passing-hash-remote-desktop/) (you need to install the `freerdp2-x11` `freerdp2-shadow-x11` packages instead of `freerdp-x11` as the article said) where you only have to provide the NT has instead of the password:

```bash
# Remember that the "Restricted Admin mode for RDP" must be enabled, a feature added as of Windows 2012 R2 and Windows 8.1:
xfreerdp /u:<USERNAME>@<DOMAIN> /pth:<HASH_NTLM> /v:<IP>
```

On the other hand, from Windows you can [inject an NT hash or Kerberos ticket with mimikatz or Rubeus and then use](https://shellz.club/pass-the-hash-with-rdp-in-2019/) `mstsc.exe /restrictedadmin` to establish an RDP connection without requiring the user's password.
