# NOVA v16 Android — সম্পূর্ণ GitHub Upload Project

এই ফোল্ডারটিই GitHub repository-র **root** হিসেবে ব্যবহার করবে। আলাদা করে কোড কপি-পেস্ট করতে হবে না। `app/src/main/assets/www/`-এর মধ্যে NOVA v16-এর সম্পূর্ণ ওয়েব অ্যাপ, FFmpeg JavaScript এবং WASM ফাইল রাখা আছে।

## GitHub-এ একবারে সব ফাইল আপলোড

1. `NOVA_V16_GITHUB_READY.zip` ডাউনলোড করে Windows-এ **Extract All** করো।
2. Extract করা `NOVA_V16_GITHUB_READY` ফোল্ডারটি খোলো। এই ফোল্ডারের ভেতরে সরাসরি `app`, `.github`, `build.gradle`, `settings.gradle`, `gradle.properties`, `README-BN.md` ইত্যাদি দেখা উচিত।
3. `.github` ফোল্ডারটি দেখা না গেলে File Explorer → **View → Show → Hidden items** চালু করো। `.github` খুব গুরুত্বপূর্ণ; এর ভেতরের workflow ছাড়া GitHub নিজে APK build শুরু করবে না।
4. GitHub repository-র **Add file → Upload files** পেজে ফিরে Extract করা ফোল্ডারের ভেতরের সব ফাইল ও ফোল্ডার একসঙ্গে drag-and-drop করো। ZIP ফাইলটি সরাসরি GitHub-এ upload করবে না; আগে Extract করতে হবে।
5. Upload তালিকায় `app/` এবং `.github/workflows/android-apk.yml` আছে কি না দেখে নিচে **Commit changes** চাপো।
6. Repository-র **Actions** ট্যাবে গিয়ে **Build NOVA Android APK** workflow খুলে সর্বশেষ run দেখো। প্রথম run-এ কয়েক মিনিট লাগতে পারে।
7. সবুজ check হলে run খুলে **Artifacts → NOVA-Android-APK** থেকে APK download করো। Artifact ZIP Extract করলে APK পাবে।

## APK-তে Player অপশন

NOVA → Watching → Episode → Play Video-তে আলাদা অপশন থাকবে:

- **Open in VLC** — VLC Android package `org.videolan.vlc`-এ সরাসরি ACTION_VIEW Intent পাঠায়।
- **Open in MX Player** — MX Player Free (`com.mxtech.videoplayer.ad`) অথবা Pro (`com.mxtech.videoplayer.pro`) ইনস্টল থাকলে সেটি খোলে।
- **Other player…** — Android-এর system chooser দেখায়।
- **Save to phone** — Android-এর Save dialog দিয়ে NOVA private storage থেকে নির্বাচিত ফোল্ডারে ভিডিও কপি করে।

নির্বাচিত player ইনস্টল না থাকলে Play Store খোলার prompt আসে। Player intent-এ `FileProvider` content URI, read permission এবং ClipData grant ব্যবহার করা হয়েছে; NOVA-র private filesystem path অন্য অ্যাপে প্রকাশ করা হয় না।

## ভিডিও সংরক্ষণ

Android-এর native file picker থেকে ভিডিও নির্বাচন করলে NOVA ফাইলটি নিজের private app storage-এ 256 KB buffer দিয়ে stream করে কপি করে। কপি সম্পূর্ণ হওয়ার পরেই আগের কপি replace হয়। Gallery-র original video মুছে গেলেও NOVA-র copy থাকবে, যতক্ষণ NOVA app data clear বা uninstall না করা হয়। ফোনে যথেষ্ট খালি জায়গা থাকতে হবে।

## স্থায়ী APK update (ঐচ্ছিক, প্রথম release-এর আগে)

পরীক্ষার জন্য branch push-এ debug-signed APK তৈরি হয়; secrets না দিলেও build চলে। Debug APK-র signing key GitHub runner অনুযায়ী বদলাতে পারে, তাই নতুন debug APK পুরোনোটির ওপর update হিসেবে install নাও হতে পারে—প্রয়োজনে আগের test APK uninstall করে নতুনটি install করো।

দীর্ঘমেয়াদে update দিতে প্রথম release-এর আগে একটি release keystore তৈরি করে নিরাপদে backup রাখো:

```bash
keytool -genkeypair -v -keystore nova-release.jks -alias nova -keyalg RSA -keysize 2048 -validity 10000
```

PowerShell-এ Base64 তৈরি:

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("nova-release.jks")) | Set-Clipboard
```

GitHub → Settings → Secrets and variables → Actions-এ এই চারটি secret যোগ করো:

- `NOVA_KEYSTORE_BASE64`
- `NOVA_STORE_PASSWORD`
- `NOVA_KEY_ALIAS`
- `NOVA_KEY_PASSWORD`

Keystore বা password GitHub repository-তে upload করবে না। একই keystore হারিয়ে গেলে একই signature-এ update APK তৈরি করা যাবে না। Secrets যোগ করার পর `v1.0.0`-এর মতো tag push করলে APK GitHub Release-এও যুক্ত হবে।

## Project technical settings

- Android Gradle Plugin: `8.6.1`
- Gradle: `8.7` (GitHub Actions build runner-এ স্বয়ংক্রিয়ভাবে provision হয়)
- Java: `17`
- Compile SDK: `35`
- Minimum Android: `6.0` / API `23`
- WebView asset origin: `https://appassets.androidplatform.net`
- Native bridge: AndroidX WebKit `WebMessageListener`, main-frame এবং app-assets origin সীমাবদ্ধ

## কী পরীক্ষা করা হয়েছে / কী বাকি

এই প্যাকেজে HTML inline JavaScript syntax, Android XML parsing, workflow YAML parsing, embedded assets, VLC/MX button-to-bridge marker এবং ZIP integrity পরীক্ষা করা হয়েছে। এই কাজের পরিবেশে Android SDK/Gradle দিয়ে সম্পূর্ণ APK build বা বাস্তব ফোনে VLC/MX playback পরীক্ষা করা হয়নি। GitHub Actions-এর প্রথম build এবং ফোনে বাস্তব পরীক্ষা এখনও আবশ্যক। সব ভিডিও codec বা সব ফোনে 100% playback guarantee দেওয়া সম্ভব নয়; codec support player ও phone hardware-এর ওপর নির্ভর করে।
