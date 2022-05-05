# 📚 Theory (Windows)

## Windows Versions

Major Windows operating systems and associated version numbers:

| Names                                | Version Number |
| ------------------------------------ | -------------- |
| Windows NT 4                         | 4.0            |
| Windows 2000                         | 5.0            |
| Windows XP                           | 5.1            |
| Windows Server 2003, 2003 R2         | 5.2            |
| Windows Vista, Server 2008           | 6.0            |
| Windows 7, Server 2008 R2            | 6.1            |
| Windows 8, Server 2012               | 6.2            |
| Windows 8.1, Server 2012 R2          | 6.3            |
| Windows 10, Server 2016, Server 2019 | 10.0           |

{% hint style="info" %}
If you want to know how to enumerate Windows versions through a console, you can go to [this link](initial-enumeration-windows.md#windows-version).
{% endhint %}

## Windows Structure

The root directory in Windows is `<drive_letter>:\`, where it is usually the `C:\` drive, which is where the operating system is installed. Other physical and virtual drives are assigned other letters, for example, **USB** (`F:\`).

### Directory structure of the boot partition (Windows)

* **AppData**: All data and configuration information of installed applications per user is stored here. Within this folder, there are 3 subfolders. The **Roaming** folder is where the data of applications that "roam" between computers with the same account is stored. The **Local** folder is where information associated with a single computer is stored and is never synchronized over the network. The **LocalLow** folder is designed to house "low intensity" or lower data integrity level applications, which are those that run with specific, more restricted security settings. For example, if an application needs to run in protected mode, its data will always be in LocalLow and not Local.
* **Perflogs**: Stores system problems and other performance reports, but is empty by default.
* **Program Files**: Third-party applications installed on the computer are found. On 32-bit systems, all 16-bit and 32-bit programs are installed here, while on 64-bit systems, only 64-bit programs are installed here.
* **Program Files (x86)**: Third-party applications installed on the computer are found. This folder is only available in 64-bit editions of Windows, where 32-bit and 16-bit programs are installed.
* **ProgramData**: Hidden folder containing all data, settings and user files required for certain installed software. This directory contains application data for all users.
* **System, System32, SysWOW64**: Folder containing all the DLLs required for the main functions of Windows and its API. By default, Windows searches these folders whenever a program requests to load a DLL without specifying an absolute path.
* **Users**: It contains the profiles of the users that connect to the system and contains the following two folders: Public and Default.
* **Windows**: Folder containing most of the files needed for the Windows operating system.
* **WinSxS**: It stores the files needed to recover your system. System updates can be downloaded to your system and then stored in the WinSxS folder.

## File System

NTFS (New Technology File System) is the default Windows file system since Windows NT 3.1. NTFS is reliable and can restore file system consistency in case of system failure or power loss, and provides security by allowing us to set granular permissions on both files and folders. It also has built-in journaling, which means that file modifications are logged. However, many older and mobile devices do not support NTFS natively.

### Permissions

NTFS main permissions:

* **Full Control**: Permission to read, write, modify and delete files/folders.
* **Modify**: Permission to read, write and delete files/folders.
* **Write**: Permission to add files to folders and subfolders and write to a file.
* **Read**: Permission to view and list folders and subfolders and to view the contents of a file.
* **List Folder Contents**: Permission to view and list folders and subfolders, as well as execute files (folders only inherit this permission).
* **Read and Execute**: Permission to view and list files and subfolders, as well as execute files (files and folders inherit this permission).
* **Traverse Folder**: Permission that allows or denies the ability to move through folders to reach other files or folders.

### icacls

While the GUI exists to manage NTFS permissions in Windows, we can use the `icacls` (Integrity Control Access Control List) command.

`icacls` displays or modifies the discretionary access control lists (DACLs) in the specified files, and applies the stored DACLs to the files in the specified directories.

```bash
icacls <PATH>/<FILE/DIRECTORY>
```

```bash
C:\Users\leviswings>icacls c:\windows
c:\windows NT SERVICE\TrustedInstaller:(F)
           NT SERVICE\TrustedInstaller:(CI)(IO)(F)
           NT AUTHORITY\SYSTEM:(M)
           NT AUTHORITY\SYSTEM:(OI)(CI)(IO)(F)
           BUILTIN\Administrators:(M)
           BUILTIN\Administrators:(OI)(CI)(IO)(F)
           BUILTIN\Users:(RX)
           BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
           CREATOR OWNER:(OI)(CI)(IO)(F)
           APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(RX)
           APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)
           APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(RX)
           APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)
