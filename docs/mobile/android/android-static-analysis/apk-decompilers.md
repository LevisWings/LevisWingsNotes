# APK decompilers

First of all, to analyze an APK you should take a look at the Java code using a decompiler:

### apktool

A tool for reverse engineering 3rd party, closed, binary Android apps. It can decode resources to nearly original form and rebuild them after making some modifications. It also makes working with an app easier because of the project like file structure and automation of some repetitive tasks like building apk, etc.

```bash
# Use:
apktool d <APP>.apk
apktool d <APP>.apk -r # No decompilar los recursos (Resources)
```

#### Structure

* **assets**: Images, fonts, etc.
* **lib**: Libraries that include the Share Objects (`.so`). We can try to read them with the `strings` command.
* **META-INF**: The `AndriodManifest.xml` is found.
* **res**: Something interesting here is the `res/values/strings.xml` file that may contain sensitive information.
* **smali**: Smali is where the source code of the application is stored.

### JADX-GUI

Open **JADX-GUI** → File -> Open files -> Select `.apk` -> Done
