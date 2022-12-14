# Objection

### Launch Target Activity
```
adb shell am start -n <package_name>/<package_name.activity>
```

### Search for Registered Deeplinks
```
adb shell dumpsys package <package_name>
```

### List Activites in Hooked Application
```
android hooking list activities
```

### Search for Available Methods in Activity
```
android hooking search methods <package> <activity>
```