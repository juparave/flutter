[back](README.md)
# UI common tips

## Lock Screen Orientation

```dart
import 'package:flutter/services.dart';

class MyApp extends StatelessWidget {


  @override
  Widget build(BuildContext context) {
  
    // lock screen orientation
    SystemChrome.setPreferredOrientations([
      DeviceOrientation.portraitUp,
      DeviceOrientation.portraitDown
    ]);
    
    // MultiProvider for top-level services that can be created right away
    return MultiProvider(
      providers: [
        Provider<AuthService>(
          create: (_) => AuthServiceAdapter(
            initialAuthServiceType: initialAuthServiceType,
          ),
          dispose: (_, AuthService authService) => authService.dispose(),
        ),
      ],
      child: AuthWidgetBuilder(builder: (BuildContext context, AsyncSnapshot<User> userSnapshot) {
        return MaterialApp(
          theme: ThemeData(primarySwatch: Colors.red),
          home: AuthWidget(userSnapshot: userSnapshot),
        );
      }),
    );
  }
}

```

## Change StatusBar Color

Using SystemChrome.setSystemUIOverlayStyle

```dart
  @override
  Widget build(BuildContext context) {

    // lock screen orientation
    SystemChrome.setPreferredOrientations([
      DeviceOrientation.portraitUp,
      DeviceOrientation.portraitDown
    ]);

    SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle(
      statusBarColor: Colors.blueGrey,
      statusBarBrightness: Brightness.dark,
      systemNavigationBarColor: Colors.white,
      systemNavigationBarIconBrightness: Brightness.dark
    ));

```

Another quick method to adjust:

```dart
...
    SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.dark);
...
```
