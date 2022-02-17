# System Enumeration

```bash
pwd # Returns the name of the current working directory.
hostname # Sets or prints the current host name.
hostname -I # View the IP of the machine. In Debian, it is: hostname -i
uname -a # To see the kernel and architecture.
echo $PATH # Actual PATH
env # Interesting information, passwords or API keys in the environment variables?
lsb_release -a # Displays information about the GNU/Linux distribution of the computer. It can also be viewed with: cat /etc/os-release or cat /etc/lsb-release.
cat /proc/version
cat /etc/issue # To see the distribution.
df -h # List all partitions.
strings <USB MOUNT> # If there is a USB where there was content before, we can visualize a history with strings.
fdisk -l # (root req) List the partition tables for the specified devices and then exit.
lscpu # 
apt list 2>/dev/null # grep "^sudo" : Ver la versión de sudo y buscar posibles vulnerabilidades. Escalation via Sudo Shell Escaping
lsof # Enumera los archivos abiertos. lsof -u <USER> para ver los archivos abiertos de un determinado usuario. lsof -i para ver las conexiones disponibles. lsof -i :<PORT NUMER> para ver si está ocupado ese puerto.
lspci # Enumera los dispositivos PCI.
lsattr <FILE>
systemctl -list-timers
dpkg -l # Para los sistemas Debian.
which <COMMAND>
command -v <COMMAND>
```

### Un/Mounted File Systems

We enumerate mounted/unmounted file systems with:

```bash
mount -l
cat /etc/fstab # Not all drives are listed here.
lsblk # Display devices, drives, partitions and their capacities (whether the drives are mounted or not).
lsusb # Display a list of USB devices.

# Mount:
mkdir /mnt/host
mount /dev/sda[1,2,3] /mnt/host # Mount sda1 (2,3,etc.) disk.
# Others disks: dm-0
```
