# Hashcat

## Syntax

```bash
# Dictionary attack (0):
hashcat -a 0 -m <HASH TYPE> <HASH> <WORDLIST>
# Combined attack:
hashcat -a 1 --stdout file1 file2 # See what combinations are going to be hashcat (just to see)
hashcat -a 1 -m <HAST TYPE> <HASH> <WORDLIST1> <WORDLIST2> # Combined attack.
# Mask attack:
hashcat -a 3 -m 0 <HASH> -1 01 'ILFREIGHT?l?l?l?l?l20?1?d' # The -1 is to refer to the ?1 in the string, which will be replaced by a 0 or a 1.
# Hybrid mode (6 or 7):
hashcat -a 6 -m 0 <HASH> <WORDLIST> '?d?s' # Hydrid mode 6 (attack with dictionary and mask).
hashcat -a 7 -m 0 <HASH> -1 01 '20?1?d' <WORDLIST> # Hybrid mode 7 (attack with mask and dictionary).
```

## Rules

```bash
hashcat --stdout -r <PATH OF THE RULE> <WORDLIST> # Just to see the output of possible word combinations.
hashcat -a <ATTACK> -m <TYPE> <HASH> <WORDLIST> -r <RULE>
ls -l /usr/share/doc/hashcat/rules/ # See the default rules.
```
