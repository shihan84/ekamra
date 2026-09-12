# Handoff Document — Ekamra OTT

**Last Updated:** Sep 12, 2026 6:25 PM IST  
**Latest Commit:** `98a112a` — fix: sign release APK with committed keystore, disable ProGuard minification  
**Repo:** https://github.com/shihan84/ekamra.git (branch: main)

---

## Project Overview

Ekamra OTT is a rebranded OTT/IPTV platform (originally OXOO by SpaGreen) consisting of:
1. **Flutter Mobile App** — `com.ekamra.ott` (Android + iOS)
2. **Android TV App** — `com.ekamra.tv` (native Java + Leanback)
3. **PHP Admin Panel** (CodeIgniter 3) — https://ekamraott.com/panel/
4. **TV Admin Panel** (CodeIgniter 3) — https://ekamraott.com/tv-panel/

---

## Current Status

### Mobile App (Flutter) ✅
- **Version:** 2.0.2+4 (pubspec.yaml)
- **App ID:** `com.ekamra.ott`
- **Flutter:** 3.32.8, Java 17, Gradle 8.14.3, compileSdk 36, minSdk 23, NDK 28.0.12674087
- **Keystore:** `android/keys/ekamra-mobile.jks` (committed, password: `123456`, alias: `key0`)
- **Signing:** Always-on signing config with env var fallbacks
- **ProGuard:** Disabled (`minifyEnabled false`, `shrinkResources false`)
- **API:** `https://ekamraott.com/panel/rest-api/v130/` (key: `e378b63a18e2beb`, auth: `admin:1234`)
- **Firebase:** project `ekamraiptv`, Google login enabled, Phone login disabled
- **OneSignal:** `f6d10de9-0fd4-4880-b252-6634b0abbea1`
- **Internal package name:** `oxoo` (in pubspec.yaml — not user-visible, changing would break imports)

### Android TV App ✅
- **App ID:** `com.ekamra.tv`
- **Build:** compileSdk 34, minSdk 23, targetSdk 34, Java 21, NDK 21.3.6528147
- **Player:** ExoPlayer 2.19.1
- **Keystore:** `android-tv/keys/ekamra-tv.jks` (committed, password: `EkamraTV2026`, alias: `ekamra-tv`)
- **API:** `https://ekamraott.com/tv-panel/rest-api/v130/` (key: `524e18465932a14`, auth: `admin:1234`)
- **Firebase:** same project `ekamraiptv`, Google login enabled, Phone login disabled
- **SHA1:** `F7:3D:93:63:99:ED:F5:D7:5D:B8:9A:C8:4C:C1:D1:8E:34:42:4E:22`
- **TODO:** Add SHA keys to Firebase Console for `com.ekamra.tv`

### Admin Panel (Main) ✅
- **URL:** https://ekamraott.com/panel/
- **Server Path:** `~/domains/ekamraott.com/public_html/panel/`
- **Framework:** CodeIgniter 3, PHP 8.3.33
- **DB:** `u254324758_ekamraott` (32 tables)
- **Admin Login:** info@ekamraott.com / Ekamra@123
- **Web Server:** LiteSpeed (lsphp) — opcache very persistent, workaround: rename files to force cache reload

### TV Admin Panel ✅
- **URL:** https://ekamraott.com/tv-panel/
- **Server Path:** `~/domains/ekamraott.com/public_html/tv-panel/`
- **DB:** `u254324758_ekamra_tv` (33 tables, content tables empty)
- **Same admin login as main panel**

---

## GitHub Actions Workflows
| Workflow | App | Output | Status |
|----------|-----|--------|--------|
| `android-build.yml` | Flutter | Debug APK | ✅ |
| `build.yml` | Flutter | Release APK + AAB (signed) | ✅ |
| `android-release.yml` | Flutter | Signed Release AAB (tags/manual) | ✅ |
| `build-tv.yml` | Android TV | Release APK + AAB | ✅ |

---

## Recent Fixes (Latest Session — Sep 8-12, 2026)

### Mobile App Crash on Installation — FIXED ✅
**Root Causes:**
1. Keystore was gitignored → GitHub Actions built unsigned release APK → Android refuses to install
2. `build.yml` checked `vars.ANDROID_KEYSTORE_BASE64` instead of `secrets.ANDROID_KEYSTORE_BASE64`
3. ProGuard minification (`minifyEnabled true` + `shrinkResources true`) stripped critical classes

**Fixes Applied:**
- Committed keystore to `android/keys/ekamra-mobile.jks`
- Always-on signing config with env var fallbacks (matching TV app approach)
- Disabled ProGuard: `minifyEnabled false`, `shrinkResources false`
- Removed broken keystore decode step from `build.yml`
- Added gitignore exception for `android/keys/*.jks`

