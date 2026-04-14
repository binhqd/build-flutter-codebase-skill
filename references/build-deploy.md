# Build & Deploy — Flutter

## Contents

- Flavors (dev / staging / prod)
- Android build configuration
- iOS build configuration
- Build scripts (Makefile)
- CI/CD pipeline
- App signing
- Store deployment checklist

## Flavors [LOW]

Three environments: `dev`, `staging`, `prod`. Each has its own:
- API base URL
- App ID suffix (e.g., `.dev`, `.staging`)
- App name suffix (e.g., "(Dev)", "(Staging)")
- Icon badge (optional)

### Dart Entry Points

```
lib/
├── main_dev.dart
├── main_staging.dart
└── main_prod.dart
```

Each entry point initializes the environment config and calls `bootstrap()`.

### Android Flavors (`android/app/build.gradle`)

```groovy
android {
    // ...

    flavorDimensions "environment"

    productFlavors {
        dev {
            dimension "environment"
            applicationIdSuffix ".dev"
            resValue "string", "app_name", "MyApp (Dev)"
            versionNameSuffix "-dev"
        }
        staging {
            dimension "environment"
            applicationIdSuffix ".staging"
            resValue "string", "app_name", "MyApp (Staging)"
            versionNameSuffix "-staging"
        }
        prod {
            dimension "environment"
            resValue "string", "app_name", "MyApp"
        }
    }
}
```

**AndroidManifest.xml** — use the flavor-defined app name:
```xml
<application android:label="@string/app_name" ...>
```

### iOS Flavors (Xcode Schemes + xcconfig)

Create 3 xcconfig files:

```
ios/Flutter/
├── Dev.xcconfig
├── Staging.xcconfig
└── Prod.xcconfig
```

**Dev.xcconfig:**
```
#include "Debug.xcconfig"
PRODUCT_BUNDLE_IDENTIFIER = com.example.myapp.dev
PRODUCT_NAME = MyApp (Dev)
DART_DEFINES = flutter.inspector.structuredErrors=true
```

**Staging.xcconfig:**
```
#include "Release.xcconfig"
PRODUCT_BUNDLE_IDENTIFIER = com.example.myapp.staging
PRODUCT_NAME = MyApp (Staging)
```

**Prod.xcconfig:**
```
#include "Release.xcconfig"
PRODUCT_BUNDLE_IDENTIFIER = com.example.myapp
PRODUCT_NAME = MyApp
```

Create 3 Xcode schemes (Dev, Staging, Prod), each pointing to the corresponding xcconfig.

### Flutter Run Commands

```bash
# Development
flutter run --flavor dev -t lib/main_dev.dart

# Staging
flutter run --flavor staging -t lib/main_staging.dart

# Production
flutter run --flavor prod -t lib/main_prod.dart
```

## Android Build Configuration

### Signing Config (`android/app/build.gradle`)

```groovy
android {
    signingConfigs {
        debug {
            // Default debug keystore
        }
        release {
            storeFile file(System.getenv("ANDROID_KEYSTORE_PATH") ?: "../keystore/release.jks")
            storePassword System.getenv("ANDROID_KEYSTORE_PASSWORD") ?: ""
            keyAlias System.getenv("ANDROID_KEY_ALIAS") ?: ""
            keyPassword System.getenv("ANDROID_KEY_PASSWORD") ?: ""
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### Generate Keystore

```bash
keytool -genkey -v \
  -keystore keystore/release.jks \
  -keyalg RSA -keysize 2048 \
  -validity 10000 \
  -alias release
```

**NEVER commit keystores to git.** Add to `.gitignore`:
```
keystore/
*.jks
*.keystore
```

## iOS Build Configuration

### Export Options Plist

```xml
<!-- ios/exportOptions.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>app-store</string>
    <key>uploadSymbols</key>
    <true/>
    <key>compileBitcode</key>
    <false/>
</dict>
</plist>
```

## Build Scripts (Makefile) [LOW]

```makefile
.PHONY: run-dev run-staging run-prod build-apk build-ios build-aab test analyze clean gen

