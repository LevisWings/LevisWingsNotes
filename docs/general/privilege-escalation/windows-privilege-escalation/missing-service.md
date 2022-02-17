# Missing Service

## Theory

In very rare cases, there may be remnants of a software installation (for example), and a service that was running at startup may not have been removed. If this service runs a binary (which does not exist) in a folder where we have write permissions, we can escalate privileges.

## Verification

```bash
autorunsc64.exe -a s > services.txt
type services.txt | findstr /i "File Not Found"
```

If we find a match, we consult the service configuration to observe its **BINARY\_PATH\_NAME**:

```shell
sc qc <SERVICE_NAME>
```

Copy the BINARY\_PATH\_NAME and check if this file exists to confirm:

```shell
dir <BINARY_PATH_NAME>
```

And if it does not exist, the last thing we have to check is if we have write permissions in the folder where the binary is located:

```shell
icacls <BINARY_PATH_FOLDER>
```

## Privilege Escalation

We simply have to upload our malicious binary to the compromised system in the path of the **BINARY\_PATH\_NAME**. Then we have to wait for the system to reboot.
