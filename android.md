## Signing app for uploads

# Flutter App Signing Guide for Google Play Store

## Getting the debug key signature

```
keytool -list -v -alias androiddebugkey -keystore ~/.android/debug.keystore
```

## Creating a Keystore

To sign your Flutter app for uploading to the Google Play Store, you need to create a keystore. This is done using the `keytool` command-line utility, which is part of the Java Development Kit (JDK).

### Step 1: Generate the Keystore

Use the following command to create a keystore with alias `upload`:

```
keytool -genkey -v -keystore upload_keystore.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

This command does the following:
- Creates a keystore named `upload_keystore.jks`
- Uses JKS (Java KeyStore) as the store type
- Uses RSA algorithm with a 2048-bit key
- Sets the validity for 10,000 days
- Creates a key with the alias "upload"

### Step 2: Provide Required Information

The tool will prompt you for the following information:

1. Keystore password (you'll need to enter this twice)
2. Distinguished name information:
   - First and last name
   - Organizational unit
   - Organization name
   - City or Locality
   - State or Province
   - Two-letter country code

### Step 3: Confirm Information

After entering the required data, you'll be asked to confirm if the information is correct. Type "yes" to proceed.

### Step 4: Key Password

You'll be prompted to enter a key password. You can press ENTER to use the same password as the keystore.

## Important Notes

1. **Keep your keystore secure**: The keystore file and its passwords are crucial for future app updates. If you lose them, you won't be able to update your app.

2. **Keystore format**: The tool may warn you about the JKS format and recommend migrating to PKCS12. You can do this with the following command:

   ```
   keytool -importkeystore -srckeystore upload_keystore.jks -destkeystore upload_keystore.jks -deststoretype pkcs12
   ```

3. **Key alias**: Remember the alias you used (in this case, "upload"). You'll need it when configuring your app's build settings.

4. **Validity period**: The example uses 10,000 days (about 27 years). Ensure this is sufficient for your app's lifecycle.

## Next Steps

After creating your keystore, you'll need to:
1. Configure your app's `build.gradle` file to use this keystore for signing.
2. Ensure you keep the keystore file and passwords secure and backed up.
3. Use this keystore to sign your app before uploading to the Google Play Store.

Remember, losing access to this keystore will prevent you from updating your app in the future, so treat it as a critical development asset.
