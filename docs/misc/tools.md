# Tools

## General

### tcpdump

```bash
wget https://github.com/ernw/static-toolbox/releases/download/tcpdump-v4.9.3/tcpdump-4.9.3-x86_64
```

### [busybox](../general/proxies-and-pivoting/busybox.md)

{% embed url="https://busybox.net/downloads/binaries" %}

## Pivoting

### [chisel](../general/proxies-and-pivoting/chisel.md)

{% tabs %}
{% tab title="Compilation" %}
```bash
git clone https://github.com/jpillora/chisel
cd chisel
go build . # Compile with our architecture
```
{% endtab %}

{% tab title="Download (recommended)" %}
```bash
# x64 architectures:
wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_windows_amd64.gz
wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
# x86 architectures:
wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_windows_386.gz
wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_386.gz
# To decompress them:
gunzip <CHISEL>.gz
# Rename files according to the operating system:
mv chisel_1.7.6_windows_amd64.gz chisel.exe # Windows
mv chisel_1.7.6_linux_amd64.gz chisel # Linux
```
{% endtab %}
{% endtabs %}

### [socat](../general/proxies-and-pivoting/socat.md)

{% tabs %}
{% tab title="Linux" %}
```bash
wget https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat # Old version
wget https://github.com/ernw/static-toolbox/releases/download/socat-v1.7.4.1/socat-1.7.4.1-x86_64 # Updated version
```
{% endtab %}

{% tab title="Windows" %}
{% embed url="https://sourceforge.net/projects/unix-utils/files/socat" %}
{% endtab %}
{% endtabs %}

### [sshuttle](../general/proxies-and-pivoting/sshuttle.md)

```bash
apt-get install sshuttle # Ubuntu/Debian
pacman -S sshuttle # Arch Linux
```

For other Linux distributions, you can check [here](https://github.com/sshuttle/sshuttle#obtaining-sshuttle).

### [plink.exe](../general/proxies-and-pivoting/plink.exe.md)

{% embed url="https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html" %}

### nc.exe

```bash
cp /usr/share/windows-resources/binaries/nc.exe . # 32 bits
wget https://github.com/int0x33/nc.exe/raw/master/nc64.exe # 64 bits
```

## Host-Port Scanner

### nmap

```bash
wget https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/nmap # Old version
wget https://github.com/ernw/static-toolbox/releases/download/nmap-v7.91SVN/nmap-7.91SVN-x86_64-portable.zip # Updated version
```

### [kotlin-port-scanner](../general/proxies-and-pivoting/kotlin-port-scanner.md)

```bash
git clone https://github.com/Hydragyrum/kotlin-port-scanner
cd kotlin-port-scanner
chmod +x gradlew
./gradlew shadowJar
cp build/libs/kotlin-port-scanner-1.0-SNAPSHOT-all.jar .
```

## HTTP Client

### curl

{% embed url="https://github.com/moparisthebest/static-curl" %}

### wget

```bash
wget https://github.com/yunchih/static-binaries/raw/master/wget
```

### [kotlin-http-client](../general/proxies-and-pivoting/kotlin-http-client.md)

```bash
git clone https://github.com/Hydragyrum/kotlin-http-client
cd kotlin-http-client
chmod +x gradlew
./gradlew shadowJar
cp build/libs/kotlin-http-client-1.0-SNAPSHOT-all.jar .
mv kotlin-http-client-1.0-SNAPSHOT-all.jar http-client.jar
```