# ─── Run ───────────────────────────────────────────────
run-dev:
	flutter run --flavor dev -t lib/main_dev.dart

run-staging:
	flutter run --flavor staging -t lib/main_staging.dart

run-prod:
	flutter run --flavor prod -t lib/main_prod.dart

# ─── Build Android ─────────────────────────────────────
build-apk-dev:
	flutter build apk --flavor dev -t lib/main_dev.dart

build-apk-staging:
	flutter build apk --flavor staging -t lib/main_staging.dart

build-apk-prod:
	flutter build apk --flavor prod -t lib/main_prod.dart --obfuscate --split-debug-info=build/debug-info

build-aab-prod:
	flutter build appbundle --flavor prod -t lib/main_prod.dart --obfuscate --split-debug-info=build/debug-info

# ─── Build iOS ─────────────────────────────────────────
build-ios-dev:
	flutter build ios --flavor dev -t lib/main_dev.dart

build-ios-staging:
	flutter build ios --flavor staging -t lib/main_staging.dart

build-ios-prod:
	flutter build ios --flavor prod -t lib/main_prod.dart --obfuscate --split-debug-info=build/debug-info

build-ipa-prod:
	flutter build ipa --flavor prod -t lib/main_prod.dart --export-options-plist=ios/exportOptions.plist

# ─── Test ──────────────────────────────────────────────
test:
	flutter test --coverage

test-unit:
	flutter test test/

test-integration:
	flutter test integration_test/

# ─── Quality ───────────────────────────────────────────
analyze:
	flutter analyze

format:
	dart format lib/ test/ --set-exit-if-changed

format-fix:
	dart format lib/ test/

# ─── Code Generation ──────────────────────────────────
gen:
	dart run build_runner build --delete-conflicting-outputs

gen-watch:
	dart run build_runner watch --delete-conflicting-outputs

# ─── Clean ─────────────────────────────────────────────
clean:
	flutter clean
	rm -rf coverage/
	rm -rf build/

# ─── Dependencies ─────────────────────────────────────
deps:
	flutter pub get

deps-upgrade:
	flutter pub upgrade --major-versions
```

## CI/CD Pipeline [MEDIUM]

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true

      - name: Install dependencies
        run: flutter pub get

      - name: Run code generation
        run: dart run build_runner build --delete-conflicting-outputs

      - name: Analyze
        run: flutter analyze

      - name: Format check
        run: dart format lib/ test/ --set-exit-if-changed

      - name: Run tests
        run: flutter test --coverage

      - name: Check coverage
        run: |
          sudo apt-get install -y lcov
          COVERAGE=$(lcov --summary coverage/lcov.info 2>&1 | grep 'lines' | grep -oP '\d+\.\d+')
          echo "Coverage: $COVERAGE%"
          if (( $(echo "$COVERAGE < 90" | bc -l) )); then
            echo "::error::Coverage $COVERAGE% is below 90% threshold"
            exit 1
          fi

  build-android:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - run: flutter pub get
      - run: dart run build_runner build --delete-conflicting-outputs
      - run: flutter build apk --flavor prod -t lib/main_prod.dart

      - uses: actions/upload-artifact@v4
        with:
          name: apk-prod
          path: build/app/outputs/flutter-apk/

  build-ios:
    needs: test
    runs-on: macos-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true

      - run: flutter pub get
      - run: dart run build_runner build --delete-conflicting-outputs
      - run: flutter build ios --flavor prod -t lib/main_prod.dart --no-codesign
```

## Store Deployment Checklist

### Before Submitting to Play Store / App Store

- [ ] App icons generated for all sizes (use flutter_launcher_icons)
- [ ] Splash screen configured (use flutter_native_splash)
- [ ] Version number and build number updated in `pubspec.yaml`
- [ ] Release notes written
- [ ] ProGuard rules configured (Android)
- [ ] App signing configured
- [ ] All flavors build successfully
- [ ] Integration tests pass on real device
- [ ] Performance profiled (no jank, no memory leaks)
- [ ] Accessibility checked (TalkBack / VoiceOver)
- [ ] Privacy policy URL ready
- [ ] Screenshots prepared for store listing
