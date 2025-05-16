## L3-DG-CUSTAP: Deployment Guide (DG) for CUSTAP  
NOTE: This revised version removes references to real-time push notifications to align with the L1-HLD statement that “No advanced real-time push notifications are implemented; the system relies on polling or manual sync.” All relevant push notification configuration steps have been removed or marked as deleted below.

This Deployment Guide explains how to package, sign, configure, and verify the Customer App (CUSTAP) for release to both staging and production environments. It aligns with the overall solution’s scale of ~400–600 concurrent system users and reflects the client clarifications on environment handling, code signing,  rollback strategy, and testing.

---

### 1. Introduction

CUSTAP is a React Native mobile application that enables customers to:  
- Register/Login via phone + OTP.  
- Submit new service requests and upload KYC documents.  
- View quotations, accept/reject them, and raise support tickets for executed leads.

Because this is a store-distributed app (Android Play Store, Apple App Store), the deployment process must handle code signing, environment variable setup,  and a basic verification routine in staging before production release.

---

### 2. Prerequisites

Before deploying CUSTAP, ensure the following prerequisites are in place:

1. **Development Environment**  
   - React Native CLI installed (if building locally).  
   - Node.js LTS version (e.g., 16.x or 18.x), npm or yarn.  
   - A Mac is required for iOS builds (Xcode installed).

2. **Mobile Platform Accounts**  
   - Apple Developer Account with relevant provisioning profiles for iOS.  
   - Google Play Developer Console access for uploading Android builds.

3. **Build & Signing Keys**  
   - Android keystore file (.jks) with associated passwords.  
   - iOS certificates (p12) or provisioning profiles.  
   - Secure location to store these assets (locally or in a private repository / CI secret store).

4. **CI/CD Pipeline Access**  
   - Azure DevOps or GitHub Actions configured to run build pipelines.  
   - Ability to store and reference secret files (keystore, provisioning profiles, signing keys).

5. **Environment Setup**  
   - Basic .env files for staging/production (see Section 3.1).  
   - Staging and production backend URLs, stored securely in the pipeline or as .env variables.



6. **Over-The-Air Hotfix Mechanism (Optional)**  
   - Microsoft CodePush account (if you plan to handle urgent JS-level patches).

---

### 3. Installation Steps

This section covers build-time configurations and procedures to create deployable binaries (APK/AAB for Android, IPA for iOS), as well as how to incorporate these steps into your CI/CD pipeline.

#### 3.1 Environment Configuration
1. Create two .env files:  
   - .env.staging – pointing to the staging backend URL (e.g., `API_URL=https://staging-api.example.com`).  
   - .env.production – pointing to the production backend URL (e.g., `API_URL=https://prod-api.example.com`).  
2. In your React Native project, use a library such as react-native-config (or similar) to load the correct .env file at build time depending on the build variant (staging vs. production).  
3. Keep sensitive values (any tokens, API secrets) out of version control; store them as pipeline variables or in a secure store, referencing them during build.

#### 3.2 Build & Code Signing Configuration
1. **Android Keystore**  
   - Place the keystore file in a secure folder (or retrieve it from your pipeline’s secure storage).  
   - Update gradle signingConfigs block in `android/app/build.gradle` to reference the keystore path, alias, and passwords.  
2. **iOS Certificates & Provisioning Profiles**  
   - On a Mac with Xcode, import the .p12 certificate and provisioning profiles.  
   - Alternatively, configure your CI pipeline to install the certificate/profiles during build (fetch from a secure artifact store or repository).  
   - Match the bundle identifier with the Apple Developer portal settings.  



#### 3.3 Building Locally for Testing
If you need a local staging build (e.g., for QA testing on physical devices):  
1. Switch your environment file to .env.staging.  
2. For iOS:  
   - `cd ios && pod install && cd ..`  
   - `npx react-native run-ios --configuration Staging` (adjust scheme as needed).  
3. For Android:  
   - `npx react-native run-android --variant=stagingDebug`

Confirm the app runs and points to your staging environment.

---

### 4. Configuration Steps

The following must be done after installing but before finalizing your deployment:

#### 4.1 Setting Environment Variables
1. Verify the correct .env file is selected (staging or production) in the pipeline.  
2. Map pipeline variables to the app’s environment if you prefer not to store them directly in .env files.  
3. Confirm “API_URL” or similar endpoints are correct by printing them in your CI logs (redacted if sensitive).

#### 4.2 Over-The-Air Update Setup (Optional)
1. If using CodePush for emergency JavaScript patches, link your app to CodePush via the AppCenter or CLI.  
2. Store the CodePush deployment keys in the pipeline’s secure store to prevent unauthorized usage.  
3. Only patch minor non-native changes after verifying on staging to avoid conflicts with major binary updates.

