# Flutter notes

## .hgignore file

[.hgignore](hgignore)

## Index

[Troubleshooting Flutter](troubleshooting.md)

## Launcher

Created and implement launcher logo with instructions from:

[How to add app launcher icons in flutter](https://medium.com/@psyanite/how-to-add-app-launcher-icons-in-flutter-bd92b0e0873a)

## Setting Application name

### Android
Open AndroidManifest.xml (located at android/app/src/main)

```
<application
    android:label="App Name" ...> // Your app name here
```

### iOS
Open info.plist file

```
<key>CFBundleName</key>
<string>App Name</string> // Your app name here
```
