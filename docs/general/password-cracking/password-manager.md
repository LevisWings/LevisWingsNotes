# Password Manager

## KeePass

KeePass is a free open source password manager, which helps you to manage your passwords in a secure way. You can store all your passwords in one database, which is locked with a master key. So you only have to remember one single master key to unlock the whole database.

To open a .kdbx file, we can use the following tools:

{% tabs %}
{% tab title="keepassxc (GUI)" %}
```bash
keepassxc <kdbx>
```
{% endtab %}

{% tab title="kpcli" %}
```bash
# Authentication:
kpcli -kdb <kdbx>
kpcli -kdb <kdbx> --key <KEY>
# Commands:
ls
cd <GROUP>
ls
show -a -f <ENTRY NUMBER>
```
{% endtab %}
{% endtabs %}
