# Password Hunting

{% hint style="warning" %}
Remember, **it is not necessary** that a **password** must be next to a **keyword** such as "**pass**", "**key**", etc. Sometimes, it may be that the password is in:

* **Processes** that require a password as a parameter.
* **Log files**, where the user commits a failure when typing a command or his username with his password. This can happen in log files to services like SSH (`auth.log`), FTP (`vsftpd.log`) and also in the `.bash_history` itself.
{% endhint %}

```bash
grep --color=auto -rnw '<PATH>' -ie "PASSWORD" --color=always 2>/dev/null
grep -r -i -E "pass|user|key|secret" 2>/dev/null
grep -r -i -E "pass|user|key|secret" --text # Recursive search (even in binaries)
grep -iRl "password" #  If it finds a match, it will only print the file path.
locate password | more # locate pass | more
find / -name authorized_keys 2>/dev/null
find / -name id_rsa 2>/dev/null
find / -name *config* 2>/dev/null # This command is good to look for in: /var/www/html
find / -type f -name ".log" 2>/dev/null
cat .bash_history
```

