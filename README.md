# Flutter notes

## .hgignore file

[.hgignore](hgignore)

## Index

* [Troubleshooting Flutter](troubleshooting.md)
* [Upload images to server](upload_image.md)
* [Theming](theming.md)
* [Firebase Auth](firebase_auth.md)
* [UI common tips](ui_common_tips.md)
* [Futures](futures.md)

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

## Icon and Splashscreen

Generate online bundle with https://apetools.webprofusion.com/#/tools/imagegorilla

## Real splash

ref: https://medium.com/flutter-community/flutter-2019-real-splash-screens-tutorial-16078660c7a1

## Setting Application name

### Android
Open AndroidManifest.xml (located at android/app/src/main)

```
<application
    android:label="App Name" ...> // Your app name here
```

### iOS
Open Info.plist file (located at ios/Runner/Info.plist)

```
<key>CFBundleName</key>
<string>App Name</string> // Your app name here
```
## Recording screen

### iOS

Start recording with:

    $ xcrun simctl io booted recordVideo --codec=h264 --mask=black --force appVideo.mov
    
Stop record with control-c