### Gradle/Build Cache Cleanup ✅
- Deleted `build/` (178.8 MB), `android/.gradle/` (1 MB), `android/build/` (0.1 MB)
- Deleted global Gradle cache `~\.gradle\` (2.4 GB)
- Deleted Flutter pub cache `~\AppData\Local\Pub\Cache` (2.8 GB)
- Total space freed: ~5.4 GB

### Admin Panel Audit (Partially Complete)
- **Completed:** Config audit, security audit, DB integrity check, REST API health check, branding scan
- **Temp audit files cleaned** from local and server

---

## Pending Tasks

### Admin Panel Fixes
- [ ] **cookie_httponly** — Main panel still has `FALSE`, should be `TRUE` (line 392 in config.php)
- [ ] **Branding cleanup** — Remaining references in:
  - `application/views/admin/index.php` line 18: `SpaGreen` in meta copyright
  - `application/controllers/Admin.php` line 6: `OXOO` in header comment, line 2965: `ovoo` in URL
  - `application/controllers/Cron.php` line 6: `OXOO` in header comment
  - `application/controllers/Legacy_api.php` line 8: `OXOO` in header comment
  - `application/controllers/Updater.php` lines 5-10: `OVOO` in header comments
- [ ] **Dashboard accessible without auth** — Investigate why dashboard loads without login
- [ ] **API returns 405** — Check REST API method handling for non-GET requests

### General
- [ ] **Add SHA keys to Firebase Console** for `com.ekamra.tv` app (needed for Google login on TV)
- [ ] **Upload content to TV panel** — TV database has empty content tables
- [ ] **TV-specific API endpoints** (optional, current `home_content` works)

---

## Key Configuration

### GitHub
- **Repo:** https://github.com/shihan84/ekamra.git
- **PAT:** (stored in agent memory, not in repo)
- **Branch:** main

### Server
- **Domain:** ekamraott.com
- **SSH:** `plink -P 65002 -pw "Mohsin@799184" u254324758@82.112.232.137`
- **SCP:** `pscp -P 65002 -pw "Mohsin@799184"`
- **PHP:** 8.3.33, CodeIgniter 3
- **Timezone:** Asia/Kolkata

### API (Main Panel)
- **Base URL:** `https://ekamraott.com/panel/rest-api/v130/`
- **API Key:** `e378b63a18e2beb`
- **Auth:** `Basic YWRtaW46MTIzNA==` (admin:1234)
- **Key endpoints:** `home_content_for_android`, `config`, `all_tv_channel_by_category`, `top_10_channels`

### API (TV Panel)
- **Base URL:** `https://ekamraott.com/tv-panel/rest-api/v130/`
- **API Key:** `524e18465932a14`
- **Auth:** `Basic admin:1234`

### Databases
| Panel | DB Name | DB User | DB Pass |
|-------|---------|---------|---------|
| Main | `u254324758_ekamraott` | `u254324758_ekamraott` | `Ekamraott321` |
| TV | `u254324758_ekamra_tv` | `u254324758_ekamra_tv` | `g*4vyB>X` |

---

## Key Files

### Flutter App
| File | Purpose |
|------|---------|
| `lib/config.dart` | API server URL, API key, OneSignal ID, auth flags |
| `lib/main.dart` | App entry point, Firebase + Hive init |
| `lib/app.dart` | Root widget, BLoC providers, theme |
| `android/app/build.gradle` | App ID, signing config, SDK versions |
| `android/keys/ekamra-mobile.jks` | Release signing keystore |
| `android/app/google-services.json` | Firebase config |

### Android TV App
| File | Purpose |
|------|---------|
| `android-tv/app/src/main/java/com/files/codes/AppConfig.java` | API URL, API key, phone login flag |
| `android-tv/app/build.gradle` | App ID, signing config, SDK versions |
| `android-tv/keys/ekamra-tv.jks` | Release signing keystore |

### Admin Panel
| File | Purpose |
|------|---------|
| `application/controllers/Admin.php` | Main admin controller |
| `application/controllers/rest_api/V130.php` | REST API controller |
| `application/models/api_v130_model.php` | API model |
| `application/config/config.php` | Session, cookie, CSRF config |

---

## Important Notes

- **Do NOT change** the internal package name `oxoo` in pubspec.yaml — it would break all Dart imports
- **LiteSpeed opcache** is very persistent — `opcache_reset()` doesn't fully work. Workaround: put new methods in controllers, or rename files to force cache reload
- **Keystores are committed** to the repo for both apps — this is intentional for CI signing
- **ProGuard must stay disabled** for the mobile app — enabling causes runtime crashes
- **Builds are done via GitHub Actions** — no local build tools needed

---

## Workflow Rules
- **Always commit changes to repo** after completing work
- **Always update HANDOFF.md** at end of session
- **Always update agent memory** with project state and decisions
- **Always deploy admin panel changes to server** via pscp
