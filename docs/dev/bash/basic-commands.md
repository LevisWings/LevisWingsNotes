# Basic commands

### date

```
# %Y -> Year (2021,2022,2023)
# %m -> Month (01,09,11)
# %d -> Day (01,15,30)
# %H -> Hour (00,15,23)
# %M -> Minute (00,35,59)
# %S -> Second (00,35,59)
```

```bash
date +%Y%m%d%H%M%S # Example: 20211224132059
```

```bash
# Get date as EPOCH format
d=$(date '+%s')
# Loop for 30 seconds more
for i in {1..30}; do
 let time=$(( d + i ))
 echo $time
done
```

### grep

```bash
cat test.txt | grep -n "<WORD>" # Find on which line a word is found in the file.
cat test.txt | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' # Filter by IP address.
cat text.txt | grep  -v '^\#' | grep . # Delete comments.
for i in $(cat test.txt); do echo $i | grep -q "a" || echo "In this line, the letter \"a\" is not found."; done # Supress normal output.
```

{% hint style="danger" %}
If in some occasions, when using regular expressions with grep we get something like: "Binary file (standard input) matches", we can solve it in the following way: `... | grep -oP '...' --text`
{% endhint %}

### sed

```bash
cat /etc/passwd | head -n 1 | sed 's/root/hello/g'
cat test.txt | sed '1000,1500!d' # To print lines from 1000 to 1500.
sed -i 's/^\s*$/d' <FILE> # Delete empty lines.
sed -i 's/\\n/\n/g' # Format file.
```
