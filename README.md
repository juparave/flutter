# Flutter notes

## .hgignore file

[.hgignore](hgignore)

## Index

* [Troubleshooting Flutter](troubleshooting.md)
* [Downloads](download_file.md)
* [Ruby](ruby.md)
* [Upload images to server](upload_image.md)
* [Theming](theming.md)
* [Firebase Auth](firebase_auth.md)
* [UI common tips](ui_common_tips.md)
* [Layouts](layouts.md)
* [Futures](futures.md)
* [Font tips](fonts.md)
* [macOS](macos.md)
* [appBar](app_bar.md)
* [iOS](ios.md)
* [Android](android.md)
* [iOS CI with fastlane](fastlane_ref.md)
* [Riverpod best practices](riverpod_best_practices.md)

External links:

* [TDD in Flutter](https://q.agency/blog/tdd-in-flutter-with-example-application-using-riverpod-and-firebase)


## Installing Flutter and Dart in linux

Flutter with fvm (flutter version manager)

```sh
# Install Dart SDK (using apt, official Google repo)
sudo apt-get update
sudo apt-get install -y apt-transport-https wget
wget -qO- https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
sudo sh -c 'wget -qO- https://storage.googleapis.com/download.dartlang.org/linux/debian/dart_stable.list > /etc/apt/sources.list.d/dart_stable.list’
sudo apt-get update
sudo apt-get install -y dart
# Add Dart to PATH
export PATH="$PATH:/usr/lib/dart/bin”
# Install FVM globally
dart pub global activate fvm
# Add FVM to PATH
export PATH="$PATH:$HOME/.pub-cache/bin”
# Install Flutter version from .fvmrc
fvm install
# Get Flutter dependencies
fvm flutter pub get
```

Flutter 

```
# Install Dart SDK (using apt, official Google repo)
sudo apt-get update
sudo apt-get install -y apt-transport-https wget
wget -qO- https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
sudo sh -c 'wget -qO- https://storage.googleapis.com/download.dartlang.org/linux/debian/dart_stable.list > /etc/apt/sources.list.d/dart_stable.list’
sudo apt-get update
sudo apt-get install -y dart
# Add Dart to PATH
export PATH="$PATH:/usr/lib/dart/bin”
flutter pub get
```


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

* [How to add app launcher icons in flutter](https://medium.com/@psyanite/how-to-add-app-launcher-icons-in-flutter-bd92b0e0873a)
* [Create adaptive icons in Flutter with flutter_launcher_icons](https://blog.logrocket.com/create-adaptive-icons-flutter-launcher-icons/)
* [Flutter launcher icons](https://pub.dev/packages/flutter_launcher_icons)

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

#### CFBundleName
A user-visible short name for the bundle. This name can contain up to 15 characters. 
The system may display it to users if CFBundleDisplayName isn't set.

#### CFBundleDisplayName
The user-visible name for the bundle, used by Siri and visible on the iOS Home screen.


```
<key>CFBundleName</key>
<string>App Name</string> // Your app name here
<key>CFBundleDisplayName</key>
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

