# Flutter AppBar tricks

### Set gradient background

ref: https://stackoverflow.com/questions/50412484/gradient-background-on-flutter-appbar

```dart
return Scaffold(
  appBar: AppBar(
    title: Text(widget.title),
    flexibleSpace: Container(
      decoration: const BoxDecoration(
        gradient: LinearGradient(
          begin: Alignment.topCenter,
          end: Alignment.bottomCenter,
          colors: <Color>[Colors.black, Colors.blue]),
      ),
    ),
  ),
);
```
