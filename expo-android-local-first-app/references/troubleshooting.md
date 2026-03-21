# Troubleshooting

Use this file only when the main step test fails. Fix the narrowest confirmed issue, rerun the same test, and only continue if it passes.

## 1. `adb devices` does not show the phone as `device`

### Symptoms
- No device listed
- `unauthorized`
- ADB path errors

### Checks
- Confirm the phone has USB debugging enabled.
- Confirm the RSA authorization prompt on the phone was accepted.
- Confirm the USB cable supports data, not only charging.
- Confirm Android SDK Platform Tools are installed.

### Fixes
- Reconnect the phone and accept the debug authorization prompt.
- Retry `adb devices`.
- On Windows PowerShell, if using a quoted full path to `adb.exe`, invoke it with `&`.

## 2. Expo Go opens but asks for URL or QR code

### Meaning
Expo Go is installed correctly. This is not a failure.

### Fix
Start the Expo server in the project directory and then scan the QR code or open the shown local URL.

## 3. Expo Go does not need login for the basic local flow

### Meaning
A login is not required for the basic create-project, run-locally, and scan-QR workflow.

### Fix
Skip login unless the chosen workflow explicitly requires an authenticated Expo service action.

## 4. PowerShell rejects a quoted executable path

### Symptom
PowerShell throws a parser error when a quoted `.exe` path is followed by arguments.

### Fix
Prefix the quoted executable path with `&`.

### Example pattern
```powershell
& "C:\path\to\adb.exe" install -r .\app-release.apk
```

## 5. Gradle cannot find the keystore

### Symptom
Release build fails with a signing validation error because the keystore file is not found.

### Checks
- Confirm the keystore file exists.
- Confirm the configured relative path matches the actual location.

### Fix
Use the stable location:
- file location: `android/keystores/release.keystore`
- Gradle property: `MYAPP_UPLOAD_STORE_FILE=../keystores/release.keystore`

Then rerun the release build.

## 6. `INSTALL_FAILED_UPDATE_INCOMPATIBLE`

### Meaning
An app with the same package name is already installed but was signed with a different key.
This usually happens when a debug/dev build exists and a release APK is installed later.

### Fix
- Uninstall the existing app with that package name from the phone.
- Reinstall the release APK.

## 7. Standalone app does not change after editing source files

### Meaning
Editing source files updates the development flow, not the installed standalone APK.

### Fix
For a standalone app update:
1. Change the code.
2. Increase Android version values.
3. Build a new signed release APK.
4. Reinstall or upgrade that APK on the phone.

## 8. Keystore creation fails because the password is too short

### Symptom
`keytool` rejects the password.

### Fix
Use a password that is at least six characters long.

## 9. Confusion between development build and standalone app

### Development build
- Installed from `expo run:android`
- Useful for fast iteration
- Depends on the laptop’s dev server while developing

### Standalone APK
- Built with the Android release build
- Installed directly on the phone
- Runs without the laptop after install

### Fix
When verifying local-phone-only behavior, test the standalone APK with the Expo server stopped and the cable disconnected.
