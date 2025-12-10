# Installation Guide - Splitter App

This guide will help you build and install the Splitter app on Android and iOS devices.

## Prerequisites

### For Android:
1. **Android Studio** - Download from [developer.android.com](https://developer.android.com/studio)
2. **Android SDK** - Installed via Android Studio
3. **Enable USB Debugging** on your Android device:
   - Go to Settings → About Phone
   - Tap "Build Number" 7 times to enable Developer Options
   - Go to Settings → Developer Options
   - Enable "USB Debugging"

### For iOS:
1. **Mac computer** (required for iOS development)
2. **Xcode** - Download from App Store or [developer.apple.com](https://developer.apple.com/xcode/)
3. **Apple Developer Account** (free account works for testing on your own device)
4. **CocoaPods** - Install via terminal:
   ```bash
   sudo gem install cocoapods
   ```

---

## Building for Android

### Option 1: Build APK (for direct installation)

1. **Open terminal in the project directory:**
   ```bash
   cd /Volumes/Sarbajit/University/MAD/Mobile-Application-Design/Split/split
   ```

2. **Build Debug APK:**
   ```bash
   flutter build apk --debug
   ```
   The APK will be located at: `build/app/outputs/flutter-apk/app-debug.apk`

3. **Build Release APK (for production):**
   ```bash
   flutter build apk --release
   ```
   The APK will be located at: `build/app/outputs/flutter-apk/app-release.apk`

4. **Install on Android device:**
   - **Via USB:**
     ```bash
     flutter install
     ```
   - **Via ADB:**
     ```bash
     adb install build/app/outputs/flutter-apk/app-release.apk
     ```
   - **Via File Transfer:**
     - Copy the APK file to your Android device
     - Open the APK file on your device
     - Allow installation from unknown sources if prompted
     - Tap "Install"

### Option 2: Build App Bundle (for Google Play Store)

```bash
flutter build appbundle --release
```
The AAB file will be at: `build/app/outputs/bundle/release/app-release.aab`

---

## Building for iOS

### Step 1: Install Dependencies

1. **Install CocoaPods dependencies:**
   ```bash
   cd ios
   pod install
   cd ..
   ```

### Step 2: Configure Xcode

1. **Open the project in Xcode:**
   ```bash
   open ios/Runner.xcworkspace
   ```

2. **Configure Signing & Capabilities:**
   - Select "Runner" in the project navigator
   - Go to "Signing & Capabilities" tab
   - Select your Team (you may need to add your Apple ID)
   - Xcode will automatically create a provisioning profile

3. **Set Bundle Identifier:**
   - Make sure it matches your Firebase iOS app configuration
   - Current: `com.splitter.split`

### Step 3: Build and Install

#### Option A: Using Flutter CLI

1. **Connect your iPhone via USB**
2. **Trust the computer** on your iPhone if prompted
3. **Build and install:**
   ```bash
   flutter build ios --release
   flutter install
   ```

#### Option B: Using Xcode

1. **Select your device** from the device dropdown in Xcode
2. **Click the Play button** or press `Cmd + R` to build and run
3. **Trust the developer** on your iPhone:
   - Go to Settings → General → VPN & Device Management
   - Tap on your developer account
   - Tap "Trust"

---

## Running in Debug Mode (Development)

### Android:
```bash
flutter run
```
This will automatically detect connected Android devices and install the app.

### iOS:
```bash
flutter run
```
This will automatically detect connected iOS devices and install the app.

---

## Troubleshooting

### Android Issues:

1. **"cmdline-tools component is missing":**
   - Open Android Studio
   - Go to Tools → SDK Manager
   - Install "Android SDK Command-line Tools"

2. **"Android license status unknown":**
   ```bash
   flutter doctor --android-licenses
   ```
   Accept all licenses when prompted.

3. **Device not detected:**
   - Make sure USB debugging is enabled
   - Try different USB cable/port
   - Run: `adb devices` to check connection

### iOS Issues:

1. **"CocoaPods not installed":**
   ```bash
   sudo gem install cocoapods
   pod setup
   ```

2. **"Xcode installation is incomplete":**
   - Install Xcode from App Store
   - Run: `sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer`
   - Run: `sudo xcodebuild -runFirstLaunch`

3. **Signing errors:**
   - Make sure you're signed in with your Apple ID in Xcode
   - Select your team in Signing & Capabilities
   - Xcode will automatically manage certificates

4. **Build errors:**
   ```bash
   cd ios
   pod deintegrate
   pod install
   cd ..
   flutter clean
   flutter pub get
   ```

---

## Distribution

### Android - Google Play Store:

1. Build App Bundle:
   ```bash
   flutter build appbundle --release
   ```

2. Go to [Google Play Console](https://play.google.com/console)
3. Create a new app
4. Upload the AAB file
5. Complete store listing and submit for review

### iOS - App Store:

1. Build for release:
   ```bash
   flutter build ios --release
   ```

2. Open Xcode → Product → Archive
3. Distribute App → App Store Connect
4. Upload to App Store Connect
5. Complete app information in App Store Connect
6. Submit for review

### iOS - TestFlight (Beta Testing):

1. Archive the app in Xcode
2. Upload to App Store Connect
3. Add testers in TestFlight section
4. Testers can install via TestFlight app

---

## Quick Commands Reference

```bash
# Check connected devices
flutter devices

# Build Android APK
flutter build apk --release

# Build iOS
flutter build ios --release

# Run on connected device
flutter run

# Clean build files
flutter clean

# Get dependencies
flutter pub get

# Check for issues
flutter doctor -v
```

---

## Notes

- **Firebase Configuration**: Make sure `google-services.json` (Android) and `GoogleService-Info.plist` (iOS) are properly configured
- **Package Name**: Currently set to `com.splitter.split` - change if needed in:
  - Android: `android/app/build.gradle.kts`
  - iOS: `ios/Runner.xcodeproj/project.pbxproj`
- **Version**: Update in `pubspec.yaml` before each release

---

## Need Help?

If you encounter issues:
1. Run `flutter doctor -v` to see detailed information
2. Check Flutter documentation: [flutter.dev](https://flutter.dev)
3. Check Firebase setup: See `FIREBASE_SETUP.md` in project root

