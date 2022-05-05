# Code Review

## Tips

* Review the behavior of single quotes (`'`) and double quotes (`"`) containing a variable. For example, in PHP there is a big difference:

```php
$value = uniqid();
# Single quotes:
print '$value';
$value
# Double quotes:
print "$value"
6222462266dec
```

## Code Review Tools

### General

{% embed url="https://owasp.org/www-community/Source_Code_Analysis_Tools" %}

```bash
https://www.sonarqube.org/downloads/
https://deepsource.io/signup/
https://github.com/pyupio/safety
https://github.com/returntocorp/semgrep
https://github.com/WhaleShark-Team/cobra
https://github.com/insidersec/insider

# Find interesting strings
https://github.com/s0md3v/hardcodes
https://github.com/micha3lb3n/SourceWolf
https://libraries.io/pypi/detect-secrets
```



### JavaScript

```
https://jshint.com/
https://github.com/jshint/jshint/
```

### NodeJS

```
https://github.com/ajinabraham/nodejsscan
```

### Electron

```
https://github.com/doyensec/electronegativity
```

### Python

```bash
# bandit
https://github.com/PyCQA/bandit
# pyt
https://github.com/python-security/pyt
```

### .NET

```bash
# dnSpy
https://github.com/0xd4d/dnSpy

# .NET compilation
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe test.cs
```

### Ruby

{% embed url="https://github.com/presidentbeef/brakeman" %}

```bash
# brakeman
## Install:
gem install brakeman
## Use:
cd <path/to/repo>
brakeman
brakeman -A
```

