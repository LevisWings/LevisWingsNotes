# Path Hijacking (Windows)

## Verification

```shell
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment"
```

We must have write permissions on the most prioritized paths. For example, if you have write permissions on the second path but the first path contains the binary that the user wants to run, we cannot perform PATH Hijacking.

```bash
icacls <PATH>
```

{% hint style="info" %}
Other alternative commands, such as `set`, do not work in this case, since these commands show the local configuration. Instead, by querying the registry, we see the global configuration.
{% endhint %}

## Privilege Escalation

We can upload a malicious binary that is widely used by users.
