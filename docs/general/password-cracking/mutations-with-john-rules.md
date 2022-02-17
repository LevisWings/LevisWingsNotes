# Mutations (with john rules)

Mutation rules, which can be combined several together:

* **Border mutation**: combinations of commonly used digits and special symbols may be added at the end or at the beginning, or both.

```bash
[List.Rules:BorderMutation3] # Example: pepito -> pepito12!
cAz"[0–9!@#$%^&*()_+\-={}][0–9!@#$%^&*()_+-={}][0–9!@#$%^&*()_+\-={}]"
```

* **Freak mutation**: Letters are replaced by special symbols of similar appearance.

```bash
john --wordlist=<WORDLIST> hash.txt --rules=l33t
```

* **Case mutation**: The program checks all case variations of any character:

```bash
john --wordlist=<WORDLIST> hash.txt --rules=NT
```

* **Order mutation:** The order of characters is reversed:

```
[List.Rules:Reverse] # Reverse word. ale -> ela
r 
```

* **Repetition mutation:** The same group of characters is repeated several times.

```
[List.Rules:Repeat] # Repeat word. nancy -> nancynancy
d
dd
ddd
dddd
ddddd
```

* **Vowels mutation**: Vowels are omitted or capitalized.
* **Strip mutation**: One or more characters are removed.
* **Swap mutation**: Some characters are swapped and changed.
* **Duplicate mutation**: Some characters are duplicated.
* **Delimiter mutation**: Delimiters are added between characters.
