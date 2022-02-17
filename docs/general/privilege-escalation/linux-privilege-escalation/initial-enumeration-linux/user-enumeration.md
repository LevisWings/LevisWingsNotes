# User Enumeration

### Practice

```bash
# Essentials
whoami # Displays the current user name.
id # Returns the users identity.
who # Shows who is logged in.
history
cat /etc/passwd
cat /etc/shadow
cat /etc/group # To see the available groups and their users.
cat /etc/group | grep <USERNAME> # To see which groups a user is in. This is really useful if we can't execute commands.
sudo su -
```

```bash
# See the privileges we have to execute with sudo:
sudo -l
cat /etc/sudoers
cat /etc/sudoers.d/<USERNAME>
# View our current UID:GID (useful when we have webshell).  This is useful for CTF if we want to know which user a service is running with and look up its homepath to see its files such as .ssh/id_rsa.
cat /proc/self/status
cat /proc/self/version
```

### find

```bash
find / -user <USERNAME> 2>/dev/null # See what folders or files we have.
find / -group <USER/GROUP> 2>/dev/null # See what folders or files we have.
find / -writable 2>/dev/null # To see what files we have to write as our current user.
```

### Advanced

```bash
getfacl <FILE> # Get file access control lists.
grep -r -i "<USERNAME>" 2>/dev/null # Search recursively for the keyword in each file. It is recommended to do this in a directory with few files, not in the root ("/").
stat <PROGRAM> # See when a program or script was last executed.
date # View the time.
```

<details>

<summary>Common locations for user-installed software:</summary>

* `/usr/local/`
* `/usr/local/src`
* `/usr/local/bin`
* `/opt/`
* `/home`
* `/var/`
* `/usr/src/`

</details>
