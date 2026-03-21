# Troubleshooting

Use only the matching fix for the actual error. After each fix, rerun the blocked step and retest.

## 1. PowerShell says `Unexpected token 'install'`

Cause:
- A quoted executable path was used in PowerShell without the call operator.

Fix:
Use `&` before the quoted executable path.

Example:

```powershell
& "<sdk-path>\platform-tools\adb.exe" install -r .\app-release.apk
```

## 2. `adb devices` does not show the phone

Possible causes:
- USB debugging is off.
- The phone has not authorized the computer.
- The USB cable is charge-only.
- Windows driver problem.

Fix order:
1. Confirm Developer options and USB debugging are enabled.
2. Check the phone for the authorization prompt and allow it.
3. Retry with a data-capable USB cable.
4. On Windows with a Google device, verify the Google USB driver is installed.
5. Rerun `adb devices`.

## 3. `adb devices` shows `unauthorized`

Cause:
- The phone has not approved the computer.

Fix:
- Unlock the phone.
- Accept the USB debugging authorization prompt.
- Rerun `adb devices`.

## 4. Gradle fails with `Keystore file ... not found`

Cause:
- The keystore path in Gradle properties does not match the real file location.

Fix:
1. Locate the actual keystore file.
2. Read the keystore path variable in `android/gradle.properties`.
3. Update the path so Gradle resolves the real file.
4. Keep the keystore in a stable project-relative location.

Typical stable location:
- `android/keystores/release.keystore`

## 5. APK install fails with `INSTALL_FAILED_UPDATE_INCOMPATIBLE`

Cause:
- The app currently installed on the phone was signed with a different key than the new APK.
- Common case: replacing a debug/dev build with a release build.

Fix:
1. Warn that uninstalling may remove local app data.
2. Uninstall the existing app from the device.
3. Install the new APK again.
4. Keep using the same release keystore for all future standalone updates.

## 6. Rebuilt APK installs but does not update the app

Cause:
- `versionCode` was not incremented.

Fix:
- Increase `versionCode` in `android/app/build.gradle`.
- Also update `versionName` to match the intended release label.
- Rebuild and reinstall.

## 7. The project has `android/` already, but config changes in `app.json` do not affect the release build

Cause:
- Once the native Android project exists, native Gradle values may become the practical source of truth for release builds.

Fix:
- For local standalone release updates, update `android/app/build.gradle` directly for `versionCode` and `versionName`.
- Use Expo config changes only when they are actually synced into the native project.

## 8. Expo dev build works, but the standalone APK does not change after editing source files

Cause:
- Editing source files updates development flow, not the already-installed standalone APK.

Fix:
- Rebuild the signed release APK.
- Reinstall the updated APK onto the device.

## 9. Build shows warnings about unrelated modules, but a different error stops the build

Cause:
- Not every warning is the blocker.

Fix:
- Focus on the fatal error near the end of the build output.
- Fix only the blocking error first.
- Ignore non-blocking warnings unless they later become fatal.

## 10. User wants to update the local app without using ADB every time

Cause:
- ADB install is optional, not mandatory.

Fix options:
- Keep using USB/ADB for the fastest repeatable update flow.
- Or transfer the rebuilt APK to the phone manually and install it there.
- In both cases, the APK still must be rebuilt on the computer first.
