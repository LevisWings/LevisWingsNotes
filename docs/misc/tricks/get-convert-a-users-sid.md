# Get/Convert a user's SID

{% tabs %}
{% tab title="SID -> User" %}
```bash
wmic useraccount where sid='<SID>' get name
wmic useraccount where sid='<SID>'
Convert-SidToName <SID> # (PowerView) Convert the SID to a name.
```
{% endtab %}

{% tab title="User -> SID" %}
```shell
whoami /user # Current user.
wmic useraccount where name='leviswings' get sid
wmic useraccount where name='leviswings'
```
{% endtab %}
{% endtabs %}
