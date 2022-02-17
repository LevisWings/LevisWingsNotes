# Admin -> SYSTEM

## Theory

## Practice

{% tabs %}
{% tab title="Service" %}
### Requirements

* Be an administrator.
* High mandatory level.
* Malicious binary.

### Practice

```shell
sc create test binPath= "<PATH>\<BINARY>.exe"
sc qc test
sc start test
# Clean Up:
sc delete test
```
{% endtab %}
{% endtabs %}

