[back](README.md)
# Upload images to server

## Using image_picker

[image_picker](https://github.com/flutter/plugins/tree/master/packages/image_picker) A Flutter plugin for iOS and Android for picking images from the image library, and taking new pictures with the camera.

Add dependency to *pubspec.yaml*

```yaml
  http: ^0.12.0+2

  path:

  # camera and gallery
  image_picker: ^0.6.0+10
  ```

On widget page

```dart
import 'package:image_picker/image_picker.dart';

...

  _openCamera(BuildContext context) async {
    print("openCamera");
    var picture = await ImagePicker.pickImage(
      source: ImageSource.camera,
    );
    this.setState(() {
      this.bikePhoto = picture;
    });
  }

  _openGallery(BuildContext context) async {
    print("openGallery");
    var gallery = await ImagePicker.pickImage(
      source: ImageSource.gallery,
    );
    this.setState(() {
      this.bikePhoto = gallery;
    });
  }

  // on form submit
  void _handleSubmitted() async {
    final FormState form = _formKey.currentState;
    if (!form.validate()) {
      _autovalidate = true; // Start validating on every change.
      showInSnackBar('Please fix the errors in red before submitting.');
    } else {
      form.save();
      setState(() {
        this._busy = true;
      });
      // add bikefoto to bicycle.filetype array
      if (this.bikePhoto != null) {
        String base64Image = base64Encode(this.bikePhoto.readAsBytesSync());
        String fileName = this.bikePhoto.path.split("/").last;
        String fileType = this.bikePhoto.path.split(".").last;
        this.bicycle.files = new List();
        FileType file = new FileType();
        file.name = fileName;
        file.data = base64Image;
        file.type = fileType;
        this.bicycle.files.add(file);
      }
      // add current user as bicycle owner if not defined
      if (this.bicycle.owner == null) {
        this.bicycle.setOwner = await this.authService.getUser();
      }
      putBicycle(this.bicycle).then((onValue) {
        print("putBicycle response: $onValue");
        setState(() {
          this._busy = false;
        });
        // TODO: Show alert with success message

        // TODO: Return to homescreen
        // Navigator.pop(context);

      });

    }
  }

  // part of bicycle.dart
  Future<Map> putBicycle(Bicycle bicycle) async {
    return http.ajaxPost('bicycle/post', bicycle.toJson());
  }

  // part of http-helper.dart
  Future<Map> ajaxPost(String serviceName, Map data) async {
    var responseBody = json.decode('{"data": "", "status": "NOK"}');
    var urlBase = await _getUrlBase();

    try {
      var response = await http.post(urlBase + '$_serverApi$serviceName',
          body: json.encode(data),
          headers: {
            'X-DEVICE-ID': await _getDeviceIdentity(),
            'X-TOKEN': await _getMobileToken(),
            'X-APP-ID': _applicationId,
            'Content-Type': 'application/json; charset=utf-8'
          });
      if (response.statusCode == 200) {
        responseBody = json.decode(response.body);
      }
    } catch (e) {
      // An error was received
      throw new Exception("AJAX ERROR");
    }
    return responseBody;
  }


```

## Server side using python

```python
    def handle_uploads(self, bicycle, body):
        files = body.get("files")
        if not bicycle.attachments:
            bicycle.attachments = list()
        ut = UploadTools()
        path = photos_upload_path(bicycle)
        log.debug("path: %s", path)
        attachments = list()
        attachments.extend(bicycle.attachments)
        attachments_filenames = [a.get('name') for a in attachments]
        for _file in files:
            log.debug("_file: %s", _file)

            # verify file isn't upload already
            if _file['name'] in attachments_filenames:
                # we already have this attachment, skip
                log.debug("Image file has same filename, don't upload")
                continue

            hashed_filename = ut.write_blob(_file['data'],
                                            _file['name'],
                                            _file['type'],
                                            path)

            filetype = _file['type']

            if '/' in filetype:
                filetype = filetype.split('/')[1]  # removing */ from mime_type

            attachments.append({
                'name': _file['name'],
                'hashed_filename': hashed_filename,
                'path': path,
                'type': filetype
            })
        return attachments
```

uploads.py
```python
    def write_blob(self, base64_data, filename, filetype, path=""):
        """ Writes files to disk as temporal in project uploads directory."""

        if '/' in filetype:
            filetype = filetype.split('/')[1]  # removing */ from mime_type

        hoy = dt.datetime.now()
        uuid_s = str(uuid.uuid4())
        m = hashlib.md5()
        pre_hash = "%s_%s_%s" % (filename, uuid_s, hoy)
        m.update(pre_hash.encode("utf-8"))
        hashed_filename = m.hexdigest() + '.' + filetype
        store_path = "%s/%s" % (UPLOADS_PATH, path)

        # if path does not exist
        if not os.path.exists(store_path):
            # create store path
            os.makedirs(store_path)

        file_path = "%s/%s" % (store_path, hashed_filename)

        try:
            h = open(file_path, "wb")
            base64_str = base64_data
            base64_str = base64_str[base64_str.find(",") + 1:]  # Strip unnecessary data 'data:jpeg' etc
            h.write(base64.b64decode(base64_str))
            h.close()
            log.debug('Successful writing')
        except Exception as e:
            log.debug('Error while writing blob')
            raise e

        return hashed_filename
```

