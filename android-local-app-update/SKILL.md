---
name: android-local-app-update
description: update an already installed local android app built with expo/react native and a native android directory, without using the play store. use when the project setup is already complete and chatgpt needs to change app code, rebuild a signed standalone apk, install the updated apk on a local android device, and verify the update step by step. especially useful for claude code or other agents that should stop after each test gate, fix common local build issues automatically when safe, and continue only when the current step has passed.
---

# Android Local App Update

Use this skill to update an existing local Android app that is already set up and already installed on a local Android phone outside the Play Store.

Assume these prerequisites are already true:
- The Expo/React Native project already exists.
- The `android/` directory already exists.
- Android Studio, SDK tools, Node/npm, Gradle, and `adb` are already installed.
- A release keystore already exists and release signing is already configured.
- The app has already been installed on the phone at least once.

## Output contract

Always guide the workflow as a gated checklist:
1. State the current step.
2. State exactly what to change or run.
3. State the test that must pass.
4. If the test fails, stop and fix the smallest safe issue automatically when possible.
5. Continue only after the test passes.

Do not skip validation steps.
Do not use the user's personal file paths, usernames, or package names in the reusable instructions unless the current task explicitly requires project-specific values.
Use placeholders like `<project-root>`, `<sdk-path>`, `<package-name>`, `<version-name>`, and `<version-code>`.

## Workflow summary

Follow this sequence for a normal local update:
1. Inspect the project and confirm required files exist.
2. Change the app code.
3. Update Android release version values.
4. Rebuild the signed release APK.
5. Verify the APK exists.
6. Install the updated APK on the phone.
7. Verify the updated app launches without the dev server.
8. Save a backup of the APK and record the release note.

## Step 1: Inspect the project before changing anything

Confirm these files exist before continuing:
- `package.json`
- `app.json` or `app.config.*`
- `android/app/build.gradle`
- `android/gradle.properties`
- the release keystore file referenced by Gradle

Test gate:
- The project root is correct.
- The native Android directory exists.
- The keystore path resolves to a real file.

If the keystore path is broken:
- Read `android/gradle.properties`.
- Find the keystore reference variable.
- Compare it with the actual keystore location.
- Fix the path before continuing.

## Step 2: Change the app code

Make the requested code/content change in the app source.
For simple verification updates, prefer a visible text change on the main screen.

Typical target in Expo Router projects:
- `app/(tabs)/index.tsx`

Test gate:
- The intended source file was edited successfully.
- The changed text or behavior is present in source control / file contents.

## Step 3: Update the Android app version for the standalone release

Because the native `android/` directory already exists, do not rely only on Expo config for release updates.
Update the native Android version values directly in `android/app/build.gradle` inside `defaultConfig`:
- increment `versionCode`
- increment `versionName`

Also update Expo config version fields if they exist, but treat the native Gradle values as the release source of truth for this workflow.

Test gate:
- `versionCode` is higher than the currently installed standalone version.
- `versionName` matches the intended release label.

If the current installed version is unknown, use the next sequential integer for `versionCode` and a matching semantic-style `versionName`.

## Step 4: Rebuild the signed release APK

Run the Gradle release build from the native Android directory:

```bash
cd <project-root>/android
./gradlew assembleRelease
```

On Windows PowerShell, use:

```powershell
cd <project-root>\android
.\gradlew assembleRelease
```

Test gate:
- The Gradle build finishes successfully.
- No signing error remains.

If build fails, diagnose the smallest safe fix first.
Use the troubleshooting rules below.

## Step 5: Verify the APK artifact exists

Expected output location:
- `android/app/build/outputs/apk/release/app-release.apk`

Test gate:
- The APK file exists.
- The file timestamp is new enough to match the current build.

If the build completed but the file is missing, inspect the Gradle output paths and confirm the build variant.

## Step 6: Install the updated APK on the phone

Preferred method: ADB install.

PowerShell example:

```powershell
& "<sdk-path>\platform-tools\adb.exe" install -r .\app-release.apk
```

Shell example:

```bash
<adb-path>/adb install -r ./app-release.apk
```

Alternative method:
- Transfer the APK to the phone manually and install it on-device.

Test gate:
- The install ends with success.
- The existing installed app is replaced by the updated build.

If install fails with `INSTALL_FAILED_UPDATE_INCOMPATIBLE` (incompatible signatures):
- The old installed app was signed with a different key (e.g. a previous debug or Expo Go build).
- Automatically run the uninstall + reinstall sequence without asking:

PowerShell:
```powershell
adb uninstall <package-name>
adb install app-release.apk
```

Shell:
```bash
adb uninstall <package-name>
adb install app-release.apk
```

- Warn the user that local app data was removed by the uninstall.
- After reinstall, continue to Step 7.

## Step 7: Verify the updated standalone app works without the dev server

After install:
- stop Expo dev server if it is running
- disconnect USB if the user wants a strict standalone test
- open the installed app from its own icon
- do not use Expo Go

Test gate:
- The app launches from its own icon.
- The updated text or behavior is visible.
- The app works without the laptop dev server.

## Step 8: Save a rollback point

After a successful update:
- copy the release APK to a backup folder
- rename it with a readable version label
- commit the project changes
- optionally tag the Git commit

Suggested outputs:
- backup APK filename like `app-v<version-name>-working.apk`
- commit message like `release: local android update <version-name>`

Test gate:
- Backup APK exists.
- Git commit succeeds or there are no new tracked changes.

## Troubleshooting rules

Consult `references/troubleshooting.md` when a step fails.
Apply only the smallest safe fix that matches the actual error.
After fixing, rerun only the blocked step and retest.

## Agent behavior rules

- Prefer deterministic, copy-pasteable commands.
- Do not ask the user to infer hidden values when they can be read from files.
- If a command must vary by OS shell, show the correct form for the current shell.
- If a test fails, stop progression until it is fixed.
- If an automatic fix is risky or destructive, explain the risk before applying it.
- Do not suggest Play Store deployment for this workflow.
- Do not claim the standalone APK updates automatically; it must be rebuilt and reinstalled.

## Minimal reusable command template

Use this exact decision pattern:
1. Read the current project values from files.
2. Propose the smallest change.
3. Run or instruct the change.
4. Run the verification command.
5. Continue only on success.
