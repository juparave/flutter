[back](README.md)

# Troubleshooting Flutter

## While building module 'GoogleUtilities' imported from ...

Most of the times when adding or upgradig dependencies the compile fails.

Some steps to consider

### Podfile

Check at the top of ios/Podfile

```ruby
platform :ios, '10.0'
```

Check at the end of ios/Podfile

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
    target.build_configurations.each do |config|
      config.build_settings.delete 'IPHONEOS_DEPLOYMENT_TARGET'
    end
  end
end
```

### Clean workspace

    $ flutter clean

### superclean.sh

A shell script to clean dependencies and pods

```bash
#!/usr/bin/env bash

echo -e "\033[0;32m ** Removing ios/Podfile.lock \033[0;0m"
rm -rf ios/Pods ios/Podfile.lock
echo -e "\033[0;32m ** Removing ~/.pub-cache/hosted/pub.dartlang.org/ \033[0;0m"
rm -rf ~/.pub-cache/hosted/pub.dartlang.org/
echo -e "\033[0;32m ** Running flutter clean... \033[0;0m"
flutter clean
echo -e "\033[0;32m ** Removing ios/Flutter/Flutter.framework \033[0;0m"
rm -rf ios/Flutter/Flutter.framework
echo -e "\033[0;32m ** Running flutter packages get \033[0;0m"
flutter packages get
echo -e "\033[0;32m ** Running pod repo update \033[0;0m"
pod repo update
cd ios
echo -e "\033[0;32m ** Running pod deintegrate --verbose \033[0;0m"
pod deintegrate --verbose
echo -e "\033[0;32m ** Running pod install \033[0;0m"
pod install --verbose
cd ..
```

## Troubleshooting when clonig to another workstation

```
↳ $ flutter run
Launching lib/main.dart on iPhone Xʀ in debug mode...
Running pod install...                                              0.9s
Running Xcode build...

Xcode build done.                                            2.6s
Failed to build iOS app
Error output from Xcode build:
↳
    ** BUILD FAILED **


Xcode's output:
↳
    === BUILD TARGET Runner OF PROJECT Runner WITH CONFIGURATION Debug ===
    The file “Debug.xcconfig” couldn’t be opened because there is no such file.
    (/Users/pablito/workspace/flutter/BikeJack/bikejack/ios/Flutter/Debug.xcconfig)
    The file “Debug.xcconfig” couldn’t be opened because there is no such file.
    (/Users/pablito/workspace/flutter/BikeJack/bikejack/ios/Flutter/Debug.xcconfig)
    === BUILD TARGET Runner OF PROJECT Runner WITH CONFIGURATION Debug ===
    diff: /Podfile.lock: No such file or directory
    diff: /Manifest.lock: No such file or directory
    error: The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation.

Could not build the application for the simulator.
Error launching application on iPhone Xʀ.
```

To solve this, add to repository the following files

    {PROJECT_NAME}/ios/Flutter/Debug.xcconfig
    {PROJECT_NAME}/ios/Flutter/Generated.xcconfig
    {PROJECT_NAME}/ios/Flutter/Release.xcconfig

## Dependencies

Sometimes dependencies gets mixed with updates and version locks, one way to figure out whats happening

    flutter pub outdated
    flutter pub upgrade

    flutter channel master
    flutter upgrade

Update dependencies with this below command line:

    flutter pub upgrade --major-versions

## Firebase adding com.google.android.gms.permission.AD_ID permission

`android/app/build.gradle` add the following

```gradle
dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation('com.google.firebase:firebase-analytics') {
        exclude module: "play-services-ads-identifier"
        exclude module: "play-services-measurement"
        exclude module: "play-services-measurement-sdk"
    }
    implementation('com.google.firebase:firebase-core') {
        exclude module: "play-services-ads-identifier"
        exclude module: "play-services-measurement"
        exclude module: "play-services-measurement-sdk"
    }
}
```

These efforts and adding the following to the manifest.xml did not worked, only accept to use `AD_ID`

```
<uses-permission
        android:name="com.google.android.gms.permission.AD_ID"
        tools:node="remove" />
```

A user claim it worked: https://github.com/firebase/firebase-android-sdk/issues/2582#issuecomment-1261875254

    Configuration that worked for me
    With the configuration as below I was able to get rid of the warning in GooglePlay console:
    app/build.gradle

    configurations.all {
        exclude group: 'com.google.android.gms', module : 'play-services-ads'
        exclude group: 'com.google.android.gms', module : 'play-services-ads-identifier'
        exclude group: 'com.google.android.gms', module : 'play-services-ads-lite'
    }
    
    app/src/main/AndroidManifest.xml

        <meta-data android:name="google_analytics_adid_collection_enabled" android:value="false" />
        <meta-data android:name="google_analytics_default_allow_ad_personalization_signals" android:value="false" />
        
    Again, if you have the following line in your AndroidManifest, ⚠️try removing it.

    <!-- remove this -->
    <uses-permission android:name="com.google.android.gms.permission.AD_ID" tools:node="remove" />

