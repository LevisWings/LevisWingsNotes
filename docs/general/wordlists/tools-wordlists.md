# Tools (Wordlists)

{% tabs %}
{% tab title="Cewl" %}
CeWL is a Ruby application that spiders a given url to a specific depth and returns a wordlist.

{% hint style="info" %}
It is recommended not to use `-o`/`--offsite` because that would also analyze external links and we would be entering a gray area of legality.
{% endhint %}

### Installation

```bash
# Ubuntu/Debian
sudo apt-get install cewl
# Arch
git clone https://github.com/digininja/CeWL
cd CeWL
bundle install #If we don't have bundle installed, we do: gem install bundle
```

### Use

```bash
cewl -d <depth to spider> -m <minimum word length> -w <output wordlist> <url of website>
cewl -w <OUTPUT WORDLIST>.txt <TARGET> # To obtain a wordlist with the words that abound on the page.
cewl -d 5 -m 6 -e <URL> -w <OUTPUT WORDLIST>.txt # Another example with the -e parameter (includes email addresses)
cewl -e --email_file <OUTPUT WORDLIST>.txt http://<TARGET>/ # To collect emails from the site.
cewl --meta-temp-dir /home/leviswings/cewl -a --meta_file <OUTPUT WORDLIST>.txt <TARGET> # To extract metadata.
```

### References

* [https://www.hackingarticles.in/comprehensive-guide-on-cewl-tool/](https://www.hackingarticles.in/comprehensive-guide-on-cewl-tool/)
{% endtab %}

{% tab title="Crunch" %}
Crunch is a word list generator in which you can specify a standard character set or a set of characters that you specify. crunch can generate all possible combinations and permutations.

### Use

```shell
# Basic syntax:
crunch <MINUMUM LENGTH> <MAXIMUM LENGTH> <CHARSET> -t <PATTERN> -o <OUTPUT FILE>
# To generate a password between 3 and 6 characters:
crunch 3 6 -o passwords.txt
# To use only certain letters:
crunch 3 6 abcdefg12345 -o passwords.txt
# To generate a password with a maximum and minimum of 7, which has at the beginning "pepe" and then 3 numbers and 1 symbol.
crunch 7 7 -t pepe%%%^ # 
```

### Patterns

```bash
@ # Lowercase
, # Uppercase
% # Numbers
^ # Special characters (or symbols)
```
{% endtab %}

{% tab title="CUPP" %}
CUPP stands for Common User Password Profiler, and is used to create very specific and personalized word lists based on information obtained from social engineering and OSINT. People tend to use personal information when creating passwords, such as phone numbers, pet names, birth dates, etc. CUPP takes this information and creates passwords from it. These word lists are mostly used to access social networking accounts.

### Installation

```bash
git clone https://github.com/Mebus/cupp
```

### Use

The "`-i`" option is used to run in interactive mode, prompting CUPP for information about the target.

```bash
python3 cupp.py -i
```
{% endtab %}

{% tab title="wordlistctl" %}
Script to fetch, install, update and search wordlist archives from websites offering wordlists with more than 6400 wordlists available.

### Installation

```bash
# Arch:
pacman -S wordlistctl
```

### Use

```bash
# Search wordlist:
wordlistctl search rockyou # Search both locally and remotely
wordlistctl fetch -l rockyou # It only searches locally, that is, if it is already downloaded.
# Download wordlist:
wordlistctl fetch rockyou # Download all wordlists with the name "rockyou".
wordlistctl fetch <WORDLIST> # Download a specific wordlist.
# Decompress wordlist:
wordlistctl fetch -l rockyou -d

# Search wordlist by categories:
wordlistctl list -g usernames
wordlistctl list -g passwords
wordlistctl list -g discovery
wordlistctl list -g fuzzing
wordlistctl list -g misc
```
{% endtab %}

{% tab title="lyricpass" %}
Password word list generator using song lyrics for audits / targeted brute force attacks. Useful for penetration testing or security research.

```bash
# Installing:
git clone https://github.com/initstring/lyricpass
cd lyricpass
# Use:
python3 lyricpass.py -a <ARTIST>
```
{% endtab %}

{% tab title="pnwgen" %}
A very flexible Python-based phone number list generator. Obviously, more than 30% of users have their cell phone numbers as passwords. Sometimes you need to get a list of words based on phone numbers for the chosen region, but you have a very slow Internet connection.

It is very useful for targeted password attacks, because if someone lives in a place, they can check the structure or example of a phone number and generate a dictionary.

```bash
# Installing:
git clone https://github.com/toxydose/pnwgen
cd pnwgen
# Use:
python3 pnwgen.py <PREFIX> <SUFFIX> <NUMBER OF DIGITS> # Example: python3 pnwgen.py +1721 '' 7 -> +17215433375
```
{% endtab %}
{% endtabs %}

