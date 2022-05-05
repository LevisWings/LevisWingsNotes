# Looking for interesting Info & Firebase

### Looking for interesting Info

Just taking a look to the `strings` of the APK you can search for **passwords**, **URLs** ([https://github.com/ndelphit/apkurlgrep](https://github.com/ndelphit/apkurlgrep)), **api** keys, **encryption**, **bluetooth uuids**, **tokens** and anything interesting... look even for code execution **backdoors** or authentication backdoors (hardcoded admin credentials to the app).

### Hardcoded strings

Often, coded strings are found in `resources/strings.xml`. Hardcoded strings can also be found in the **source code of the activity**, i.e. in UI elements, and that can occur, for example, with a test coupon code for an application.

Threat vector:

* Login bypass (username/password, or client credentials).
* Exposed URLs (http/https).
* Exposed API keys.
* Firebase URLs ([firebase.io](http://firebase.io))

Search also in other XML files: `xmls.xml` and `integers.xml`.

### Search (JADX-GUI)

To search the source code with JADX-GUI, we can do the following:

![](../../../.gitbook/assets/jadx\_gui\_search.png)

![](../../../.gitbook/assets/jadx\_gui\_search2.png)

Always try searching for:

* `api`, `secret`, `cleartext`, `bucket`, `aws`, `cred`, `password`, `user`, `id`, `url`, `firebase`, `http://`, `https://`, `SQL`, `firebase.io`, `key`, etc.

![AWS search results](../../../.gitbook/assets/injured\_android\_strings.png)

### Practical example with InjuredAndriod

An example of bypassing the login is the following:

![](../../../.gitbook/assets/injured\_android\_source\_code1.png)

There is a comparison (`if`) between what the user enters (`obj`) with a variable (`a2`) that executes a class/function `g` that calls the method `a`:

![](../../../.gitbook/assets/injured\_android\_source\_code2.png)

So, here we must look for the class `g`:

![](../../../.gitbook/assets/injured\_android\_source\_code3.png)

We see that it is a base64 string, and that we can decode it to see what sensitive information it contains.

### Firebase

Pay special attention to **firebase URLs** and check if it is bad configured. More information: Firebase Database