```

### Inheritance rights

Inheritance rights can precede either form and are applied only to directories:

* `(CI)`: container inherit = destination folder, child folder, grandchild folder.
* `(OI)`: object inherit = destination folder, child object (file), grandchild object (file).
* `(IO)`: inherit only
* `(NP)`: do not propagate inherit
* `(I)`: permission inherited from parent container

Combinations:

* `(OI)` + `(IO)` = "**Files only**". Child object (folder file), grandchild object (subfolders file). ACE is not applied to this container, but is propagated to the files it contains. Subfolders do not receive this ACE.
* `(CI)` + `(IO)` = "**Subfolders only**". Child folder, grandchild folder. ACE does not apply to this container, but propagates to subfolders. Does not propagate to contained files.
* `(CI)` + `(OI)` = "**This folder, subfolders and files**". Target folder, child folder, child object (file), grandchild folder, grandchild object (file). All subordinate objects inherit this ACE, unless they are configured to block ACL inheritance altogether.
* `(CI)` + `(OI)` + `(IO)` = "**Subfolders and files only**". Child folder, child object (file), grandchild folder, grandchild object (file). ACE does not apply to this container, but propagates to both the subfolders and the files it contains.

{% hint style="info" %}
More information here: [https://ss64.com/nt/icacls.html](https://ss64.com/nt/icacls.html)
{% endhint %}

### Simple rights / Specific rights

Permission is a permission mask and can be specified in one of two forms:

#### Sequence of simple rights

* `F` : Full access
* `D` :  Delete access
* `N` :  No access
* `M` :  Modify access
* `RX` :  Read and execute access
* `R` :  Read-only access
* `W` :  Write-only access

#### Comma-separated list in parentheses of specific rights

* `DE` - Delete
* `RC` - read control
* `WDAC` - write DAC
* `WO` - write owner
* `S` - synchronize
* `AS` - access system security
* `MA` - maximum allowed
* `GR` - generic read
* `GW` - generic write
* `GE` - generic execute
* `GA` - generic all
* `RD` - read data/list directory
* `WD` - write data/add file
* `AD` - append data/add subdirectory
* `REA` - read extended attributes
* `WEA` - write extended attributes
* `X` - execute/traverse
* `DC` - delete child
* `RA` - read attributes
* `WA` - write attributes

### Modify permissions

```bash
icacls "<PATH TO FILE/DIRECTORY>" /grant <USERNAME/GROUP>:<PERM> # The <PERM> must be one of the permissions.
icacls "<PATH TO FILE/DIRECTORY>" /grant:r <USERNAME/GROUP>:<PERM>
icacls "<PATH TO FILE/DIRECTORY>" /remove <USERNAME/GROUP> # Remove all occurrences of User from the acl.
```

### Share Permissions

The Server Message Block (SMB) protocol is used in Windows to connect shared resources such as files and printers. It is used in environments of different sizes. SMB uses shared permissions, which are as follows:

* **Full Control**: Users can perform Change and Read permissions, and change permissions of NTFS files and subfolders.
* **Change**: Users can read, edit, delete and add files and subfolders.
* **Read**: Users can view the contents of files and subfolders.

It should be clarified that NTFS permissions and sharing permissions are not the same but both can be applied to the same resource.

### NTFS vs Share permissions

* NTFS permissions apply to the system where the folder and files are hosted. Folders created in NTFS inherit the permissions of the parent folders by default, although this inheritance can be disabled. These permissions are considered whether you are connected locally or via RDP, even if it is a shared folder with shared permissions. Shared permissions are applied when the folder is accessed via the SMB protocol, usually from a different system over the network.
* Shared permissions provide less granular control than NTFS permissions.

## Windows Services & Processes

### Windows Services

Services allow the creation and management of long-running processes, and in Windows they can be started automatically at system startup and also continue to run in the background even after the user logs out of his or her account on the system.

In Windows, services play a very important role, since they perform network functions, updates, user credentials management, etc.

They can be managed in two ways:

* **GUI**: services.msc -> Service Control Manager (SCM)
* **Command line**: sc.exe or Get-Service (PowerShell)

Characteristics of services in Windows:

* Services can be in three states: running, stopped and paused.
* When running or stopping a service, the status of the service will change to "Starting" or "Stopping" until the specified action is completed.
* There are three types of services: Local Services, Network Services and System Services.
* They can be configured to start manually or automatically.

## Services Permissions

We can examine the permissions of the services using the following command:

```bash
# CMD:
sc sdshow wuauserv

D:(A;;CCLCSWRPLORC;;;AU)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SY)
# PowerShell:
Get-ACL -Path HKLM:\System\CurrentControlSet\Services\wuauserv | Format-List
```

Every named object in Windows is a securable object, and even some unnamed objects are securable, so if it is securable (in Windows), it will have a security descriptor.

Security descriptors identify the owner of the object and a primary group containing a discretionary access control list (DACL) and a system access control list (SACL).

Generally, a DACL is used to control access to an object, and a SACL is used to account for and log access attempts.

The order to read a DACL is first the characters after the 3 semicolons, then the character after the 2 semicolons, and finally the characters in the middle of the semicolons.

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
