# 📚 Theory

## Windows Security

Security is a critical issue in Windows operating systems. Windows systems have many moving parts that present a large attack surface. It has many built-in features that can be abused and has suffered from a wide variety of critical vulnerabilities, resulting in remote and local exploits that are popular in the infosec field.

In addition, the large number of built-in applications, functions and settings, Windows systems can be misconfigured, thus allowing attacks even if they are fully patched due to human error.

Windows systems follow certain security principles. These are units of the system that can be authorized or authenticated for a given action. These units include users, computers on the network, threads or processes. The principles are designed to make it difficult for attackers or malicious software to gain unauthorized access and exploit the system unintentionally.

## Windows Security Model

This model focuses on 3 main parts:

| Part      | Description                                                                                                                                        |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Resources | Directories, files and registries.                                                                                                                 |
| Processes | Processes that use system resources and are executed in the context of a specific user (SYSTEM, Administrator, local user, service account, etc.). |
| Kernel    | The system kernel decides which process can access a certain resource.                                                                             |

How is access to resources granted or denied?

On the resource side, each resource has its own security descriptor that is composed of the Owner, the Group and the Access Control Lists (ACLs), the latter of which indicates who can and cannot access a given resource.

Now, on the other side, processes use access tokens, which are objects that describe the security context of a process and are created at login.

Finally, on the kernel side, the Security Reference Monitor checks whether a process call has access to the requested resource.

Finally, on the kernel side, the Security Reference Monitor checks whether a process call has access to the requested resource. To verify this, it performs the following steps:

* Verify the integrity level (UAC). There are 4: Low, Medium, High and SYSTEM. Users generally have the Medium level, so if they need to execute or access a resource that requires a higher level, they will be denied access.
* Check the Owner.
* Verify the ACL.

## Windows Tokens

An [_access token_](https://docs.microsoft.com/en-us/windows/desktop/SecGloss/a-gly) is an object that describes the [_security context_](https://docs.microsoft.com/en-us/windows/desktop/SecGloss/s-gly) of a [_process_](https://docs.microsoft.com/en-us/windows/desktop/SecGloss/p-gly) or thread. The information in a token includes the identity and privileges of the user account associated with the process or thread. When a user logs on, the system verifies the user's password by comparing it with information stored in a security database. If the password is [_authenticated_](https://docs.microsoft.com/en-us/windows/desktop/SecGloss/a-gly), the system produces an access token. Every process executed on behalf of this user has a copy of this access token. Access tokens are issued by the Local Security Authority Subsystem Service (lsass) when a user logs in, i.e. when they are authorized.

The system uses an access token to identify the user when a thread interacts with a [securable object](https://docs.microsoft.com/en-us/windows/win32/secauthz/securable-objects) or tries to perform a system task that requires privileges. Access tokens contain the following information:

* The [security identifier](https://docs.microsoft.com/en-us/windows/win32/secauthz/security-identifiers) (SID) for the user's account
* SIDs for the groups of which the user is a member
* A [_logon SID_](https://docs.microsoft.com/en-us/windows/desktop/SecGloss/l-gly) that identifies the current [_logon session_](https://docs.microsoft.com/en-us/windows/desktop/SecGloss/l-gly)
* A list of the [privileges](https://docs.microsoft.com/en-us/windows/win32/secauthz/privileges) held by either the user or the user's groups
* SID of the owner of the user who created it.
* SID of the primary group, usually the first group they were part of.
* The default [DACL](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control-lists) that the system uses when the user creates a securable object without specifying a [_security descriptor_](https://docs.microsoft.com/en-us/windows/desktop/SecGloss/s-gly)
* Source of the access token, such as the local system or a domain.
* Token type, primary or impersonation.
* Optional list of restricted tokens.

### Token types

* **Primary**: The standard access token provided to the user at login.
* **Impersonation**: Used when a process needs to run as another user.
* **Restricted**: Used when the CreateRestrictedToken api call is made. The restricted part may be needed when a user only needs a smaller access scope than the original primary token.

## Privileges & Access Rights

**Privileges** determine the type of operations a user can perform on the system. An example of such operations could be shutting down the system, changing user privileges, loading device drivers, etc.. In short, they control access to tasks related to the Windows system.

**Access rights** control access to secure objects, which can be directories, files, registry keys, Windows services, pipes, access tokens, jobs, printers, etc.

Privileges are assigned to user and group accounts. In the case of access rights, they are assigned to the access control lists (ACL) of objects.

## UAC

**User Access Control** (UAC) is a Windows security feature to prevent malicious programs from executing or manipulating processes that could harm the computer or its contents. Within UAC there is Admin Approval Mode, which is designed to prevent unwanted software from being installed without the administrator's knowledge or to prevent system-wide changes from being made.

When UAC is enabled, applications and tasks always run under the security context of a non-administrator account, unless an administrator explicitly authorizes these applications/tasks to have administrator-level access to the system to run.

You have probably already seen the approval prompt if you have installed specific software, and your system has asked you for confirmation if you want it installed. This consent prompt stops the execution of scripts or binaries that malware or attackers attempt to run until the user enters the password or confirms the execution:

![Consent prompt](../../../.gitbook/assets/consent\_prompt.gif)

![Credential prompt](../../../.gitbook/assets/credential\_prompt\_uac.gif)

When UAC is running, a user can log on to your system with their standard user account. When processes are launched using a standard user token, they can perform tasks using the rights granted to a standard user. Some applications require additional permissions to run, and UAC can provide additional access rights to the token for them to run correctly.

This [page](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) takes an in-depth look at how UAC works, including the login process, user experience, and UAC architecture. Administrators can use security policies to configure UAC operation specifically for their organization locally (using secpol.msc), or configure and distribute it through Group Policy Objects (GPOs) in an Active Directory domain environment. The different configurations are discussed in detail [here](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings).
