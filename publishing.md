# Publishing Flutter Apps

To avoid Non-public API usage - The app references non-public symbols in Frameworks/Flutter.framework/Flutter: _ptrace. Clean workspace as follows

    $ flutter clean
    $ rm -rf ios/Flutter/Flutter.framework
    $ flutter build ios --release
    
Be sure to select 
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
 
