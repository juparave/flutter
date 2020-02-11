# Publishing Flutter Apps

## iOS

To avoid Non-public API usage - The app references non-public symbols in Frameworks/Flutter.framework/Flutter: _ptrace. Clean workspace as follows

    $ flutter clean
    $ rm -rf ios/Flutter/Flutter.framework
    $ flutter build ios --release
    
Be sure to select Generic iOS device.

![GenericiOS Device](/assets/generic_ios_device.png)


After that, create binary from Xcode **Product** -> **Archive**

![Product -> Archive](/assets/product_archive.png)

#### When in doubt, just wipe and reinstall, rather than manually deal with cascading dependencies.

    $ rm -rf ios/Pods ios/Podfile.lock
    $ rm ~/.pub-cache/hosted/pub.dartlang.org/
    $ flutter clean
    $ flutter packages get
    $ pod repo update
    $ cd ios
    $ pod install
 
#### * Multiple commands produce 

Workarounds
There are two workarounds:

1. Use the legacy build system . As noted by @gi097, open ios/Runner.xcworkspace, and change the build system to Legacy Build System.
1. Use the new Xcode 10 build system.
Open ios/Runner.xcworkspace
Select the Runner project in the project navigator sidebar.
In the main view, select the Runner target, then select the Build Phases tab.
Expand the Embed Frameworks phase and select Flutter.framework from the embedded frameworks list.
Click - to remove Flutter.framework from the list (be sure to keep App.framework).

#### '...Release-iphoneos/FMDB/FMDB.framework/FMDB' does not contain bitcode

Enable bitcode on Podfile

```
...
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_BITCODE'] = 'YES'
      config.build_settings['SWIFT_VERSION'] = '4.2'
    end
  end
end
```

#### Too many symbol files

ref: [iOS newbie gotcha reminders](https://medium.com/ios-newbies/ios-swift-newbie-gotcha-reminders-5-too-many-symbol-files-1d3b20691f52)

Enable *dwarf* for `DEBUG_INFORMATION_FORMAT` on Podfile

```
...
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_BITCODE'] = 'YES'
      config.build_settings['SWIFT_VERSION'] = '4.2'
      config.build_settings['DEBUG_INFORMATION_FORMAT'] = 'dwarf'
    end
  end
end
...
```

## Android

https://play.google.com/apps/publish


#### Build app bundle

    $ flutter clean
    $ flutter build appbundle
