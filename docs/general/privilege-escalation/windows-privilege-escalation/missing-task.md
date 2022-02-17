# Missing Task

## Theory

On rare occasions, we may find a missing binary for a task. It may occur because administrators, in the process of reconfiguring a Windows system, delete the binary, but do not delete the task.

## Verification

```shell
autorunsc64.exe -a t > tasks.txt
type tasks.txt | findstr /i "File Not Found"
```

If we find a match, we consult the service configuration to observe its **BINARY\_PATH\_NAME**:

```shell
schtasks /query /tn OneDriveChk /xml
```

```xml
<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.0" xmlns="...">
  <RegistrationInfo>
    ...
  </RegistrationInfo>
  <Principals>
    <Principal>
      <UserId>S-1-5-21-3461203602-4096304019-2269080069-1003</UserId>
      <LogonType>Password</LogonType>
      <RunLevel>HighestAvailable</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    ...
  </Settings>
  <Triggers>
    <LogonTrigger />
  </Triggers>
  <Actions Context="Author">
    <Exec>
      <Command>C:\binaries\test.exe</Command>
    </Exec>
  </Actions>
</Task>
```

* **UserId**: The user's SID. It is important that this task is run as another user or, ideally, as an administrator user.
* **LogonType**: In this case, use a password for the task.
* **RunLevel**: The level of integrity with which this task will be executed. In this case, it is high level.
* **Command**: The command to be executed. In this case, since it is a binary, we can check if it still exists or not.
* **Triggers**: Triggers indicates how the task is activated. In this case, we see that it is **LogonTrigger**, which means that it is triggered when the user logs in.

```
dir C:\binaries\test\.exe
```

## Privilege Escalation

We simply have to upload our malicious binary to the compromised system, where the missing task binary is located. Then we have to wait for the system to reboot.
