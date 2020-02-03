[back](README.md)
# Theming flutter apps

## Status bar color

![Status Bar]({{ site.url }}/flutter/assets/status_bar.png)

### With AppBar


```dart
Scaffold(
  appBar: AppBar(
    brightness: Brightness.light,
  )
)
```

or

```dart
Scaffold(
  appBar: AppBar(
    brightness: Brightness.dark,
  )
)
```

### Without AppBar

On builder function:

```dart
SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.dark.copyWith(
  statusBarColor: Colors.black, 
));

// also
SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.dark);

```

or

```dart
SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.dark.copyWith(
  statusBarColor: Colors.white,
));

// also
SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.light);

```
