# Customization and enhancement

### Scripting techniques

```bash
# Numbers
for i in {1..5}; do echo $i >> numbers.txt; done
for i in {01..05}; do echo $i >> numbers.txt; done

# 1. We convert all uppercase letters to lowercase:
cat wordlist.txt | tr '[A-Z]' '[a-z]' > custom.txt
# 2. We convert all lowercase letters to uppercase:
cat wordlist.txt | tr '[a-z]' '[A-Z]' >> custom.txt
# 3. (Optional) We convert all words with the first letter capitalized:
cat wordlist.txt | sed 's/.*/\u&/' >> custom.txt
# We add the content of the custom.txt file to wordlist.txt:
cat custom.txt >> wordlist.txt
```

### Custom Username Wordlist

```bash
git clone https://github.com/21y4d/usernameGenerator.git
cd usernameGenerator && chmod +x usernameGenerator.sh
./usernameGenerator.sh <FIRST NAME> <LAST NAME>
```

### Remove comments to wordlists

```bash
sed -i 's/^\#.*$//g' <WORDLIST> && sed -i '/^$/d' <WORDLIST>
```
