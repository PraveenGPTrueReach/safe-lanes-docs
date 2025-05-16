``
## L3-DG-CPAPP: Deployment Guide (DG) for CPAPP

### 1. Introduction
This document provides a step-by-step reference for deploying the Channel Partner App (CPAPP). It is tailored to the current project scale of approximately 400–600 concurrent users, following the pragmatic approaches outlined in higher-level documents (L1-HLD, L1-KD). The goal is to ensure a stable, repeatable deployment process without unnecessary complexity.

This guide covers:
- Prerequisites for deployment
- Installation and environment setup steps
- Configuration instructions (environment variables, push credentials)
- Verification procedures
- Rollback options
- Overview of the CI/CD pipeline and any utility scripts

### 2. Prerequisites
Before deploying CPAPP, ensure the following prerequisites are met:

1. Hardware/OS:
   - macOS (for iOS builds) with Xcode installed, or
   - Windows/Linux machine (for Android builds only) with Android SDK installed.
   - Sufficient CPU/RAM for building large React Native apps (e.g., 8GB+ RAM).

2. Development Tools:
   - Node.js (LTS version, e.g., 16.x or 18.x) and npm or yarn.
   - Watchman (recommended for React Native on macOS).
   - Xcode (for iOS), Android Studio or Android SDK CLI tools (for Android).

3. Credentials & Certificates:
   - Apple Developer Account (for iOS App Store/TestFlight distributions).
   - Google Play Developer Account (for Android release distributions).
   - Keystore file for Android signing (securely stored).
   - Provisioning profiles and certificates for iOS signing.

4. Git Repository Access:
   - Credentials to clone/pull the CPAPP code repository.
   - Sufficient privileges to commit changes if updates are needed.

5. Environment Variables & Files:
   - .env.staging and .env.production files with relevant API endpoints, push notification keys, and other secrets (securely managed).
   - Knowledge of how to handle separate push credentials for staging vs. production.

6. CI/CD Setup (If applicable):
   - A configured pipeline (e.g., GitHub Actions, Azure DevOps) that can run build/test jobs and optionally produce app artifacts for staging and production.
   - Access to any vault or secret store (if used) for environment secrets.

### 3. Installation Steps

#### 3.1 Obtain Source Code
1. Clone or pull the CPAPP repository from the approved Git host:
   ```
   git clone <CPAPP-repo-URL>
   cd CPAPP
   ```

2. Check out the correct branch for your deployment target:
   - “staging” branch (for test/QA builds).
   - “production” branch (for final releases).

#### 3.2 Install Dependencies
From the project’s root directory:
```
npm install
```
(or use yarn: `yarn install`)

#### 3.3 Project Structure Overview
Typical directories include:
- /android – Native Android project structure.  
- /ios – Native iOS project structure.  
- /src – Main React Native/TypeScript application code.  
- /env – (optional) location holding environment-specific .env files.  
- /scripts – possible utility scripts for bundling or distribution.

#### 3.4 Android Build (Local or CI)
1. Copy the correct .env file:
   - For staging: rename or link “.env.staging” → “.env”  
   - For production: rename or link “.env.production” → “.env”
2. Generate a release build:
   ```
   cd android
   ./gradlew assembleRelease
   ```
   - This references the Android keystore in the gradle config. If building locally, ensure the keystore is placed correctly (usually in android/app/), and the build.gradle keeps the correct signingConfigs.

3. The generated APK (or AAB) is usually located in:
   ```
   android/app/build/outputs/apk/release/app-release.apk
   ```
   or
   ```
   android/app/build/outputs/bundle/release/app-release.aab
   ```

4. Sign & Align (if not already performed automatically):
   - By default, Gradle reads the signing details from your gradle config. No separate step is needed if properly configured.

#### 3.5 iOS Build (Local or CI)
1. Copy the correct .env file:
   - For staging: rename/link “.env.staging” → “.env”  
   - For production: rename/link “.env.production” → “.env”
2. Install CocoaPods dependencies (inside the ios folder):
   ```
   cd ios
   pod install
   cd ..
   ```
3. Open the Xcode workspace:
   ```
   open ios/CPAPP.xcworkspace
   ```
4. Configure signing:
   - Select the correct team/provisioning profile in Xcode “Signing & Capabilities” for the scheme (staging or production).
5. Archive and export:
   - Xcode → Product → Archive → Distribute
   - For staging builds, use TestFlight or Ad Hoc distribution to share with testers.
   - For production, distribute to the App Store.

### 4. Configuration Steps

#### 4.1 Environment Variables (.env Files)
- CPAPP uses minimal environment config loaded at build time.  
- Typical keys:  
  - REACT_APP_BASE_URL: Points to the backend (staging or production).  
  - REACT_APP_PUSH_KEY: FCM key or other push credentials.  
  - REACT_APP_ENV: “staging” or “production” (optional).  

