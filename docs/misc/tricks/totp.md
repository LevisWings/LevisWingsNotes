# TOTP

## Time-based one-time password

**Time-based one-time password** (**TOTP**) is a [computer algorithm](https://en.wikipedia.org/wiki/Computer\_algorithm) that generates a [one-time password](https://en.wikipedia.org/wiki/One-time\_password) (OTP) that uses the current time as a source of uniqueness.

## Generate TOTP

{% tabs %}
{% tab title="oauthtool" %}
```bash
oathtool -b --totp '2UQI3R52WFCLE6JTLDCSJYMJH4'
```
{% endtab %}

{% tab title="Python" %}
```python
#!/usr/bin/python3

import ntplib
import pyotp
from time import ctime

c = ntplib.NTPClient()
resp = c.request("10.10.10.246")
print(f"Current time on target machine: {ctime(resp.tx_time)}")

totp = pyotp.TOTP("orxxi4c7orxwwzlo")
print(f"Token: {totp.at(resp.tx_time)}")
```
{% endtab %}
{% endtabs %}
