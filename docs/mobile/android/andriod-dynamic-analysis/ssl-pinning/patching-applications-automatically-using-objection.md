# Patching Applications Automatically using Objection

Remember to install frida-tools FIRST and then objection:

```bash
pip3 install frida-tools
pip3 install objection
```

Then we place ourselves in the directory where we have the `.apk` and we do:

```bash
objection patchapk --source <APK> # Android
objection patchipa --source <APK> # iOS
```
