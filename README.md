# Flutter notes

## .hgignore file

[.hgignore](hgignore)

## Index

* [Troubleshooting Flutter](troubleshooting.md)
* [Ruby](ruby.md)
* [Upload images to server](upload_image.md)
* [Theming](theming.md)
* [Firebase Auth](firebase_auth.md)
* [UI common tips](ui_common_tips.md)
* [Layouts](layouts.md)
* [Futures](futures.md)
* [Font tips](fonts.md)
* [macOS](macos.md)

External links:

* [TDD in Flutter](https://q.agency/blog/tdd-in-flutter-with-example-application-using-riverpod-and-firebase)

## Create a new app

    $ flutter create --project-name myapp --org com.companyname --android-language kotlin --ios-language swift --description "My super app" myapp

* --org com.companyname iOS PRODUCT_BUNDLE_IDENTIFIER and applicationId for Android
* -android-language kotlin to write Android code using Kotlin
* -ios-language swift iOS code using Swift
* --description 'Your App Description' sets package description in our pubspec.yaml

Usign `skeleton` template to prep app for localization

    $ flutter create -t skeleton create --project-name myapp --org com.companyname --android-language kotlin --ios-language swift --description "My super app" myapp

## Launcher

Created and implement launcher logo with instructions from:

[How to add app launcher icons in flutter](https://medium.com/@psyanite/how-to-add-app-launcher-icons-in-flutter-bd92b0e0873a)

## Icon and Splashscreen

* Generate online bundle with https://apetools.webprofusion.com/#/tools/imagegorilla
* Generate App Icon with https://appicon.co/
* Icons library https://icons8.com

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

### MacOS

Enable desktop support

At the command line, perform the following command to enable desktop support

    $ flutter config --enable-macos-desktop

For more information, see [Desktop support for Flutter](https://docs.flutter.dev/desktop)
