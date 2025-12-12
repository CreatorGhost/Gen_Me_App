# Build and Release Guide for Gen_Me App

## Prerequisites

Due to Java 25 compatibility issues with Kotlin 1.9.25, you'll need to use a compatible Java version (Java 11-21) to build this project.

### Option 1: Build Locally (Recommended)
On your local machine with compatible Java (11-21):

```bash
# Navigate to the project directory
cd Gen_Me_App

# Build the release APK
./gradlew clean assembleRelease

# Build the debug APK (for testing)
./gradlew clean assembleDebug

# Create APK Bundle (for Play Store)
./gradlew clean bundleRelease
```

The built APK will be located at:
- **Release APK**: `app/build/outputs/apk/release/app-release-unsigned.apk`
- **Debug APK**: `app/build/outputs/apk/debug/app-debug.apk`
- **Bundle**: `app/build/outputs/bundle/release/app-release.aab`

### Option 2: Using Docker
If you prefer a containerized build:

```dockerfile
FROM ubuntu:22.04

# Install Java 17
RUN apt-get update && apt-get install -y openjdk-17-jdk android-sdk

# Copy project
COPY . /app
WORKDIR /app

# Build
RUN ./gradlew clean assembleRelease
```

## Signing the APK (Optional but Recommended)

To sign your release APK:

```bash
# Create a keystore (one-time)
keytool -genkey -v -keystore my-release-key.keystore -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias

# Sign the APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore app/build/outputs/apk/release/app-release-unsigned.apk my-key-alias

# Optimize the APK
zipalign -v 4 app/build/outputs/apk/release/app-release-unsigned.apk app/build/outputs/apk/release/app-release-signed.apk
```

## Publishing to GitHub Releases

### Method 1: Using GitHub Web UI
1. Go to your repository: https://github.com/Amartyapratapsingh/Gen_Me
2. Click on "Releases" in the sidebar
3. Click "Draft a new release"
4. Tag version (e.g., `v1.0.0`)
5. Add title and description
6. Drag and drop the APK file into the release
7. Publish

### Method 2: Using GitHub CLI
First, ensure you have a built APK file ready. Then:

```bash
# Create a new release with the APK
gh release create v1.0.0 ./app/build/outputs/apk/release/app-release-unsigned.apk \
  --title "Gen_Me App v1.0.0" \
  --notes "Release notes for v1.0.0"

# Add more files to existing release
gh release upload v1.0.0 ./path/to/apk
```

### Method 3: Using `git` Tag
```bash
# Create a local tag
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push the tag to remote
git push origin v1.0.0

# Then create release on GitHub Web UI using the tag
```

## Current Configuration

**API Base URL**: http://35.226.2.144/

The app is configured to connect to the backend at the above IP address. Update this in the following files if needed:
- `app/src/main/java/com/example/genme/repository/VirtualTryOnRepository.kt` (Line 31)
- `app/src/main/java/com/example/genme/repository/FigurineRepository.kt` (Line 28)
- `app/src/main/java/com/example/genme/repository/HairstyleRepository.kt` (Line 32)

## Troubleshooting

### Java Version Error
If you encounter a Java version error:
```
What went wrong:
25.0.1
```

This means your Java version is incompatible. Use Java 11-21 instead:
```bash
# Check Java version
java -version

# Switch Java version (if you have multiple installed)
update-alternatives --config java
```

### Build Hangs
If the build appears to hang, try:
```bash
./gradlew clean --stop
./gradlew clean assembleRelease
```

### Out of Memory
If you encounter memory errors:
```bash
export GRADLE_OPTS="-Xmx4096m"
./gradlew clean assembleRelease
```

## Version Info

- **Min SDK**: 27 (Android 8.1)
- **Target SDK**: 34 (Android 14)
- **Kotlin**: 1.9.25
- **AGP**: 8.0 (Android Gradle Plugin)
- **Java Target**: 1.8

