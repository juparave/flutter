[back](README.md)
# Firebase Auth

Add the following dependencies on `pubspec.yaml`

```yaml
  firebase_core: ^0.4.3+2
  # FlutterFire plugin for Google Analytics
  firebase_analytics: ^5.0.10
  # Firebase Authentication and Cloud Firestore
  firebase_auth: ^0.15.3+1
```

Simple main.dart file

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'package:firebase_auth/firebase_auth.dart';
import 'package:kitili/screens/home/index.dart';
import 'package:kitili/screens/login/index.dart';

Future<void> main() async {
  // Fix for: Unhandled Exception: ServicesBinding.defaultBinaryMessenger was accessed before the binding was initialized.
  WidgetsFlutterBinding.ensureInitialized();
  runApp(new MaterialApp(title: "Kitili", home: _handleWindowDisplay()));
}

Widget _handleWindowDisplay() {
  return StreamBuilder(
    stream: FirebaseAuth.instance.onAuthStateChanged,
    builder: (BuildContext context, snapshot) {
      if (snapshot.connectionState == ConnectionState.waiting) {
        return Center(child: Text("Loading"));
      } else {
        if (snapshot.hasData) {
          return HomeScreen();
        } else {
          return LoginScreen();
        }
      }
    },
  );
}

```


## Google Sign-in

To implement Google Sign-in follow instructions from [Implementing Google Sign In](https://medium.com/flutter-community/flutter-implementing-google-sign-in-71888bca24ed)

Add to the bottom of `android/app/build.gradle`

    apply plugin: 'com.google.gms.google-services'


