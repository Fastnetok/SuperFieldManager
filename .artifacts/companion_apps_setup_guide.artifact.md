# 📱 ساتھی ایپس (Companion Apps) کا ماسٹر سیٹ اپ اور مائیگریشن گائیڈ

یہ دستاویز تمام ساتھی ایپس (`SuperFieldManager`, `SuperDealer`, `SuperCustomerIDAap`) کو نئے پروجیکٹ یا کلون کے طور پر سیٹ کرنے، پیکیج آئی ڈی تبدیل کرنے، فائر بیس کنفیگر کرنے، سائننگ سیٹ اپ کرنے اور گٹ ہب آٹومیشن کے تمام مراحل کی جامع رہنمائی فراہم کرتی ہے۔

---

## 📋 فہرسـت (Table of Contents)
1. [اہم ترین پیشگی اصول (Golden Rules)](#1-اہم-ترین-پیشگی-اصول-golden-rules)
2. [اسٹیپ ۱: گٹ ریموٹ کی علیحدگی (Git Remote Separation)](#2-اسٹیپ-۱-گٹ-ریموٹ-کی-علیحدگی-git-remote-separation)
3. [اسٹیپ ۲: پیکیج آئی ڈی اور نیم اسپیس کی تبدیلی (Package ID & Namespace)](#3-اسٹیپ-۲-پیکیج-آئی-ڈی-اور-نیم-اسپیس-کی-تبدیلی-package-id--namespace)
4. [اسٹیپ ۳: فائر بیس اور فائر اسٹور انٹیگریشن (Firebase & Firestore)](#4-اسٹیپ-۳-فائر-بیس-اور-فائر-اسٹور-انٹیگریشن-firebase--firestore)
5. [اسٹیپ ۴: پرمیننٹ سائننگ کانفیگریشن (Permanent Signing Config)](#5-اسٹیپ-۴-پرمبیننٹ-سائننگ-کانفیگریشن-permanent-signing-config)
6. [اسٹیپ ۵: گٹ ہب سیکرٹس کی تشکیل (GitHub Secrets Configuration)](#6-اسٹیپ-۵-گٹ-ہب-سیکرٹس-کی-تشکیل-github-secrets-configuration)
7. [اسٹیپ ۶: ورژن چیکر اور آٹو اپ ڈیٹ (VersionChecker API)](#7-اسٹیپ-۶-ورژن-چیف-اور-آٹو-اپ-ڈیٹ-versionchecker-api)

---

## ۱. اہم ترین پیشگی اصول (Golden Rules)
1. **مکمل آئسولیشن:** ہر ایپ کی اپنی الگ پیکیج آئی ڈی، الگ فائر بیس پروجیکٹ اور الگ گٹ ہب ریپوزٹری ہونی چاہیے۔
2. **سائننگ مطابقت:** ڈیবাগ اور ریلیز دونوں بلڈ ٹائپس ایک ہی سائننگ کی سے سائن ہوں تاکہ "App not installed" کا مسئلہ کبھی نہ آئے۔

---

## ۲. اسٹیپ ۱: گٹ ریموٹ کی علیحدگی (Git Remote Separation)
جب آپ پروجیکٹ کاپی یا کلون کریں، تو پرانے گٹ ریموٹ کو ہٹا کر نئی ریپوزٹری سے لنک کریں:

1. اینڈرائیڈ اسٹوڈیو کے ٹرمینل یا پاور شیل میں پروجیکٹ فولڈر پر جائیں:
   ```powershell
   git remote remove origin
   ```
2. اپنی نئی گٹ ہب ریپوزٹری کا لنک جوڑیں:
   ```powershell
   git remote add origin https://github.com/Fastnetok/<NEW_REPO_NAME>.git
   ```
3. پہلا کمٹ اور پش کریں:
   ```powershell
   git add .
   git commit -m "Initial commit for companion app"
   git push -u origin master
   ```

---

## ۳. اسٹیپ ۲: پیکیج آئی ڈی اور نیم اسپیس کی تبدیلی (Package ID & Namespace)
ایپ کی شناخت بدلنے کے لیے `app/build.gradle.kts` میں تبدیلیاں کریں:

1. **`app/build.gradle.kts`** کھولیں:
   ```kotlin
   android {
       namespace = "com.example.superdealer" // نئی ایپ کا پیکیج
       ...
       defaultConfig {
           applicationId = "com.example.superdealer" // نئی ایپ کی پیکیج آئی ڈی
           ...
       }
   }
   ```
2. **Package & Imports:** کوٹلین سورس فائلز کے اندر پیکج کے نام اور امپورٹس کو نئے نام کے مطابق ری فیکٹر (Refactor) کریں۔

---

## ۴. اسٹیپ ۳: فائر بیس اور فائر اسٹور انٹیگریشن (Firebase & Firestore)
ہر ایپ کا اپنا ڈیٹا بیس ہونا لازمی ہے:

1. [Firebase Console](https://console.firebase.google.com/) پر جائیں۔
2. ایک نیا پروجیکٹ بنائیں یا پرانے میں **نئی اینڈرائیڈ ایپ** رجسٹر کریں (پیکیج نام وہی دیں جو اسٹیپ ۲ میں رکھا تھا)۔
3. **`google-services.json`** فائل ڈاؤن لوڈ کر کے پروجیکٹ کے **`app/`** فولڈر میں رکھ دیں۔
4. **Firestore Database** اور **Realtime Database** کے سیکیورٹی رولز (Rules) کو پبلک یا ٹیسต์ موڈ پر سیٹ کریں:
   ```json
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if true;
       }
     }
   }
   ```

---

## ۵. اسٹیپ ۴: پرمیننٹ سائننگ کانفیگریشن (Permanent Signing Config)
"App not installed" کے ایرر سے بچنے کے لیے `app/build.gradle.kts` میں یہ سیٹنگ لازمی رکھیں:

```kotlin
android {
    signingConfigs {
        create("release") {
            val isGitHub = System.getenv("GITHUB_ACTIONS") == "true"
            storeFile = if (isGitHub) {
                file("keystore.jks")
            } else {
                file("D:/AndroidKeys/EboneReleaseKey.jks")
            }
            storePassword = System.getenv("STORE_PASSWORD") ?: "aeiougabbas"
            keyAlias = System.getenv("KEY_ALIAS") ?: "ebone"
            keyPassword = System.getenv("KEY_PASSWORD") ?: "aeiougabbas"
        }
    }

    buildTypes {
        debug {
            signingConfig = signingConfigs.getByName("release")
        }
        release {
            signingConfig = signingConfigs.getByName("release")
            isMinifyEnabled = false
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
    }
}
```

---

## ۶. اسٹیپ ۵: گٹ ہب سیکرٹس کی تشکیل (GitHub Secrets Configuration)
گٹ ہب آٹومیشن کے ذریعے APK بلڈ کرنے کے لیے ریپوزٹری کی **Settings > Secrets and variables > Actions** میں یہ ۴ Secrets لازمی ایڈ کریں:

| Secret Name | Value |
| :--- | :--- |
| **`SIGNING_KEY`** | کی اسٹور فائل (`.jks`) کا مکمل Base64 انکوڈڈ اسٹرنگ |
| **`STORE_PASSWORD`** | `aeiougabbas` |
| **`KEY_ALIAS`** | `ebone` |
| **`KEY_PASSWORD`** | `aeiougabbas` |

---

## ۷. اسٹیپ ۶: ورژن چیکر اور آٹو اپ ڈیٹ (VersionChecker API)
`VersionChecker.kt` میں گٹ ہب API کا پاتھ نئی ریپوزٹری پر پوائنट کریں:

```kotlin
private const val GITHUB_API = "https://api.github.com/repos/Fastnetok/<NEW_REPO_NAME>/releases"
```
