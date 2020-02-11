# Publishing FLutter Apps

To avoid Non-public API usage - The app references non-public symbols in Frameworks/Flutter.framework/Flutter: _ptrace. Clean workspace as follows

    $ flutter clean
    $ rm -rf ios/Flutter/Flutter.framework
    $ flutter build ios --release
    
After that, create binary from Xcode **Product** -> **Archive**

#### When in doubt, just wipe and reinstall, rather than manually deal with cascading dependencies.

    $ rm -rf ios/Pods ios/Podfile.lock
    $ rm ~/.pub-cache/hosted/pub.dartlang.org/
    $ flutter clean
    $ flutter packages get
    $ pod repo update
    $ cd ios
    $ pod install
 
