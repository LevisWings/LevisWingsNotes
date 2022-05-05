# PrivEsc via Process

Take a look to **what processes** are being executed and check if any process has **more privileges than it should** (maybe a tomcat being executed by root?)

```bash
ps aux
ps -ef
top -n 1
```

Always check for possible [**electron/cef/chromium debuggers** running, you could abuse it to escalate privileges](broken-reference). **Linpeas** detect those by checking the `--inspect` parameter inside the command line of the process.

Also **check your privileges over the processes binaries**, maybe you can overwrite someone.

### Process monitoring

{% hint style="success" %}
It is recommended to leave pspy running in a separate terminal to see if there is any other strange behavior when, for example, running a suspicious binary.
{% endhint %}

{% tabs %}
{% tab title="pspy" %}
You can use tools like [**pspy**](https://github.com/DominicBreuker/pspy) to monitor processes. This can be very useful to identify vulnerable processes being executed frequently or when a set of requirements are met.

{% hint style="danger" %}
Remember to check the system architecture to download the correct version of pspy.
{% endhint %}

### Download binary

```bash
# 64 bits:
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64
# 32 bits:
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy32
```

### Compilation

```bash
git clone https://github.com/DominicBreuker/pspy
go get https://github.com/DominicBreuker/pspy/tree/master/cmd # First time
cd pspy
```

Now, compile:

```bash
GOOS=<OS> GOARCH=<amd64/386> go build .
GOOS=<OS> GOARCH=<amd64 o 86> go build -ldflags "-s -w" .
```

### Throubleshoting

![](../../../.gitbook/assets/pspy\_throubleshoting.png)

```bash
go env -w GO111MODULE=auto
go get <URL MODULE>
# When we finish compiling, we do this to leave it as it was:
go env -w GO111MODULE=off
```
{% endtab %}

{% tab title="procmon.sh" %}
Creation of a Bash script to capture commands executed at the system level:

```bash
#!/bin/bash

old_process=$(ps -eo command)
while true; do
    new_process=$(ps -eo command)
    diff <(echo "$old_process") <(echo "$new_process") | grep "[\<\>]"
    old_process=$new_process
done
```
{% endtab %}
{% endtabs %}
