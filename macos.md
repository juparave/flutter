# macOS dev

## Add desktop support to an existing Flutter project

Run the following command in a terminal from the root project directory:

    flutter create --platforms=windows,macos,linux .

This adds the necessary desktop files and directories to your existing Flutter project. To add only specific desktop platforms, change the platforms list to include only the platform(s) you want to add.

## Network access

macOS needs you to request a specific entitlement in order to access the network. To do that open `macos/Runner/DebugProfile.entitlements` and add the following key-value pair.

    <key>com.apple.security.network.client</key>
    <true/>

Then do the same thing in `macos/Runner/Release.entitlements`.

You can read more about this in the Desktop support for [Flutter documentation](https://flutter.dev/desktop#setting-up-entitlements).
