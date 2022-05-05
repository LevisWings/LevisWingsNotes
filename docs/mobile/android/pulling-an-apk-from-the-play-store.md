# Pulling an APK from the Play Store

{% hint style="danger" %}
Example with the [InjuredAndroid](https://github.com/B3nac/InjuredAndroid) application.
{% endhint %}

### Play Store + adb

1. Open Google Play Store.
2. Then we search for "InjuredAndroid" and install that application. Finally, we open it to see if everything works correctly:

![](../../.gitbook/assets/pulling\_apk\_playstore2.png)

3\. If we see that everything works, we go to our terminal and enter debugger mode with adb to search for the APK of this installed application:

```bash
abd shell
# Filter by the "injured" package, the application we have just installed:
pm list packages | grep injured
```

From the results obtained, we copy the name of the package to which our application belongs. The package name is found in: `package:<NAME_PACKAGE>`

```bash
pm path <NAME_PACKAGE> # Example: pm path b3nac.injuredandroid
# Copy the complete PATH and exit debugger mode.
exit # abd shell exit
```

Having the PATH complete, in our Linux terminal we will create a directory, followed by pulling the APK with adb:

```bash
mkdir test_apk
cd test_apk
adb pull <PATH> <RENAME>.apk
```

### APK Extractor (Play Store)

You can use "APK Extractor" [https://play.google.com/store/apps/details?id=com.ext.ui\&hl=e](https://play.google.com/store/apps/details?id=com.ext.ui\&hl=e).
