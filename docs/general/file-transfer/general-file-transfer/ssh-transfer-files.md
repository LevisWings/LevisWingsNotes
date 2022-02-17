# SSH Transfer Files

```bash
# SCP Download:
scp <USER>@<IP>:"<PATH>/<FILENAME>" .
scp <USER>@<IP>:C:/Window/Temp/test/test.zip .
scp <USER>@<IP>:<PATH>/* .
scp <USER>@<IP>:"<PATH>/<FILE>" C:\Windows\Temp\mimikatz.exe
# SCP Upload:
scp <PATH>/<FILE> <USER>@<IP>:/home/leviswings/<FILE>
```
