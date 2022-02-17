# Processes

## Practice

Look at the processes that are running and check if there are any weird (or out of the ordinary) processes or have more privileges than they should.

```bash
# View all processes:
ps -ef
# View all processes (with the exception of "session leaders" and processes without ttys control) (recommended):
ps aux
# Search for the process by the name of the command used:
ps -eC <command> # Example: ps -eC atril, ps -eC firefox
# View more information about a process:
ps -fawwx | grep "<PROCESS>"
ps axjf
# View internal processes:
cat /proc/sched_debug
```

Some common privilege escalation scenarios are as follows:

* Web server with root privileges (i.e., if we manage to breach the web server and manage to execute commands, we will do so in the context of the root user).
* Credentials when initializing a process.
* Processes with vulnerable software.
* Process binaries with insecure permissions.
* And other interesting services: For example, if the MySQL service is running as root, we can do the following attack: Rogue SQL Server (INSERT LINK TO: Rogue MySQL).
* Always check the possible electron/cef/chromium debuggers that are running, you could abuse them to escalate privileges. Linpeas detects those by checking the `--inspect` parameter inside the process command line.

### Process monitoring

You can use tools such as procmon.sh or [pspy](https://github.com/DominicBreuker/pspy) to monitor processes. This can be very useful to identify vulnerable processes that run frequently or when a set of requirements are met.

{% code title="procmon.sh" %}
```bash
#!/bin/bash

old_process=$(ps -eo command)

while true; do
	new_process=$(ps -eo command)
	diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -v "procmon.sh" | grep -v "command"
	old_process=$new_process
done
```
{% endcode %}

{% hint style="danger" %}
It may happen that on a 64-bit machine, when running the pspy64 binary, not all processes are detected. However, if we try pspy32, it may appear. I dont' know the reason.
{% endhint %}

### Process management

```bash
# Identify the PIDs of a program or application.
pidof <PROGRAM> # Example: pidof firefox
# Kill a process:
kill 9 <PID>
pkill <PROGRAM>
# Put a process in the background:
firefox &
# If we execute a command, we can do:
Ctrl + Z # Then we can go back with the command: bg
# If we have multiple jobs in the background (either with & or Ctrl + Z), we can see with the command:
jobs
fg %1 # Put the first job in the foreground. If we wanted the second job, it would be %2 and so ....
fg %<PROGRAM> # Return to the job indicating the name of the program.
fg % # Return to the previous job.
kill % # Kill the last job or process.
```
