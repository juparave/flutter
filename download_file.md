# Download files

This document provides guidance on downloading files, such as XLSX files, in a
Flutter application. The process involves handling permissions, making HTTP
requests, and saving the downloaded file to a local directory. We recommend
using the following dependencies:

### Dependencies

Recomended dependecies to get permissions and folders uris

```yaml
dependencies:
  path_provider: ^2.0.15
  permission_handler: ^11.0.1
```

The path_provider package is used to access local file system paths, while
permission_handler is used to request and check necessary permissions.

### Permission handling

Before downloading files, it's crucial to request and check permission from the
user. Without the necessary permissions, you won't be able to save the
downloaded file locally. You can create a permission-checking function like
this:


```dart
import 'package:permission_handler/permission_handler.dart';

Future<bool> checkPermission() async {
  final status = await Permission.storage.status;
  if (status != PermissionStatus.granted) {
    final result = await Permission.storage.request();
    if (result == PermissionStatus.granted) {
      return true;
    }
  } else {
    return true;
  }
  return false;
}
```

Make sure to call checkPermission() before proceeding with the download.

### File Download

Inside your Flutter widget, use a function like the following to download a file:


```dart
Future<bool> getXlsReport({required QAReportID id, required String downloadDir}) async {
    // Check for write permission and exit if not granted
    if (!await checkPermission()) {
      debugPrint('Permission not granted');
      return false;
    }

    try {
      final response = await _dio.get('/report-xlsx/$id', options: Options(responseType: ResponseType.bytes));

      // Check for a successful response
      if (response.statusCode == 200) {
        final result = response.data;

        // Get the filename from content-disposition header
        final headers = response.headers;
        final contentDisposition = headers.value('content-disposition');
        final fileName = contentDisposition?.split('filename=').last ?? 'report.xlsx';

        // Define the file path
        final filePath = '$downloadDir/$fileName';

        // Save the file to the specified download directory
        final file = File(filePath);
        await file.writeAsBytes(result, flush: true);

        if (file.existsSync()) {
          debugPrint('File saved to ${file.path}');
          return true;
        } else {
          debugPrint('Error saving file');
          return false;
        }
      } else {
        debugPrint('Failed to download XLSX file. Status code: ${response.statusCode}');
        return false;
      }
    } on DioException catch (e) {
      debugPrint('DioException: ${e.message}');
      return false;
    } catch (e) {
      debugPrint('Exception: $e');
      return false;
    }
  }
```

This function makes an HTTP request to download the file and saves it to the
specified directory.

### Android Permissions

On Android, you need to add permissions to your AndroidManifest.xml file to
access the internet and write external storage. Add the following lines inside
the <manifest> tag:


```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```


