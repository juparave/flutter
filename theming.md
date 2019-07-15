# Theming flutter apps

## Status bar color

![Status Bar]({{ site.url }}/assets/status_bar.png)

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

On builer function:

```dart
SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.dark.copyWith(
  statusBarColor: Colors.black, 
));
```

or

```dart
SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.dark.copyWith(
  statusBarColor: Colors.white,
));
```
