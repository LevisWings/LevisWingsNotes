# RF + SOCKS Proxy

One problem with Remote and Local Forwarding, is that the tunnel is open for a port(s) you specify, which slows down the progress to type the command if you need multiple ports. To remedy this, and to enable dynamic port and target mapping in the same way as with SSH Dynamic Forwarding, the trick is to bind a proxy server to the pivot instead of the target.

To do this, you will need to deploy a proxy server on the pivot.

Of course, there is the well-known `proxychains` tool or its next-generation variant, which may be useful as a client but will be difficult to use as a proxy server on the pivot due to compilation issues. That is why we are going to focus on `3proxy`.

* `proxychains` only works with dynamically linked programs and with the same version used for proxychains
* `proxychains-ng` has the same limitation.
* 3proxy\` advanced proxy, can be deployed as a portable (system library agnostic) version.

### Method with 3proxy:

* We will compile the binary to be deployed on the pivot from the auditor's machine and distribute it via an HTTP server:

```bash
git clone https://github.com/z3APA3A/3proxy.git
cd 3proxy
make -f Makefile.Linux
```

{% hint style="info" %}
Note: If the pivot is a Windows, there is also a Windows makefile or precompiled binaries. The same for other more exotic operating systems.
{% endhint %}

* Here we will use one of the standalone binaries (socks) provided by 3proxy instead of the full 3proxy binary which is more powerful but requires a configuration file. We transfer the `socks` file, which is located in the bin directory, to the victim machine: File Transfer
* Launch the proxy server on port 10080:

```bash
# Pivot machine (victim):
./socks '-?' # Help panel.
./socks -p10080 -tstop -d
```

* Finally, we launch the SSH reverse remote forwarding:

```bash
ssh sshpivot@<LHOST> -R <LPORT>:127.0.0.1:10080 -N # Linux 
plink.exe sshpivot@<LHOST> -R <LPORT>:127.0.0.1:10080 # Windows with plink.exe
```
