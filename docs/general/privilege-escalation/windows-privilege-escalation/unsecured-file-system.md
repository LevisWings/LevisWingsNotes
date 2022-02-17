# Unsecured File System

## Theory

Sometimes, we may have write access to a folder that is heavily used by users. If in that folder there is, for example, a binary, we can replace it with our own binary with a malicious payload. Another case may be if we can modify/overwrite a configuration file to achieve a certain action.

## Practice

```
accesschk.exe -accepteula -wus "Users" c:\*.* > C:\Windows\Temp\rw-folder-users.txt
accesschk.exe -accepteula -wus "Authenticated Users" c:\*.* > C:\Windows\Temp\rw-folder-authusers.txt
```

### Case #1

If we find a folder with write permissions and inside that folder there is a binary widely used by users (or at least, we suspect that they are using it), we can replace it with our own binary with a malicious payload.

In this case, we can use the "**Backdooring PE**" technique:

{% embed url="https://www.ired.team/offensive-security/code-injection-process-injection/backdooring-portable-executables-pe-with-shellcode" %}
