---
name: expo-android-local-first-app
description: build a first expo-based android app for local installation on a physical android phone when the machine setup is already complete. use for step-by-step workflows that start after prerequisites are installed and configured, including creating the project, testing on device, creating a dev build, signing android, building a standalone apk, installing it locally, and verifying each checkpoint before moving on.
---

# Expo Android Local First App

Guide an agent through creating a first Expo Android app and installing a standalone APK on a local Android phone.

Assume prerequisites are already complete:
- Android Studio, SDK Manager, platform tools, and Google USB driver are installed
- Node.js and npm are installed
- USB debugging is enabled on the phone
- `adb devices` already works
- Expo Go is installed on the phone
- The user wants local Android testing and local APK install, not Play Store deployment

## Workflow rules

- Keep the workflow to **14 steps maximum**.
- Run the steps in order.
- After **every** step, run the listed test.
- Continue only if the test passes.
- If a test fails, stop the sequence, diagnose the exact failure, apply the smallest fix, rerun the same test, and only then continue.
- Prefer local verification over guesswork. Read terminal output, file contents, and device state directly.
- Generalize all paths, package names, and app names. Never use personal data or hard-coded user-specific paths.
- Explain clearly whether a step affects the **development build** or the **standalone APK**.
- For ambiguous tool errors, use official docs to confirm the fix before changing config.

## Output format

For each step, use this structure:

### Step N — [short action]
What to do:
1. ...
2. ...

Test:
- ...
- ...

Pass condition:
- ...

If failed:
- ...

## 14-step workflow

### Step 1 — Verify the connected Android device
What to do:
1. Open a terminal.
2. Run `adb devices` using the installed Android SDK Platform Tools.
3. Confirm a physical Android device appears with status `device`.

Test:
- `adb devices` runs without path errors.
- A physical device is listed.
- The status is `device`, not `unauthorized`.

Pass condition:
- The development machine can talk to the phone over ADB.

If failed:
- Recheck USB cable, USB debugging, RSA authorization on the phone, and SDK Platform Tools installation.
- If Windows PowerShell is used with a quoted executable path, invoke it with `&`.
- Consult `references/troubleshooting.md`.

### Step 2 — Create the Expo project
What to do:
1. Choose a project folder.
2. Run `npx create-expo-app@latest <project-name> --template default`.
3. Enter the project directory.
4. Confirm core files such as `package.json` and `app.json` exist.

Test:
- Project creation finishes without fatal errors.
- The project directory exists.
- `package.json` is present.

Pass condition:
- A clean Expo project exists locally.

If failed:
- Check Node/npm availability, network access for package install, and write permissions for the target directory.

### Step 3 — Start Expo and open the app in Expo Go
What to do:
1. Run `npx expo start` in the project directory.
2. Keep the terminal open.
3. Open Expo Go on the phone.
4. Scan the QR code or open the local URL.

Test:
- Expo starts and shows a QR code or local URL.
- Expo Go opens the starter app.

Pass condition:
- The base project runs on the phone through Expo Go.

If failed:
- Check local network reachability, Expo server status, and whether Expo Go is already installed and open.
- Login to Expo Go is optional; do not require it unless a specific flow demands it.

### Step 4 — Prove live editing works
What to do:
1. Open the main screen file in the project.
2. Change one visible text string.
3. Save the file.

Test:
- The app reloads.
- The new text appears on the phone.

Pass condition:
- The development loop works end to end.

If failed:
- Confirm the correct screen file was edited, the Expo server is still running, and the device is still attached to the right project session.

### Step 5 — Configure the app name and Android package
What to do:
1. Open `app.json`.
2. Set a human-readable Expo `name`.
3. Set a URL-safe `slug`.
4. Add `android.package` with a lowercase reverse-domain identifier such as `com.example.appname`.

Test:
- `app.json` saves cleanly.
- Expo still starts without config errors.

Pass condition:
- The app has a stable name and Android package ID suitable for local installs.

If failed:
- Recheck JSON syntax, commas, quote usage, and package name formatting.

### Step 6 — Install the development client package
What to do:
1. Run `npx expo install expo-dev-client`.
2. Confirm the dependency is added to `package.json`.

Test:
- Installation finishes normally.
- `expo-dev-client` appears in dependencies.

Pass condition:
- The project is ready for a local Android dev build.

If failed:
- Check dependency resolution errors and align package versions with the project’s Expo SDK.

### Step 7 — Build and install the Android development build
What to do:
1. Stop any existing Expo server.
2. Run `npx expo run:android`.
3. Wait for native project generation, Android build, and install to finish.

Test:
- The command finishes without fatal errors.
- A project-specific app is installed on the phone.
- The app opens from its own icon.

