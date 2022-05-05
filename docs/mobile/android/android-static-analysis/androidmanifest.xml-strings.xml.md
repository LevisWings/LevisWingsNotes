# AndroidManifest.xml, strings.xml

## AndroidManifest.xml

By reading the AndriodManifest.xml you can find vulnerabilities.

### debuggable

First, check if **the application is debuggable**. A production APK should not be (or others will be able to connect to it). You can check if an application is debuggable by looking in the manifest for the `debuggable="true"` attribute inside the `<application` Example tag:

```bash
<application theme="@2131296387" debuggable="true"
```

Learn here how to find debuggable applications on a phone and exploit them.

### backup

The `android:allowBackup` attribute defines whether application data can be backed up and restored by a user who has enabled usb debugging. If the backup flag is set to **true**, it allows an attacker to take backup of application data via `adb`, even if the device is not rooted. Therefore, applications that handle and store sensitive information, such as card data, passwords, etc., should have this setting explicitly set to false, as it is set to true by default to avoid these risks.

```bash
<application android:allowBackup="false"
```

### networkSecurity

The application network security can be overwritten the defaults values with `android:networkSecurityConfig="@xml/network_security_config"`. A file with that name may be put in _**res/xml.**_ This file will configure important security settings like certificate pins or if it allows HTTP traffic. You can read here more information about all the things that can be configure, but check this example about how to configure HTTP traffic for some domains:

```bash
<domain-config cleartextTrafficPermitted="true"> <domain includeSubdomains="true">formation-software.co.uk </domain></domain-config>
```

### Exported activities

Check for exported activities within the manifest as this could be dangerous. INSERT LINK TO: Dynamic Analysis -> Exported activities

### Content Providers

If an exported provider is being exposed, you could b able to access/modify interesting information. In dynamic analysis you will learn how to abuse them.

* Check for **FileProviders** configurations inside the attribute:

```bash
android:name="android.support.FILE_PROVIDER_PATHS"
```

### **Exposed Services**

Depending on what the service is doing internally vulnerabilities could be exploited. In dynamic analysis you will learn how to abuse them.

### Broadcast Receivers (INSERT LINK TO: Broadcast Receivers)

### URL scheme

Read the code of the activity managing the schema and look for vulnerabilities managing the input of the user.

* **minSdkVersion**, **targetSDKVersion**, **maxSdkVersion**: They indicate the versions of Android the app will run on. It's important to keep them in mind because from a security perspective, supporting old version will allow known vulnerable versions of android to run it.

## strings.xml/resources.arsc

Reading **resources.arsc/strings.xml** you can find some **interesting info**:

* API Keys
* Custom schemas
* Other interesting info developers save in this file

For example, in the source code it may try to compare the user input with the value of a variable located in the resources, usually in `strings.xml`:

![Source code](../../../.gitbook/assets/apk\_strings1.png)

![strings.xml, R.string.cmVzb3VyY2VzX3lv](../../../.gitbook/assets/apk\_strings2.png)
