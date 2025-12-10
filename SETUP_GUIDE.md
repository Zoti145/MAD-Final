# Complete Setup Guide - Split Expense App

This guide will walk you through setting up the Split Expense app from scratch, including Firebase configuration and database setup.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Clone the Repository](#clone-the-repository)
3. [Firebase Project Setup](#firebase-project-setup)
4. [Android Configuration](#android-configuration)
5. [iOS Configuration (Optional)](#ios-configuration-optional)
6. [Install Dependencies](#install-dependencies)
7. [Configure Firebase in Code](#configure-firebase-in-code)
8. [Run the App](#run-the-app)
9. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before you begin, ensure you have the following installed:

### Required Software
- **Flutter SDK** (version 3.0 or higher)
  - Download from: https://flutter.dev/docs/get-started/install
  - Verify installation: `flutter doctor`
- **Android Studio** (for Android development)
  - Download from: https://developer.android.com/studio
  - Install Android SDK and Android SDK Platform-Tools
- **Xcode** (for iOS development - macOS only)
  - Download from: Mac App Store
  - Install Command Line Tools: `xcode-select --install`
- **Git** (for cloning the repository)
  - Download from: https://git-scm.com/downloads
- **VS Code or Android Studio** (recommended IDEs)
  - VS Code: https://code.visualstudio.com/
  - Android Studio: https://developer.android.com/studio

### Required Accounts
- **Google Account** (for Firebase)
- **Firebase Account** (free tier is sufficient)

---

## Clone the Repository

1. **Open Terminal/Command Prompt**

2. **Navigate to your desired directory**
   ```bash
   cd ~/Projects  # or wherever you want to store the project
   ```

3. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd Split/split
   ```

   Replace `<repository-url>` with your actual repository URL.

---

## Firebase Project Setup

### Step 1: Create Firebase Project

1. **Go to Firebase Console**
   - Visit: https://console.firebase.google.com/
   - Sign in with your Google account

2. **Create a New Project**
   - Click **"Add project"** or **"Create a project"**
   - Enter project name: `split-expense` (or your preferred name)
   - Click **Continue**
   - Disable Google Analytics (optional, can enable later)
   - Click **Create project**
   - Wait for project creation (30-60 seconds)
   - Click **Continue**

### Step 2: Enable Authentication

1. **Navigate to Authentication**
   - In Firebase Console, click **Authentication** in the left sidebar
   - Click **Get started**

2. **Enable Email/Password Sign-in**
   - Click on **Sign-in method** tab
   - Click on **Email/Password**
   - Toggle **Enable** to ON
   - Click **Save**

3. **Enable Google Sign-in**
   - Still in **Sign-in method** tab
   - Click on **Google**
   - Toggle **Enable** to ON
   - Enter a **Support email** (your email address)
   - Click **Save**

### Step 3: Create Firestore Database

1. **Navigate to Firestore Database**
   - In Firebase Console, click **Firestore Database** in the left sidebar
   - Click **Create database**

2. **Choose Security Rules**
   - Select **Start in test mode** (for development)
   - Click **Next**

3. **Choose Location**
   - Select a location closest to you (e.g., `us-central`, `asia-south1`)
   - Click **Enable**
   - Wait for database creation (30-60 seconds)

4. **Update Security Rules** (Important!)
   - Click on **Rules** tab
   - Replace the default rules with:
   ```javascript
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       // Users can only read/write their own user document
       match /users/{userId} {
         allow read: if request.auth != null;
         allow write: if request.auth != null && request.auth.uid == userId;
       }
       
       // Groups: members can read, only creator can write
       match /groups/{groupId} {
         allow read: if request.auth != null && 
           (request.auth.uid in resource.data.memberIds || 
            request.auth.uid == resource.data.createdBy);
         allow create: if request.auth != null && 
           request.auth.uid == request.resource.data.createdBy;
         allow update, delete: if request.auth != null && 
           request.auth.uid == resource.data.createdBy;
       }
       
       // Expenses: group members can read/write
       match /expenses/{expenseId} {
         allow read: if request.auth != null;
         allow create: if request.auth != null;
         allow update, delete: if request.auth != null && 
           request.auth.uid == resource.data.payerId;
       }
       
       // Meals: group members can read/write
       match /meals/{mealId} {
         allow read: if request.auth != null;
         allow create: if request.auth != null;
         allow update, delete: if request.auth != null && 
           request.auth.uid == resource.data.userId;
       }
     }
   }
   ```
   - Click **Publish**

### Step 4: Enable Firebase Storage (for receipt uploads)

1. **Navigate to Storage**
   - Click **Storage** in the left sidebar
   - Click **Get started**

2. **Set Security Rules**
   - Select **Start in test mode**
   - Click **Next**
   - Click **Done**

3. **Update Storage Rules**
   - Click on **Rules** tab
   - Replace with:
   ```javascript
   rules_version = '2';
   service firebase.storage {
     match /b/{bucket}/o {
       match /receipts/{allPaths=**} {
         allow read: if request.auth != null;
         allow write: if request.auth != null && 
           request.resource.size < 5 * 1024 * 1024; // 5MB limit
       }
     }
   }
   ```
   - Click **Publish**

---

## Android Configuration

### Step 1: Register Android App in Firebase

1. **Go to Project Settings**
   - In Firebase Console, click the **gear icon** (⚙️) next to "Project Overview"
   - Click **Project settings**

2. **Add Android App**
   - Scroll down to **"Your apps"** section
   - Click **Android icon** (or **"Add app"** → **Android**)

3. **Register App**
   - **Android package name**: `com.splitter.split`
   - **App nickname** (optional): `Split Android`
   - **Debug signing certificate SHA-1** (optional for now)
   - Click **Register app**

4. **Download google-services.json**
   - Click **Download google-services.json**
   - **IMPORTANT**: Save this file - you'll need it in the next step

### Step 2: Add SHA-1 Fingerprint (for Google Sign-In)

1. **Get Debug SHA-1 Fingerprint**
   - Open Terminal/Command Prompt
   - Run:
   ```bash
   # On macOS/Linux
   keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android | grep SHA1
   
   # On Windows
   keytool -list -v -keystore %USERPROFILE%\.android\debug.keystore -alias androiddebugkey -storepass android -keypass android | findstr SHA1
   ```

2. **Copy the SHA-1 fingerprint** (format: `XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX`)

3. **Add SHA-1 to Firebase**
   - Go back to Firebase Console → Project Settings
   - Scroll to **"Your apps"** → Find your Android app
   - Click **"Add fingerprint"**
   - Paste your SHA-1 fingerprint
   - Click **Save**

4. **Download Updated google-services.json**
   - Click **Download google-services.json** again
   - Replace the old file with the new one

### Step 3: Place google-services.json

1. **Navigate to Android app directory**
   ```bash
   cd android/app
   ```

2. **Replace google-services.json**
   - Delete the existing `google-services.json` (if any)
   - Copy the downloaded `google-services.json` to this location:
   ```
   android/app/google-services.json
   ```

3. **Verify the file**
   - Open `google-services.json`
   - Check that `package_name` is `com.splitter.split`
   - Check that `project_id` matches your Firebase project

---

## iOS Configuration (Optional - for iOS development)

### Step 1: Register iOS App in Firebase

1. **Go to Project Settings**
   - In Firebase Console → Project Settings

2. **Add iOS App**
   - Click **iOS icon** (or **"Add app"** → **iOS**)

3. **Register App**
   - **iOS bundle ID**: `com.splitter.split`
   - **App nickname** (optional): `Split iOS`
   - Click **Register app**

4. **Download GoogleService-Info.plist**
   - Click **Download GoogleService-Info.plist**
   - Save this file

### Step 2: Place GoogleService-Info.plist

1. **Open Xcode**
   ```bash
   open ios/Runner.xcworkspace
   ```

2. **Add the plist file**
   - In Xcode, right-click on **Runner** folder
   - Select **"Add Files to Runner"**
   - Select the downloaded `GoogleService-Info.plist`
   - Make sure **"Copy items if needed"** is checked
   - Click **Add**

3. **Verify the file**
   - The file should appear in `ios/Runner/GoogleService-Info.plist`

---

## Install Dependencies

1. **Navigate to project root**
   ```bash
   cd /path/to/Split/split
   ```

2. **Get Flutter dependencies**
   ```bash
   flutter pub get
   ```

3. **Verify installation**
   ```bash
   flutter doctor
   ```
   - Fix any issues shown (especially Android/iOS toolchain)

---

## Configure Firebase in Code

### Step 1: Update firebase_options.dart

The `firebase_options.dart` file should be auto-generated, but if it's missing or incorrect:

1. **Install FlutterFire CLI** (if not already installed)
   ```bash
   dart pub global activate flutterfire_cli
   ```

2. **Configure Firebase**
   ```bash
   flutterfire configure
   ```
   - Select your Firebase project
   - Select platforms: Android, iOS (if needed)
   - The CLI will automatically update `lib/firebase_options.dart`

### Step 2: Verify firebase_options.dart

1. **Open** `lib/firebase_options.dart`

2. **Check that it contains:**
   - Your Firebase project ID
   - Correct API keys
   - Correct app IDs

3. **If the file is missing or incorrect:**
   - Run `flutterfire configure` again
   - Or manually update with values from Firebase Console

---

## Run the App

### For Android

1. **Connect Android Device or Start Emulator**
   ```bash
   # List available devices
   flutter devices
   
   # Start Android emulator (if using)
   # Open Android Studio → AVD Manager → Start emulator
   ```

2. **Run the app**
   ```bash
   flutter run
   ```

3. **Or build APK**
   ```bash
   flutter build apk
   # APK will be in: build/app/outputs/flutter-apk/app-release.apk
   ```

### For iOS (macOS only)

1. **Open Xcode**
   ```bash
   open ios/Runner.xcworkspace
   ```

2. **Configure Signing**
   - Select **Runner** in project navigator
   - Go to **Signing & Capabilities**
   - Select your **Team**
   - Xcode will automatically manage provisioning

3. **Run the app**
   ```bash
   flutter run
   ```

   Or run from Xcode:
   - Select a simulator or connected device
   - Click **Run** button (▶️)

---

## Database Structure

The app uses Firestore with the following collections:

### Collections

1. **users**
   - Document ID: User UID
   - Fields: `id`, `email`, `displayName`, `avatarUrl`

2. **groups**
   - Document ID: Auto-generated
   - Fields: `id`, `name`, `description`, `type`, `memberIds[]`, `createdBy`, `createdAt`, `isClosed`, `closedAt`

3. **expenses**
   - Document ID: Auto-generated
   - Fields: `id`, `groupId`, `payerId`, `amount`, `description`, `date`, `receiptUrl`, `splitType`, `splitDetails{}`

4. **meals**
   - Document ID: Auto-generated
   - Fields: `id`, `groupId`, `userId`, `mealType`, `date`

### Indexes Required

Firestore may require composite indexes. If you see errors about missing indexes:

1. **Check Firebase Console**
   - Go to Firestore → Indexes
   - Click on any error links to create indexes automatically

2. **Common Indexes Needed:**
   - `meals`: `groupId` + `date` (descending)
   - `expenses`: `groupId` + `date` (descending)

---

## Troubleshooting

### Issue: "No matching client found for package name"

**Solution:**
- Verify `google-services.json` is in `android/app/`
- Check `package_name` in `google-services.json` matches `applicationId` in `android/app/build.gradle.kts`
- Rebuild: `flutter clean && flutter pub get && flutter run`

### Issue: Google Sign-In not working

**Solution:**
1. Verify SHA-1 fingerprint is added to Firebase Console
2. Check Google Sign-In is enabled in Firebase Authentication
3. Download updated `google-services.json`
4. Rebuild the app
5. Wait 2-5 minutes after Firebase changes

### Issue: "FirebaseApp not initialized"

**Solution:**
- Check `firebase_options.dart` exists and has correct values
- Verify Firebase is initialized in `lib/main.dart`
- Run `flutterfire configure` to regenerate options

### Issue: Firestore permission denied

**Solution:**
- Check Firestore security rules are published
- Verify user is authenticated
- Check rules allow the operation you're trying to perform

### Issue: Build errors

**Solution:**
```bash
# Clean build
flutter clean
cd android
./gradlew clean
cd ..
flutter pub get
flutter run
```

### Issue: Dependencies not found

**Solution:**
```bash
flutter pub get
flutter pub upgrade
```

### Issue: Android build fails

**Solution:**
1. Check Java version (should be 17)
2. Verify `android/build.gradle.kts` has correct Java version
3. Update Android SDK:
   ```bash
   # In Android Studio: Tools → SDK Manager → Update SDK
   ```

---

## Quick Start Checklist

- [ ] Flutter SDK installed (`flutter doctor` passes)
- [ ] Repository cloned
- [ ] Firebase project created
- [ ] Authentication enabled (Email/Password + Google)
- [ ] Firestore database created with security rules
- [ ] Storage enabled with security rules
- [ ] Android app registered in Firebase
- [ ] `google-services.json` placed in `android/app/`
- [ ] SHA-1 fingerprint added to Firebase
- [ ] iOS app registered (if developing for iOS)
- [ ] `GoogleService-Info.plist` added to Xcode (if iOS)
- [ ] Dependencies installed (`flutter pub get`)
- [ ] `firebase_options.dart` configured
- [ ] App runs successfully

---

## Additional Resources

- **Flutter Documentation**: https://flutter.dev/docs
- **Firebase Documentation**: https://firebase.google.com/docs
- **Firestore Security Rules**: https://firebase.google.com/docs/firestore/security/get-started
- **FlutterFire CLI**: https://firebase.flutter.dev/docs/cli

---

## Support

If you encounter issues not covered in this guide:

1. Check the [Troubleshooting](#troubleshooting) section
2. Review Firebase Console for error messages
3. Check Flutter logs: `flutter run -v` (verbose mode)
4. Review Firebase logs in Console → Logs

---

**Last Updated**: 2024
**App Version**: 1.0.0

