name: Build Rankawat Dairy Farm APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build-apk:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Extract Android project
        shell: bash
        run: |
          set -e

          echo "Searching for Android source ZIP..."

          ZIP=$(find . -type f -path '*/source/*.zip' | head -n 1)

          if [ -z "$ZIP" ]; then
            echo "Source ZIP not found"
            find . -maxdepth 4 -type f
            exit 1
          fi

          echo "Found: $ZIP"

          mkdir -p extracted
          unzip -q "$ZIP" -d extracted

          ROOT=$(find extracted -type f -name settings.gradle -printf '%h\n' | head -n 1)

          if [ -z "$ROOT" ]; then
            echo "Android project not found after extraction"
            find extracted -maxdepth 5 -type f
            exit 1
          fi

          echo "Android project found at: $ROOT"

          cp -a "$ROOT"/. .

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "17"

      - name: Set up Android SDK
        uses: android-actions/setup-android@v3

      - name: Install Android SDK
        run: |
          yes | sdkmanager --licenses || true
          sdkmanager "platform-tools" \
            "platforms;android-35" \
            "build-tools;35.0.0"

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: "8.9"

      - name: Build APK
        run: |
          gradle assembleDebug --no-daemon

      - name: Rename APK
        run: |
          mkdir -p release
          cp app/build/outputs/apk/debug/app-debug.apk \
            release/Rankawat-Dairy-Farm.apk

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Rankawat-Dairy-Farm
          path: release/Rankawat-Dairy-Farm.apk
          if-no-files-found: error
