# NOVA v16 Android — GitHub-ready source project

This directory is the repository root. The complete NOVA v16 web application and FFmpeg assets are bundled under `app/src/main/assets/www/`.

## Build APK with GitHub Actions

1. Extract `NOVA_V16_GITHUB_READY.zip` locally.
2. Upload **all contents inside the extracted folder** to the root of a GitHub repository. Do not upload the ZIP itself. Include the hidden `.github` folder.
3. Commit the upload. The `Build NOVA Android APK` workflow runs on pushes to `main` or `master`, or manually from Actions.
4. Open the latest workflow run. After a green check, download **Artifacts → NOVA-Android-APK** and extract the artifact ZIP to get the APK.

A branch build produces a debug-signed test APK. Stable update releases require a persistent keystore configured as the four GitHub Actions secrets documented in `README-BN.md`.

## Native video player integration

The Android app exposes separate **Open in VLC**, **Open in MX Player**, and **Other player…** actions. The first two send explicit Android `ACTION_VIEW` intents to the selected package using a FileProvider content URI and temporary read grant. The third opens the Android system chooser. `Save to phone` uses the Storage Access Framework.

- VLC: `org.videolan.vlc`
- MX Player Free: `com.mxtech.videoplayer.ad`
- MX Player Pro: `com.mxtech.videoplayer.pro`

## Project versions

- Android Gradle Plugin 8.6.1 / Gradle 8.7
- Java 17 / compile SDK 35 / min SDK 23
- AndroidX WebKit WebMessageListener bridge restricted to the bundled app-assets HTTPS origin and main frame.

## Validation status

The packaged source has been checked for inline JavaScript syntax, Android XML parsing, workflow YAML syntax, required bundled assets, native player button/bridge wiring, and ZIP integrity. A complete Android SDK/Gradle build and actual device playback tests have not yet been run in this environment. Codec playback depends on the installed player and the device hardware; universal 100% compatibility cannot be promised.
