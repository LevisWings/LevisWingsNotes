# Escape commands

### Use "?"

```bash
/usr/bin/cat /etc/passwd # Permission denied.
/usr/???/??t /etc/passwd
??t /etc/passwd
```

### Use non-existent variables

```bash
cat /etc/passwd # Permission denied.
c${ASAD}a${KDSFH}t /etc/passwd
```

### Escape letters

```bash
cat /etc/passwd # Permission denied.
\c\a\t /etc/passwd
```

### Use environment variables

For example, if when running `env`, we see a HOME variable that has a slash ("/") as its value, then we can use it to escape that character:

```bash
cat ${HOME}root${HOME}root.txt
```

### Use ASCII

If we cannot enter slashes ("/"), we could do the following:

```bash
ascii -o # We print the ASCII table in octal and look for the slash.
slash=$(printf "\<OCTAL>"); cat "$slash"root"$slash"root.txt
```
