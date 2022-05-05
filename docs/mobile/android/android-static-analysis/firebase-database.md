# Firebase Database

## What is Firebase

Firebase is a Backend-as-a-Services primarily for mobile applications. It focuses on removing the programming burden from the back-end by providing a nice SDK as well as many other interesting things that facilitate the interaction between the application and the back-end.

## Pentest Methodology

### Manual

Therefore, some **Firebase endpoints** could be found in **mobile applications**. It is possible that the Firebase endpoint used is **misconfigured giving everyone privileges to read (and write)** to it.

This is the common methodology for finding and exploiting misconfigured Firebase databases:

* [ ] **Get the APK** of the application you can use any of the tools to get the APK from the device for this POC. [Pulling an APK from the Play Store](../pulling-an-apk-from-the-play-store.md#apk-extractor-play-store)****
* [ ] [Decompile the APK](apk-decompilers.md).
* [ ] Go to `res/values/strings.xml` folder and search for this and **search** for the keyword "`firebase`".
* [ ] You may find something like this `https://xyz.firebaseio.com/` URL.
* [ ] Next, go to the browser and **navigate to the URL found**: `https://xyz.firebaseio.com/.json`
* [ ] You may get 2 types of responses:
  * [ ] "**Access Denied**": this means it cannot be accessed, so it is well configured.
  * [ ] "**null**" **response** or a **bunch of JSON data**: This means that the database is public and you at least have read access.
    * [ ] In this case, you could **check write privileges**, an exploit to test write privileges can be found here: [https://github.com/MuhammadKhizerJaved/Insecure-Firebase-Exploit](https://github.com/MuhammadKhizerJaved/Insecure-Firebase-Exploit)
* [ ] EVEN if it says **Access denied**, it may be that in other paths it is not protected. Example: `https://xyz.firebaseio.com/dev/.json` . These paths can be found in the APK code itself or by fuzzing.

### MobSF (Firebase)

When analyzing a mobile application with MobSF, if it finds a Firebase database it will check if it is publicly available and notify you.

### Firebase Scanner

Alternatively, you can use [Firebase Scanner](https://github.com/shivsahni/FireBaseScanner), a python script that automates the above task as shown below:

```bash
python FirebaseScanner.py -f <commaSeperatedFirebaseProjectNames>
```

## Authenticated

If you have credentials to access the Firebase database you can use a tool such as [**Baserunner**](https://github.com/iosiro/baserunner) to access more easily the stored information. Or a script like the following:

```python
#Taken from https://blog.assetnote.io/bug-bounty/2020/02/01/expanding-attack-surface-react-native/
import pyrebase

config = {
  "apiKey": "FIREBASE_API_KEY",
  "authDomain": "FIREBASE_AUTH_DOMAIN_ID.firebaseapp.com",
  "databaseURL": "https://FIREBASE_AUTH_DOMAIN_ID.firebaseio.com",
  "storageBucket": "FIREBASE_AUTH_DOMAIN_ID.appspot.com",
}

firebase = pyrebase.initialize_app(config)

db = firebase.database()

print(db.get())
```

To test other actions on the database, such as writing to the database, refer to the Pyrebase documentation which can be found [here](https://github.com/thisbejim/Pyrebase).

## References

* [https://blog.securitybreached.org/2020/02/04/exploiting-insecure-firebase-database-bugbounty/](https://blog.securitybreached.org/2020/02/04/exploiting-insecure-firebase-database-bugbounty/)
* [https://medium.com/@danangtriatmaja/firebase-database-takover-b7929bbb62e1](https://medium.com/@danangtriatmaja/firebase-database-takover-b7929bbb62e1)
