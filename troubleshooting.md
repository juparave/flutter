# Troubleshooting Flutter

[README.md](back)

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
