# Compress/Decompress Cheat Sheet

{% tabs %}
{% tab title="7z" %}
```bash
7z l <FILE> # List the contents of the compressed file.
7z 
```
{% endtab %}

{% tab title="fcrackzip" %}
```bash
fcrackzip -b -D -u -p <WORDLIST> <FILE> # Unzip .zip files
```
{% endtab %}

{% tab title="zip" %}
```bash
zip <NAME>.zip <DIRECTORY or FILE> # Compress
zip -r <NAME>.zip <DIRECTORY> # Compress directory recursive
```
{% endtab %}

{% tab title="tar" %}
```bash
tar czvf <NAME>.tar.gz <DIRECTORY> # Compress
tar cf <NAME>.tar <DIRECTORY> # Compress
tar xvzf <FILE>.tar.gz # Decompress (tar.gz)
tar xf <FILE>.tar # Decompress (.tar)
```
{% endtab %}
{% endtabs %}
