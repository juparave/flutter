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

## Material Switch on a Dialog

Use StatefulBuilder to use setState inside Dialog and update Widgets only inside of it.

```dart
    showDialog(
        context: context,
        builder: (BuildContext context) {
          return StatefulBuilder(builder: (context, setState) {
            return AlertDialog(
              title: Text("Notas de la visita"),
              content: Form(
                key: _formKey,
                child: Column(
                  mainAxisSize: MainAxisSize.min,
                  children: <Widget>[
                    Padding(
                      padding: EdgeInsets.all(8.0),
                      child: TextFormField(
                        decoration: InputDecoration(labelText: "Notas"),
                        keyboardType: TextInputType.multiline,
                        maxLines: 4,
                        initialValue: this._visit!.notes,
                        onSaved: (value) => this._visit!.notes = value,
                      ),
                    ),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceAround,
                      children: <Widget>[
                        Expanded(
                            child: Text(
                          "Exposición adicional",
                          style: Theme.of(context).textTheme.caption,
                        )),
                        Switch(
                            activeColor: Theme.of(context).primaryColor,
                            value: this._visit!.additionalDisplay ?? false,
                            onChanged: (bool value) {
                              setState(() {
                                this._visit!.additionalDisplay = value;
                                log("additionalDisplay: " + this._visit!.additionalDisplay!.toString());
                              });
                            })
                      ],
                    ),
                    Padding(
                      padding: const EdgeInsets.all(8.0),
                      child: RaisedButton(
                        child: Text("Guardar"),
                        onPressed: () {
                          if (_formKey.currentState!.validate()) {
                            _formKey.currentState!.save();
                          }
                          Navigator.of(context).pop();
                        },
                      ),
                    )
                  ],
                ),
              ),
            );
          });
        });
```