---

### 5. Verification Procedures

Because store deployment rollbacks are non-instant, verifying builds before release is essential:

#### 5.1 Smoke Tests in Staging
1. **Install the staged build** on test devices or emulators.  
2. **Login Test**: Confirm phone+OTP login with a test user.  
3. **New Lead Test**: Submit a lead (multi-service cart) and ensure it appears in your staging backend.  
4. **Quotation & Acceptance**: If a test quote is shared, attempt an accept or reject flow.  
5. **KYC Doc Upload**: Upload a small test image or PDF, verifying the correct “Pending Review” state.  
  
6. **Manual Sync Check**: Use any “Refresh” or in-app polling approach to confirm the data updates align with the server state when triggered.  
7. **Logout**: Ensure the session is invalidated and re-login is required.

#### 5.2 Basic End-to-End Check on Physical Devices
- Install the release-signed build (IPA for iOS, AAB or APK for Android) on real devices.  
- Repeat the essential flows to confirm no debug-only issues remain.  
- Address any discovered issues and re-run the build if necessary.

---

### 6. Rollback Procedures

Unlike servers, mobile apps can’t be instantly reverted once published to an app store. If a release has critical issues:

1. **Immediate Hotfix (JS Only)**  
   - Use CodePush to push a small JS patch that resolves the urgent bug, provided it doesn’t require native changes.  
   - Users with the installed version will receive an over-the-air update on next app launch.

2. **Emergency Re-Versioning**  
   - If the issue requires native changes or CodePush isn’t usable, increment the version number (e.g., 1.0.1 → 1.0.2), fix the issue, and publish a new build.  
   - Optionally, unpublish or halt further downloads of the problematic release in the app store (where feasible).  
   - Communicate to end-users that a new version is available and any older version might be unstable.

3. **Backend Mitigation**  
   - If a front-end bug can be mitigated by toggling a backend feature or temporarily disabling a problematic endpoint, do so while preparing the patched mobile release.

---

### 7. Deployment Scripts

Below is a simplified CI/CD pipeline outline for staging and production deployments, referencing Azure DevOps or GitHub Actions. Adjust naming and commands as needed for your environment.

#### 7.1 Example YAML for Staging Build

```yaml
name: CUSTAP Staging Build
on:
  push:
    branches: [ "develop" ]

jobs:
  build-staging:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Set up Node
        uses: actions/setup-node@v2
        with:
          node-version: '16'

      - name: Install Dependencies
        run: yarn install

      - name: Prepare Env
        run: |
          cp .env.staging .env

      - name: Build Android Staging
        run: cd android && ./gradlew assembleStagingRelease

      - name: Build iOS Staging
        run: |
          cd ios
          pod install
          xcodebuild -workspace MyApp.xcworkspace \
           -scheme MyAppStaging \
           -configuration Release \
           -sdk iphoneos \
           -archivePath $PWD/build/MyApp.xcarchive archive
        # Additional signing steps as required

      - name: Upload Artifacts
        # e.g., store .apk, .aab, .ipa for QA testers
        uses: actions/upload-artifact@v2
        with:
          name: staging-artifacts
          path: |
            android/app/build/outputs/apk/stagingRelease/*.apk
            ios/build/MyApp.xcarchive
```

#### 7.2 Production Build & Deployment

1. Similar steps as staging, but use `.env.production` and the “production” signing configs or provisioning profiles.  
2. Upload the final artifacts (AAB/IPA) to the respective stores via CLI tools or manually.

#### 7.3 Store Submission
1. **Google Play**  
   - Use Gradle Play Publisher or manual Play Console upload.  
2. **Apple App Store**  
   - Use Fastlane deliver or Xcode’s “Archive & Distribute” process.

Track the release in each store’s console, verifying that the status is “In Review” then “Approved/Live.”

---

### Final Remarks

  
This deployment guide ensures CUSTAP is delivered confidently to staging and production at the current ~400–600 concurrent user scale by managing environment variables, signing keys, performing smoke tests in staging, and preparing a rollback strategy through CodePush or emergency re-versioning. In alignment with L1-HLD, any notifications or lead status updates are handled via polling/manual refresh rather than real-time push.

Deployment success hinges on:  
1. Correct environment variable setup.  
2. Valid code signing keys and provisioning profiles.  
3. Thorough smoke tests in staging.  
4. Well-documented rollback steps.



By following the above procedures, teams can reliably build, release, and maintain the CUSTAP mobile app in alignment with the overall Solarium Green Energy platform’s poll-based update approach.