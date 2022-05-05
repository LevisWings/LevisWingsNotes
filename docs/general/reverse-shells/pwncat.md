# pwncat

## Install

```bash
# A virtual environment is recommended
sudo python -m venv /opt/pwncat
# Install pwncat within the virtual environment
sudo /opt/pwncat/bin/pip install git+https://github.com/calebstewart/pwncat
# This allows you to use pwncat outside of the virtual environment
sudo ln -s /opt/pwncat/bin/pwncat /usr/local/bin
```

## Use

{% hint style="info" %}
**Ctrl + D** to switch between pwncat and local shell
{% endhint %}

```bash
sudo pwncat -l -p 443
```
