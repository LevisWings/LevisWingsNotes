# Patching Applications Manually

```bash
apktool d -r <.apk>
wget https://github.com/frida/frida/releases/download/15.1.1/frida-gadget-15.1.1-android-x86_64.so.xz
7z x frida-gadget-15.1.1-android-x86_64.so.xz
mv frida-gadget-15.1.1-android-x86_64.so libfrida-gadget.so
mv libfrida-gadget.so <APP DIRECTORY>/lib/x86_64/
```

Now we need to inject a `System.loadLibrary("frida-gadget")` call into the application bytecode, ideally before any other bytecode is executed or any native code is loaded. A suitable place is usually the static initializer of the application's entry point classes, for example, the Main Activity of the application, found via the `AndroidManifest.xml`.

An easy way to do this is to add the following smali code in a suitable function:

```bash
const-string v0, "frida-gadget"
invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V
```

![](../../../../.gitbook/assets/smali\_patching\_manually.png)

Now we rebuild the code and sign it:

```bash
apktool b <DIRECTORY> -o repackaged.apk

# In the next command, add at least the name, organization and occupation. At the end we type Y (yes):
keytool -genkey -v -keystore custom.keystore -alias mykeyaliasname -keyalg RSA -keysize 2048 -validity 10000
# sign the APK
jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore custom.keystore -storepass <PASS> <APK>.apk mykeyaliasname
# verify the signature you just created
jarsigner -verify repackaged.apk

# zipalign the APK
zipalign 4 repackaged.apk repackaged-final.apk
```

Install the APK on the Android and that's it. Now we can use objection without problems to disable SSL Pinning:

{% hint style="danger" %}
WARNING: When you try to open the application, you will see a white screen or a screen that looks like it will not load, but this is a good sign.
{% endhint %}

```bash
objection explore

> android sslpinning disable
```

Ready, now we can configure the proxy and intercept all traffic.

### References

[https://koz.io/using-frida-on-android-without-root/](https://koz.io/using-frida-on-android-without-root/)
