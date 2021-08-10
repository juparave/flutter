# Font tips

## Monospaced type fonts using `FontFeature.tabularFigures()`

ref: [Font Features in Flutter](https://suragch.medium.com/font-features-in-flutter-320222fc171d)

```dart
appointmentTime(DateTime? date) {
  if (date == null) {
    return const Text("*ND*", style: TextStyle(
                                        color: Colors.orange, 
                                        fontWeight: FontWeight.normal, 
                                        fontSize: 16, 
                                        fontFeatures: [FontFeature.tabularFigures()]);
                                     );
  }
  
  final DateFormat formatter = DateFormat('Hm');
  return Text(formatter.format(date), style: TextStyle(
                                        color: Colors.orange, 
                                        fontWeight: FontWeight.normal, 
                                        fontSize: 16, 
                                        fontFeatures: [FontFeature.tabularFigures()]);
                                     );
}
```

![image]({{ site.url }}/flutter/blob/master/assets/FontFeature.tabularFigures.png)
![gif]({{ site.url }}/juparave/flutter/blob/master/assets/1*L_RP__onb-LHcwYbXNwD9g.gif)
