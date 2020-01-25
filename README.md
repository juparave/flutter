# Flutter notes

## .hgignore file

[.hgignore](hgignore)

## Index

* [Troubleshooting Flutter](troubleshooting.md)
* [Upload images to server](upload_image.md)
* [Theming](theming.md)

## Create a new app

    $ flutter create --androidx -t app --org com.companyname -a kotlin -i swift --description 'Your App Description' myapp
    
* --androidx Option Migration androidX, Platform languages
* -t app template (default) Generate a Flutter application
* --org com.companyname iOS PRODUCT_BUNDLE_IDENTIFIER and applicationId for Android
* -a kotlin to write Android code using Kotlin
* -i swift iOS code using Swift
* --description 'Your App Description' sets package description in our pubspec.yaml

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
