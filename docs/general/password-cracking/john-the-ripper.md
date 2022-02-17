# John The Ripper

John The Ripper is a free password cracking software tool.

## Basic use

```bash
john --wordlist=<PATH TO WORDLIST> <HASH FILE>
john <HASH>
john --format=<FORMAT> --wordlist=<PATH TO WORDLIST> <HASH FILE> # Specific format:
john --list=formats # List all formats
john --show <HASH FILE> # To view the previously decrypted hash. Sometimes, it is necessary to clarify the format.
john --format=<FORMAT> --show <HASH FILE>
```

### Example with yescrypt hash

To decrypt the **yescrypt hash**, we have to specify the **crypt** format to the `john` **** command:

```bash
john --format=crypt --wordlist=rockyou.txt hash.txt
```

## Single Crack Mode

From a name, such as Markus, you can generate different combinations:

* Markus1, Markus2, Markus$, MARkus, etc.
* `john --single --format=<FORMAT> <HASH FILE>`

If we have a hash like this: AMUUSKJ43NB, single mode will not work since John takes GECOS fields (fields that are separated by a colon or ":"). But we could add a possible password or directly the username and try if we have any luck. We convert that hash to:

* `<USERNAME>:AMUUSKJ43NB`
* Then we would run the single mode.

## 2john to decrypt files

### Decrypt file passwords

* We can decrypt passwords of files such as ZIP, PDF, KEEPASS, etc. You simply need to extract the hash in John format, which can be done with 2john tools.
* To search all 2john tools, we do: `locate 2john`.

### Dependencies of some 2john

```bash
# For 7z2john:
perl -MCPAN -e "install Compress::Raw::Lzma"
```

## Rules

### Create rules

* Open the `/etc/john/john.conf` file:

```bash
sudo nvim /etc/john/john.conf
sudo nvim /usr/share/john/john.conf
```

![Add rulers in the part of the cursor shown in the image](../../.gitbook/assets/john\_rule.png)

### Use rules

```bash
john --format=raw-sha1 --wordlist=<DICTIONARY> hash.txt --rules=ALL # Use rules in real time
john --wordlist=<DICTIONARY> --rules=1number1symbol -stdout > <OUTPUT>.txt # Export wordlist
```

### Examples of rules

```bash
# Palabra → 1 número → 1 símbolo
[List.Rules:1number1symbol]
*"[0-9][!#$%&=]"
```

### Other rules

```bash
[List.Rules:2nums] # Word -> Number -> Number
$[0-9]$[0-9] 
[List.Rules:Reverse] # Reverse word. ale -> ela
r 
[List.Rules:Repeat] # Repeat word. pepe -> pepepepe
d
dd
ddd
dddd
ddddd
```
