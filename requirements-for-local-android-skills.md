# Requirements Before Running the Skills

## Required
- A Windows PC
- An Android phone
- A USB data cable
- Android Studio installed
- Android SDK installed through Android Studio
- At least one recent Android SDK platform installed
- Android SDK Build-Tools installed
- Android SDK Platform-Tools installed
- Android SDK Command-line Tools installed
- Google USB Driver installed on Windows if needed for the phone connection
- Node.js installed
- npm installed
- A working Expo project folder
- A generated native `android/` folder in the Expo project
- `adb` working and able to detect the Android device
- Developer Options enabled on the Android phone
- USB Debugging enabled on the Android phone
- The computer authorized for USB debugging on the Android phone
- A release keystore created for Android signing
- The keystore path configured in `android/gradle.properties`
- The release signing config set in `android/app/build.gradle`

## Needed for the first skill: build the first local Android app
- Expo Go installed on the Android phone for the early testing phase
- `expo-dev-client` installed in the project
- A successful `npx expo run:android` build at least once
- A successful `./gradlew assembleRelease` or `gradlew assembleRelease` release build

## Needed for the second skill: update the installed local Android app
- An already installed standalone APK of the app on the Android phone
- The same package name kept across updates
- The same release keystore kept across updates
- The app version increased for each update
- The Android `versionCode` increased for each update
- A way to reinstall the updated APK on the phone:
  - USB with `adb install -r`, or
  - manual APK transfer to the phone and local installation

## Optional but recommended
- A GitHub repository for backup and version control
- A code editor or AI coding environment such as Claude Code
- A backup copy of the latest working APK
- A saved copy of the keystore in a safe place
