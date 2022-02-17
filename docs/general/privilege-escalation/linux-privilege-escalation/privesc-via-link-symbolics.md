# PrivEsc via Link Symbolics

Sometimes, as another user we can edit a file. However, if we have permissions to modify that file, we could add a symbolic link to point to another file where that user has permissions to read and/or edit it. Example:

```bash
ln -sf <PATH TO READ/MODIFY FILE> <FILE>
ln -sf <PATH TO DIRECTORY> <DIRECTORY>
# Examples:
ln -sf /root/.ssh/id_rsa <FILE>
ln -sf /root/ <DIRECTORY>
```

Remember that this can be concatenated with a backup file as a **.tar** or **.tar.gz** where before its compression, we make a symbolic link to a file/directory and when it is compressed (through a cron job, etc.) as another user, when decompressing it we will see the linked content.
