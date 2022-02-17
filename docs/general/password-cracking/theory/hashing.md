# Hashing

## Theory

Hashing is the process of converting a text into a string, which is unique to that particular text. Normally, a hash function always returns hashes with the same length, regardless of the type, length or size of the data. Hashing is a one-way process, which means that there is no way to reconstruct the original text from a hash.

## Identify the type of hash

{% tabs %}
{% tab title="haiti" %}
```bash
gem install haiti-hash # Installation
haiti '<HASH>'
# Documentation: https://noraj.github.io/haiti/#/
```
{% endtab %}

{% tab title="hashid" %}
```bash
hashid '<HASH>'
hashid '<HASH>' -m # To know the mode or type of hash for Hashcat.
```
{% endtab %}

{% tab title="hash-identifier" %}
```bash
hash-identifier
*Put the hash and Enter*
```
{% endtab %}

{% tab title="hashcat" %}
We copy the first part of the hash (ex: **$krb5asrep$**):

{% hint style="info" %}
**IMPORTANT**: Escape the "$" with backslashes (`\$`)
{% endhint %}

```bash
hashcat --example-hashes | grep '<PART OF HASH>' -B 2
```
{% endtab %}
{% endtabs %}

### Other references

* [https://hashcat.net/wiki/doku.php?id=example\_hashes](https://hashcat.net/wiki/doku.php?id=example\_hashes)
* [https://www.tunnelsup.com/hash-analyzer/](https://www.tunnelsup.com/hash-analyzer/)
* [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)
