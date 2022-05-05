# OpenSSL CheatSheet

### Generate RS512, RS384 and RS256 keys

{% tabs %}
{% tab title="ssh-keygen + openssl" %}
```bash
# RS512
ssh-keygen -t rsa -b 4096 -E SHA512 -m PEM -P "" -f RS512.key
openssl rsa -in RS512.key -pubout -outform PEM -out RS512.key.pub
# RS384
ssh-keygen -t rsa -b 3072 -E SHA354 -m PEM -P "" -f RS384.key
openssl rsa -in RS384.key -pubout -outform PEM -out RS384.key.pub
# RS256
ssh-keygen -t rsa -b 2048 -E SHA256 -m PEM -P "" -f RS256.key
openssl rsa -in RS256.key -pubout -outform PEM -out RS256.key.pub
```
{% endtab %}

{% tab title="Only openssl" %}
```bash
# Generate private key:
openssl genrsa -out private.pem 2048
# Generate public key:
openssl rsa -in private.pem -out public.pem -pubout
```
{% endtab %}
{% endtabs %}

