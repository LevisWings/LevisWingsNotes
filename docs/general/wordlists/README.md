# 📕 Wordlists

## Methodology

### For password cracking and guessing

These wordlists can be used for local password cracking, and also for password guessing on services such as HTTP, FTP, SSH, etc.:

```bash
wget https://raw.githubusercontent.com/LevisWings/personalUtilities/master/fast_wordlist.txt # 42 lines
/opt/SecLists/Passwords/Common-Credentials/top-20-common-SSH-passwords.txt
wget https://raw.githubusercontent.com/drtychai/wordlists/master/fasttrack.txt # 222 lines
/opt/SecLists/Passwords/Leaked-Databases/rockyou.txt
wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt
```

### For fuzzing on a web service

```bash
/opt/SecLists/Discovery/Web-Content/quickhits.txt # 1
/opt/SecLists/Discovery/Web_Content/Top1000-RobotsDisallowed.txt # 2
/opt/SecLists/Discovery/Web-Content/common.txt # 3
/opt/SecLists/Discovery/Web-Content/big.txt # 4
/opt/SecLists/Discovery/Web-Content/raft-medium-words.txt # 5
/opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt # 6
/opt/SecLists/Discovery/Web-Content/raft-medium-words-lowercase.txt # 7

# Combine all 6 wordlists in 1.
```

### For extensions

```bash
/opt/SecLists/Discovery/Web-Content/web-extensions.txt # Web extensions (39)
/opt/SecLists/Discovery/Web-Content/raft-small-extensions-lowercase.txt # Extensions wordlist (914)
```

### For usernames

```
/opt/SecLists/Usernames/Names/names.txt
wget https://raw.githubusercontent.com/nmap/ncrack/master/lists/minimal.usr
/opt/SecLists/Usernames/Honeypot-Captures/multiplesources-users-fabian-fingerle.de.txt
/opt/SecLists/Discovery/Web-Content/big.txt
wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt
```

### API endpoints

```bash
/opt/SecLists/Discovery/Web-Content/common.txt # 1
/opt/SecLists/Discovery/Web-Content/raft-medium-words.txt # 2
wget "https://gist.githubusercontent.com/yassineaboukir/8e12adefbd505ef704674ad6ad48743d/raw/3ea2b7175f2fcf8e6de835c72cb2b2048f73f847/List%2520of%2520API%2520endpo"
```

### Misc

```bash
/opt/SecLists/Fuzzing/special-chars.txt # Characters or symbols.
/opt/SecLists/Discovery/Web-Content/burp-parameter-names.txt # Parameters
/opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt # Subdomains
/opt/SecLists/Discovery/Web-Content/tomcat.txt # Tomcat fuzzing
/opt/SecLists/Passwords/Default-Credentials/tomcat-betterdefaultpasslist.txt # Tomcat default passwords
/opt/SecLists/Passwords/Leaked-Databases/alleged-gmail-passwords.txt # Gmail passwords
```