Pass condition:
- The phone has a development build for this project, not just Expo Go.

If failed:
- Check Android SDK, Gradle, Java, and device connectivity.
- Confirm the generated `android/` directory exists after a successful native generation step.

### Step 8 — Verify the normal development workflow uses the installed app
What to do:
1. Run `npx expo start`.
2. Open the installed development app from its icon.
3. Let it connect to the running development server.
4. Change a visible text string again and save.

Test:
- The installed development app connects without Expo Go.
- The text update appears after save.

Pass condition:
- Daily development works through the installed dev build.

If failed:
- Confirm the development client is installed, the server is running, and the app is opening the correct local project.

### Step 9 — Create the Android signing key
What to do:
1. Run `keytool -genkey -v -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -keystore release.keystore -alias <alias> -dname "CN=<package>,OU=,O=,L=,S=,C=US"` from the project root.
2. Choose and save the keystore and key passwords.

Test:
- `release.keystore` is created.
- The command completes without a fatal error.

Pass condition:
- A release signing key exists for future local updates.

If failed:
- Confirm `keytool` is available.
- Use passwords of at least six characters.

### Step 10 — Add signing values to Gradle properties
What to do:
1. Open `android/gradle.properties`.
2. Add:
   - `MYAPP_UPLOAD_STORE_FILE=../keystores/release.keystore`
   - `MYAPP_UPLOAD_KEY_ALIAS=<alias>`
   - `MYAPP_UPLOAD_STORE_PASSWORD=<keystore-password>`
   - `MYAPP_UPLOAD_KEY_PASSWORD=<key-password>`
3. Place the keystore at `android/keystores/release.keystore`.

Test:
- The keystore file exists at the configured location.
- `gradle.properties` contains all four lines.

Pass condition:
- Gradle knows where the keystore is and how to access it.

If failed:
- Check relative path correctness from `android/app/build.gradle` to the keystore.
- Use the standardized `android/keystores/release.keystore` location.

### Step 11 — Add release signing config in `android/app/build.gradle`
What to do:
1. Open `android/app/build.gradle`.
2. Add a `signingConfigs.release` block that reads the Gradle properties.
3. Set `buildTypes.release.signingConfig signingConfigs.release`.

Test:
- The file saves without syntax errors.
- The release build block points to `signingConfigs.release`.

Pass condition:
- The Android release build is wired to the custom signing key.

If failed:
- Recheck Groovy syntax and make sure the signing config is inside the `android {}` block.

### Step 12 — Build the standalone release APK
What to do:
1. Open a terminal in the `android/` directory.
2. Run `./gradlew assembleRelease` on macOS/Linux or `./gradlew.bat assembleRelease` / `gradlew assembleRelease` on Windows depending on the shell.
3. Wait for the release APK to build.

Test:
- The build finishes successfully.
- `app-release.apk` appears under `android/app/build/outputs/apk/release/`.

Pass condition:
- A signed standalone APK exists.

If failed:
- Check keystore path, passwords, alias, Java/Gradle installation, and exact build logs.
- If Gradle cannot find the keystore, verify the file was moved to `android/keystores/release.keystore` and that `MYAPP_UPLOAD_STORE_FILE` matches.

### Step 13 — Install the standalone APK on the phone
What to do:
1. Connect the phone through USB or another ADB connection.
2. Install with ADB using the built APK.
3. If PowerShell is used with a quoted `adb.exe` path, prefix the command with `&`.

Test:
- The install command ends with `Success`.
- The standalone app appears in the app drawer.

Pass condition:
- The release APK is installed on the device.

If failed:
- If the error is `INSTALL_FAILED_UPDATE_INCOMPATIBLE`, uninstall the existing app with the same package name and reinstall the release APK.
- If the shell rejects the command, correct the shell syntax instead of changing the APK.

### Step 14 — Verify the standalone app works without the laptop
What to do:
1. Stop the Expo server.
2. Disconnect the USB cable.
3. Close Expo Go.
4. Open the installed standalone app from its own icon.

Test:
- The app opens without the Expo server.
- The app shows the latest built content.

Pass condition:
- The local Android app is fully installed and usable without the laptop.

If failed:
- Confirm the standalone APK was installed, not only the development build.
- Rebuild the release APK and reinstall if necessary.

## Update rule for later local releases

When the user later wants to update the standalone app:
1. Change the code.
2. Increase the Android version values (`versionCode`, and where applicable `versionName`).
3. Rebuild the signed release APK.
4. Install it over the existing app with the same signing key.

Do not treat saving source files as a standalone-app update. The standalone app only changes after a new signed APK is built and installed.

## Troubleshooting

For failure modes seen in this workflow, consult `references/troubleshooting.md`.