Example .env.production:
```
REACT_APP_BASE_URL=https://api.prod.solarium.com
REACT_APP_PUSH_KEY=PROD-FCM-KEY
REACT_APP_ENV=production
```

Example .env.staging:
```
REACT_APP_BASE_URL=https://api.staging.solarium.com
REACT_APP_PUSH_KEY=STAGING-FCM-KEY
REACT_APP_ENV=staging
```
Use a secure method to store and retrieve these files (e.g., CI environment variables or an encrypted store).

#### 4.2 Push Notification Credentials
- Maintain separate GCM/FCM (Android) and APNs (iOS) credentials for staging vs. production to avoid cross-environment misrouting.  
- Ensure the correct “Google Services” config files are placed in the android/app/ and iOS target as needed:
  - google-services.json (Android)
  - GoogleService-Info.plist (iOS)

#### 4.3 Code-Signing and Certificates
- For Android:  
  - Keep the release keystore (e.g., `my-release-key.keystore`) in a secure location.  
  - Reference it in `android/app/build.gradle` via the signingConfigs block.
- For iOS:
  - Use the correct Provisioning Profiles and certificates from the Apple Developer Portal.  
  - If building locally, store them in Xcode credentials. For CI, store them in a vault or secure pipeline secrets.

#### 4.4 Over-the-Air (OTA) Updates
- If CodePush (or similar) is enabled for minor hotfixes, ensure staging/production CodePush deployment keys are specified in the .env or build config.  
- Validate that a fallback is in place: if an OTA push fails in production, revert the CodePush update to prevent widespread disruption.

### 5. Verification Procedures

#### 5.1 Manual Functional Checks
Given the moderate user count (~400–600 concurrent), a concise manual checklist is typically sufficient:

1. Launch app (staging build) on a device/emulator.  
2. Perform login with a known test account.  
3. Create a new lead, verifying it appears in the “My Leads” section.  
4. Generate a test quotation; check that the final PDF or share action completes.  
5. Upload a document (e.g., KYC or lead doc). Confirm the file can be viewed in the lead’s detail page.  
6. Switch to offline mode (disable network) and confirm the app shows data read-only, blocks new leads or changes.  
7. Re-enable network and confirm the lead list syncs properly.  
8. (Optional) Test push notifications using staging push credentials.

#### 5.2 Additional Smoke Tests
- If desired, implement an automated test suite with Detox/Appium that covers basic flows (login, lead creation, doc upload).  
- Confirm the build runs without red screen errors on both iOS and Android.

### 6. Rollback Procedures

#### 6.1 Store-Based Rollback
- If a new version in the store is defective, issue an immediate hotfix release to patch the defect.  
- Alternatively, unpublish the broken version from the Google Play Console and re-publish the previous stable version if still available.  
- For Apple’s App Store, a direct “revert” is not available. You must release a new build quickly with the fix (hotfix approach).

#### 6.2 Over-the-Air (CodePush) Revert
- If an OTA update delivered via CodePush causes critical errors, use the CodePush CLI or platform dashboard to roll back to the previous bundle immediately.

### 7. Deployment Scripts & CI/CD Pipeline Details

#### 7.1 CI/CD Pipeline Overview
1. Developer merges code into “staging” branch.  
2. CI pipeline triggers:
   - Installs dependencies (npm install)  
   - Runs lint/test checks  
   - Builds a staging APK (Android) or staging archive (iOS)  
   - Optionally publishes to TestFlight/Internal App Sharing for QA  
3. Developer or QA tests the staging app.  
4. Once approved, code is merged into “production” branch.  
5. Pipeline triggers a production build, signing with production keys.  
6. Final artifacts are uploaded to App Store Connect and Google Play Console for release.

#### 7.2 Example Commands

- Staging Build (Android):
  ```
  # in CI or local
  cp .env.staging .env
  npm install
  cd android
  ./gradlew assembleRelease
  ```
- Production Build (iOS):
  ```
  # in CI or local
  cp .env.production .env
  npm install
  cd ios
  pod install
  # Xcode command-line or manual archive
  xcodebuild -workspace CPAPP.xcworkspace \
     -scheme CPAPP \
     -configuration Release \
     -archivePath ./build/CPAPP.xcarchive archive
  ```

- CodePush Deployment (if applicable):
  ```
  appcenter codepush release-react -a MyOrg/MyCPApp-Android \
    -d Production \
    --description "Minor hotfix for CPAPP"
  ```

### 8. Conclusion
Following the above instructions ensures a consistent and reliable deployment of CPAPP for both staging and production. By maintaining separate environment files, push credentials, and signing configurations—as clarified in the client’s consolidated decisions—CPAPP can be safely tested and then rolled out to Channel Partners. Immediate hotfix releases, CodePush rollbacks, or a swift manual store reversion strategy provide adequate safeguards for the ~400–600 user load. All future changes to environment or scale should be revisited in alignment with solution-level architectural updates. 
```
