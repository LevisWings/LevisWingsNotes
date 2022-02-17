# Encoding

## Theory

Encoding is the process of changing a sequence of characters into a specialized format. It is commonly used for data transfer. It's bidirectional.

It's needed to provide a standard format for data. It is commonly needed for applications. It can be used for both Unicode and ASCII.

## Practice

{% tabs %}
{% tab title="base64" %}
```bash
echo -n "leviswings" | base64
echo "bGV2aXN3aW5ncwo=" | base64 -d
```
{% endtab %}

{% tab title="hex" %}
```bash
echo -n "leviswings" | xxd -ps
echo "6c6576697377696e6773" | xxd -ps -r
```
{% endtab %}

{% tab title="HTML" %}
HTML encoding ensures that text is displayed as-is in the browser and is not interpreted by the browser as HTML.
{% endtab %}

{% tab title="URL" %}
* [https://www.urlencoder.org/](https://www.urlencoder.org)
{% endtab %}

{% tab title="UUEncode" %}
Unix-2-Unix encoding has been used to send email attachments.
{% endtab %}

{% tab title="WebSphere XOR" %}
XOR encoding: [https://strelitzia.net/wasXORdecoder/wasXORdecoder.html](https://strelitzia.net/wasXORdecoder/wasXORdecoder.html)
{% endtab %}
{% endtabs %}
