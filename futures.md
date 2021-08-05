# Dart Futures

### How to use `async` code inside `map()`

`async` functions must return a `Future`, so adding `async` keyword to your callback means that your `List.map()` 
call must now return a List of Futures.  ref: [stackoverflow](https://stackoverflow.com/a/58981983)


```dart
  Future<List<Appointment>>? _loadAppointments() async {
    List<Appointment> response;
    try {
      response = await getAppointments();
      var appointments = response.map<Future<Appointment>>((appointment) async {
        if (appointment.driver == null) {
          appointment.driver = await getDriver(appointment.driverId!);
          debugPrint("Got driver: ${appointment.driver!.firstName}");
        }
        return appointment;
      });

      return await Future.wait(appointments.toList());    // <-- return a list of Futures
    } catch (error) {
      debugPrint(error.toString());
      rethrow;
    }
  }
```
