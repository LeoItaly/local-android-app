# Switch from APK Version to Development Version

Use this file to switch the app on this machine from the installed **standalone APK** back to the **Expo development version**.

## Environment

- Project folder: `C:\Users\Leuro\Documents\panini-app`
- Android SDK ADB path: `C:\Users\Leuro\AppData\Local\Android\Sdk\platform-tools\adb.exe`
- Android package name: `com.leuro.paniniapp`

## Goal

Uninstall the currently installed standalone APK from the phone, reinstall the Expo development build, and get the app running again in development mode.

---

## Step 1 — Confirm the phone is connected

Open **PowerShell** and run:

```powershell
& "C:\Users\Leuro\AppData\Local\Android\Sdk\platform-tools\adb.exe" devices
```

### Pass condition
The output must show a connected device with status `device`.

Example:
```text
List of devices attached
58301FDCR004P0  device
```

### If this fails
- Check that the phone is connected by USB
- Check that USB debugging is enabled
- Check that the phone has approved this computer for debugging
- Rerun the same command until the device appears as `device`

Do not continue until this test passes.

---

## Step 2 — Uninstall the currently installed APK version

Run:

```powershell
& "C:\Users\Leuro\AppData\Local\Android\Sdk\platform-tools\adb.exe" uninstall com.leuro.paniniapp
```

### Pass condition
The command ends with:

```text
Success
```

### If this fails
- If it says the app is not installed, continue to the next step
- Otherwise stop and inspect the exact error before continuing

---

## Step 3 — Go to the project folder

Run:

```powershell
cd C:\Users\Leuro\Documents\panini-app
```

### Pass condition
PowerShell is now in:

```text
C:\Users\Leuro\Documents\panini-app
```

Do not continue until the working directory is correct.

---

## Step 4 — Install the development build again

Run:

```powershell
npx expo run:android
```

### What this should do
- Build the Android debug/development version
- Install it on the connected phone
- Start the Metro/Expo development server

### Pass condition
- The command completes without a fatal error
- The app is installed on the phone
- The project starts in development mode

### If this fails
Stop and inspect the exact error.
Typical issues to check:
- Android SDK / Gradle errors
- Device not detected
- Dependency or native build errors

Do not continue until the development build is installed successfully.

---

## Step 5 — Open the development app on the phone

On the phone:
- Find the installed app icon
- Open it

If needed, connect it to the local development server by scanning the QR code shown by Expo.

### Pass condition
The app opens on the phone as the development version, not the standalone APK workflow.

---

## Step 6 — Verify the normal development workflow works

Keep the terminal running, then use this normal daily command from the project folder whenever you want to test code changes:

```powershell
cd C:\Users\Leuro\Documents\panini-app
npx expo start
```

Then:
- Open the installed development app on the phone
- Make a small code change
- Save the file
- Confirm the change appears in the app

### Pass condition
A small code change updates in the app without rebuilding a new APK.

---

## Important rule

For this setup:

- Use **development mode** to test changes quickly
- Use **release APK builds** only when you want to update the standalone local app

You do **not** need to build a new APK for normal UI/text/logic testing in development mode.

---

## Short command summary

```powershell
& "C:\Users\Leuro\AppData\Local\Android\Sdk\platform-tools\adb.exe" devices
& "C:\Users\Leuro\AppData\Local\Android\Sdk\platform-tools\adb.exe" uninstall com.leuro.paniniapp
cd C:\Users\Leuro\Documents\panini-app
npx expo run:android
```

After that, the normal development loop is:

```powershell
cd C:\Users\Leuro\Documents\panini-app
npx expo start
```
