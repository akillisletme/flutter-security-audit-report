# Flutter Application Security Audit Report
### (A scanning instruction prepared to be dropped into any Flutter + Firebase / Supabase project and handed to Claude Code)
**Version:** 2026-07 (rev.4) · **References:** OWASP Mobile Top 10 (2024 Final, still current in 2026) · OWASP MASVS/MASTG · Firebase & Supabase official security documentation · attacker-side research (reFlutter/Blutter/Frida/objection, multilingual sources)

---

## HOW YOU WILL USE THIS FILE (the command to give Claude Code)

This file is an **audit instruction**, not a fixed checklist. Different projects use different technologies (one project may use Supabase, another only Firebase, yet another both at once). Claude Code must **first understand the project**, decide by itself which items apply, and mark those that do not as "NOT APPLICABLE".

Tell the Claude Code instance running in the application's root directory the following:

> "There is a `GUVENLIK_DENETIM_RAPORU.md` in the project root directory. First, understand the project:
> 1. Read `pubspec.yaml` and extract which packages/services are used (Firebase or Supabase or both; what local storage; which auth providers; what network client).
> 2. Scan the `android/` and `ios/` folder structure and the manifest/plist files.
> 3. Then apply all the audit items in this report in order. For each item, read the relevant files and search for the red-flag patterns. Mark items that have no counterpart in the project as 'NOT APPLICABLE'; do not make things up.
> 4. Write every vulnerability you find into a file named `GUVENLIK_BULGULARI.md` according to the **Output Format** at the very end.
> **Do NOT change code. Only detect and report.** We will apply fixes item by item as I approve them."

Claude Code must detect before writing any code. Let us first see all the vulnerabilities, and we will apply fixes as the user approves them. Put every item you cannot be certain about (out-of-code settings such as Cloud/dashboard configuration) into the **"MANUAL VERIFICATION"** list.

**For a deep scan (so the LLM does not skim/skip lazily):** When examining each file or module, do not jump straight to a conclusion. First open an `<analysis>` tag and think step by step: *what am I looking for in this file, what do I see in the code, which rule in the report (A1 / C3 / ...) does this situation map to, did a red-flag pattern match?* When the analysis is done, write the finding into `GUVENLIK_BULGULARI.md`. Also, **do not silently skip any item:** at the end of the report, produce a **self-audit (coverage) list** that marks every item from A1 through F4 as "evaluated / NOT APPLICABLE / MANUAL VERIFICATION" — prove this way that no item was left unevaluated. In a large codebase, proceed section by section (give a brief interim summary when each major section is done); do not skim through it in one pass.

---

## SEVERITY DEFINITIONS

- 🔴 **CRITICAL** — If exploited, all data or accounts are compromised; loss of money/reputation is certain. Must be fixed immediately.
- 🟠 **HIGH** — A serious data leak or unauthorized access path. Must be fixed before the next release.
- 🟡 **MEDIUM** — Enlarges the attack surface; not critical on its own but becomes a link in the chain.
- 🔵 **LOW / INFO** — A best-practice violation, a long-term risk.

## STEP 0 — PROJECT INVENTORY (Claude Code should produce this first)

Before starting the report, produce a "Project Profile" table:

| Field | Detection |
|------|--------|
| Backend | Firebase / Supabase / both / custom API |
| Auth providers | email+password / Google / Apple / phone / anonymous / custom |
| Local storage | Hive / SharedPreferences / secure_storage / SQLite / Isar |
| Network client | Firebase SDK / dio / http / graphql |
| API paradigm | REST / GraphQL / gRPC / (SDK-only if none) |
| Custom native code | MethodChannel / EventChannel / Kotlin-Swift plugin present or not |
| State/DI | bloc / riverpod / provider / get_it ... |
| Navigation | go_router / auto_route / Navigator ... |
| Sensitive features | payment / location / camera / health / messaging / UGC (listing-comment) |
| Cloud Functions / Edge Functions | present or not |
| Firebase Storage / Supabase Storage | present or not |
| Application risk class | finance-payment / health / identity / UGC-social / game-hobby (calibrate severities accordingly) |

This profile determines which sections apply and how the finding severities should be calibrated. (The same "hardcoded secret" does not carry the same real risk in a hobby app as it does in a fintech app — use the risk class when weighing the impact of each finding.)

---

# ATTACKER PERSPECTIVE — THE REALITY OF CLIENT-SIDE PROTECTIONS IN 2026

This section is not an audit item, it is a **threat model.** When evaluating the items below, Claude Code (and the developer) must be aware of this reality: as of 2026, the tools a determined attacker has against a Flutter application are mature and largely automated. The ready-made toolchain produced by research communities in various languages (English, Russian, Chinese, Spanish, German) does the following:

**Static / code extraction:**
- **reFlutter** (an official tool in OWASP MASTG): patches `libapp.so`, dumps Dart class/function/name information, and bypasses some Flutter cert-pinning implementations **without requiring root**. (Note: In some Flutter versions the embedded proxy IP was removed, so a proxy setting on the device is now required — meaning it is not a block, just one extra step. Verify version-specific behavior at audit time.)
- **Blutter**: decodes the Dart AOT metadata from `libapp.so`; produces symbol names, an object pool dump and a ready Frida template for IDA/Ghidra. In an app without obfuscation, class/method names become readable.
- **Ghidra/IDA + Blutter script**: cryptographic details such as AES mode, SHA-256 usage, even an embedded RSA private key can be extracted from the disassembly.

**Dynamic / runtime:**
- **Frida + objection**: pinning bypass with a single command (`ios sslpinning disable`, `android sslpinning disable`), **keychain dump**, file system browsing, biometric verification bypass. **Without requiring jailbreak/root**, frida-gadget can be embedded into the app with `patchipa`/`patchapk`. (Verify tool versions with the current release at audit time; this depends on capability, not the specific version here.)
- **Multilingual ready-made script repos**: "universal" scripts that bypass root detection + SSL pinning + emulator + debug detection together are publicly available (especially widespread in the Chinese and English communities). Root/jailbreak detection alone is almost never sufficient — it can be bypassed.
- **Even jailbreak-detection libraries such as IOSSecuritySuite / freerasp** can be bypassed with a Frida hook or a patched binary (recent work by teams such as CyberCX and Appknox demonstrates this).

**Real-world outcome (published case):** A production Flutter (Android) app was fully reverse-engineered step by step — **the embedded RSA private key was extracted, the custom RSA-SHA256 signing format was reconstructed, and the production API was successfully called from an independent Python client.** So the app's assumption of "I reject any request that does not come from my own client" collapsed.

### The three defensive rules that follow from this (the foundation the entire report rests on)

1. **Everything that resides on the client is assumed to be readable.** Secrets, keys, signing logic, business rules — all can be extracted. This is why items A1 (secrets) and A11 (crypto) cannot be closed with "I obfuscated it, it's safe."
2. **Client-side protections (obfuscation, pinning, root/JB detection) do not *prevent* the attack, they make it *more expensive*.** They are not worthless — they eliminate mass/automated attacks and raise the cost of an attack — but as the sole line of defense they are never sufficient. They must be layered (defense-in-depth).
3. **The only real security boundary is the backend.** Authorization, data ownership, payment/premium verification, rate limiting must be enforced **on the server** (Firestore Rules / RLS / Cloud Function + App Check/Play Integrity). An attacker calls not your app but directly your API — the API must be able to validate that call on its own.

> **Claude Code should use this as follows:** For every finding, state whether a control is a "client-side protection" or a "backend enforcement". If a client-side protection alone guards a critical decision (premium, authorization, payment, data access) and there is no counterpart on the backend → clearly write in the "Impact" part of the finding: **"client-side, can be bypassed, no backend validation"**.

---

# SECTION A — CLIENT (FLUTTER / DART) SIDE

The fundamental truth is this: **the compiled APK/IPA is a publicly accessible file.** With tools such as `jadx`, `apktool`, `reFlutter`, `blutter`, `libapp.so` is decompiled; Dart AOT compilation makes this harder but does not prevent it. Do not treat anything that resides on the client as "secret". In OWASP Mobile Top 10 2024, **M7 (Insufficient Binary Protections)** and **M8 (Security Misconfiguration)** emphasize this reality.

> **OWASP 2024 category map (correct numbers):** M1 Improper Credential Usage · M2 Inadequate Supply Chain Security · M3 Insecure Authentication/Authorization · M4 Insufficient Input/Output Validation · M5 Insecure Communication · M6 Inadequate Privacy Controls · M7 Insufficient Binary Protections · M8 Security Misconfiguration · M9 Insecure Data Storage · M10 Insufficient Cryptography

## A1 — 🔴 Embedded Secrets / API Keys (OWASP M1: Improper Credential Usage)

**Number 1** on the OWASP 2024 list and still first in 2026. The most frequent and most expensive mistake. Uber's 2017 breach (S3 key in the repo), the Strava API key leak, the hardcoded credentials found in dozens of popular Android/iOS apps in 2024 — all belong to this category.

**Claude Code should scan the following:**
- All `.dart` files under `lib/`, `assets/`, `.env`, `pubspec.yaml`
- `android/app/google-services.json`, `ios/Runner/GoogleService-Info.plist`
- `android/local.properties`, `android/app/build.gradle(.kts)`, `ios/Runner/Info.plist`
- `firebase_options.dart` (FlutterFire CLI generated)
- Git history: keys that were later deleted but remain in history, via `git log -p | grep -iE '(apikey|secret|token|password|service_role)'`

**Red-flag patterns (regex/grep):**
```
apiKey|api_key|apikey|secret|password|passwd|token|bearer|private_key|client_secret
AIza[0-9A-Za-z_\-]{35}        → Google/Firebase API key
sk_live_|sk_test_|pk_live_    → Stripe key
eyJ[A-Za-z0-9_\-]+\.eyJ       → JWT (critical if it contains service_role)
service_role                  → Supabase service key must NEVER be on the client
supabaseKey|SUPABASE_SERVICE|SUPABASE_URL
ghp_|gho_|github_pat_         → GitHub token
xox[baprs]-                   → Slack token
-----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY-----
```

**Important distinctions (do not produce false positives):**
- **Firebase `apiKey`** (the one inside google-services.json / firebase_options.dart): technically "not secret", it identifies the project. Official Firebase documentation says: restricted keys for Firebase services are not considered secrets. **BUT if it is not restricted** (see A2) or is open to user enumeration, it can be abused. Mark the finding as "restriction check required", do not say "leaked secret".
- **Supabase `anon` key**: intentionally public, not a problem on its own — **BUT a disaster if RLS is off** (see section C). Tie the finding to the RLS status.
- **Supabase `service_role` key**: if found on the client, **immediately 🔴 CRITICAL** — it bypasses all RLS and grants full database access.
- **Google Maps / Places / Gemini API key**: it being in `AndroidManifest.xml`/`Info.plist`/code is sometimes mandatory, but **if it is not restricted by package name + SHA-1 (Android) or bundle ID (iOS)**, someone else burns your quota → you pay the bill. It must be a **separate** key from the Firebase key and restricted to the relevant API.

**The correct approach:** Real secrets reside on the backend (Cloud Functions / Supabase Edge Functions); the client requests a temporary token. `flutter_dotenv`, `--dart-define`, `--dart-define-from-file` are only for **non-sensitive** config — they can be read from the compiled binary, they are not a security tool.

## A2 — 🟠 No API Key Restriction (out-of-code, MANUAL VERIFICATION)

**Check:** In the Google Cloud Console, for each key:
- Is there an application restriction? (Android: package name + SHA-1 signature; iOS: bundle ID; Web: HTTP referrer)
- Is there an API restriction? (Allow only the APIs actually used)
- Is the Firebase key **separate** from the Maps/Gemini key?

Claude Code cannot see the Cloud Console setting by reading the code — mark this item as **"MANUAL VERIFICATION"** and remind the user. An unrestricted key = quota abuse, spam accounts, data enumeration, a surprise bill.

## A3 — 🔴 Insecure Local Storage (OWASP M9: Insecure Data Storage)

Critical reality: **Hive (hive/hive_ce), SharedPreferences, SQLite are by default unencrypted** plaintext. On a rooted/jailbroken device, or via ADB backup, they are read from under `/data/data/<package>/`. Talsec research: even the Firebase Auth JWT sits in **plaintext** inside `shared_prefs/com.google.firebase.auth.*.xml` and can be stolen on a rooted device and used for impersonation.

**Claude Code should scan the following:**
- Everywhere that writes `SharedPreferences`, `Hive`/`hive_ce`, `sqflite`, `isar`
- **What is written** to these stores: token, password, email, phone, national ID, location, session, card/IBAN → sensitive
- **Is `flutter_secure_storage` present in the project?** If not, and sensitive data is written locally → finding.

**Red flags:**
```
prefs.setString(...token...) / setString(...password...) / setString(...jwt...)
Hive box / SharedPreferences fields: authToken, refreshToken, accessToken,
    session, jwt, credit, card, iban, phone, email, tckn, otp, pin, apiKey
writing token/PII to a file with writeAsString(...) (to path_provider directories)
```

**Rules:**
- Sensitive data (token, session, identity, payment) → **`flutter_secure_storage`** (iOS Keychain / Android Keystore + EncryptedSharedPreferences).
- If there is sensitive data that must remain in Hive → an **AES-encrypted box** (`HiveAesCipher`), with the encryption key stored in **`flutter_secure_storage`** (not embedded in code).
- **Non-sensitive** offline data such as score/dice/theme/language/last tab → plain Hive/SharedPreferences is **fine**.
- Note: The Firebase SDK holding the token in plaintext is SDK behavior; the defense against this is **device integrity** (App Check / Play Integrity) + **short token lifetime + token revocation on logout** (see B and C).
- **Firestore offline persistence cache is also unencrypted** (`Settings(persistenceEnabled: true)` — on by default in Flutter). Every Firestore piece of data the app reads accumulates in a plaintext file on the device. For highly sensitive collections (messages, payment, identity), consider disabling persistence or adding a device-lock/extra encryption layer; at minimum check whether **`clearPersistence()` is called on logout** (if not called, the previous user's data remains on the device → leakage on a shared device, 🟠).

Claude Code should list every field written to all local stores and classify it as **sensitive / non-sensitive**.

## A4 — 🔴/🟠 Insecure Communication (OWASP M5) — HTTPS and Certificate Pinning

**Scan:**
- All endpoints starting with `http://` (non-HTTPS) → 🔴
- `android:usesCleartextTraffic="true"` in `AndroidManifest.xml` → 🟠
- `cleartextTrafficPermitted="true"` or loose `trust-anchors` in `res/xml/network_security_config.xml` → 🟠
- iOS `Info.plist`: `NSAllowsArbitraryLoads = true` under `NSAppTransportSecurity` (ATS bypass) → 🟠
- If `badCertificateCallback` always returns `true` in the Dio/http client, or `HttpClient` `onBadCertificate = (_,__,___) => true` → 🔴 (certificate validation disabled, open to MITM)

**Recommendation:** Enforce TLS 1.2+, forbid cleartext. In flows handling sensitive data, use **certificate pinning** (against MITM on public Wi-Fi). Firebase SDK traffic is already TLS; if there are additional `dio`/`http` calls, add pinning + token injection via interceptor, and do not leak sensitive data in error logs.

> **Realistic expectation:** Pinning eliminates mass/automated MITM and the amateur attacker, but reFlutter (`libapp.so` patch) and objection (`ios/android sslpinning disable`) can bypass many pinning implementations with ready-made commands **without root/jailbreak**. So pinning is not a guarantee of "my traffic can never be seen"; **the server must still validate every request on its own** (App Check/Play Integrity + auth). Adding pinning is good, *relying* on it is a mistake.

## A5 — 🟡 Sensitive Data Logging / Debug Mode On (OWASP M8)

**Scan:**
- Lines within `print(...)`, `debugPrint(...)`, `developer.log(...)`, `logger.*` that include token/password/PII/the full response body
- Verbose logging that runs in production without a `kReleaseMode`/`kDebugMode` check
- Sending PII to Crashlytics/Sentry as custom keys/logs (`setCustomKey`, `recordError`, `log`) — sending email/phone/token
- `android/app/build.gradle`: if `debuggable true` is left in the release build → 🟠
- If Dio `LogInterceptor` is left on in release (logs all requests/responses) → 🟠

**Rule:** Logs off/masked in release. Do not send raw PII to Crashlytics.

## A6 — 🟡 No Reverse Engineering / Obfuscation (OWASP M7: Insufficient Binary Protections)

**Scan:**
- Is `flutter build ... --obfuscate --split-debug-info=<dir>` used in build scripts / CI?
- Android `android/app/build.gradle`: are there `minifyEnabled true` + `shrinkResources true` + R8/ProGuard rules?

**Note:** Obfuscation is not 100% protection, it only raises the cost of the attack. **The real rule: move the sensitive business logic and authorization to the backend, treat the client as untrusted.** Riskless game/UI logic such as "did it reach 101", "which sound to play" can be on the client; but "is this user premium / is this transaction valid / is this score real" must be validated on the backend. Base critical checks not on obfuscation alone but on server validation.

> **Concrete:** If there is NO obfuscation, Blutter makes the class/method names from `libapp.so` readable and produces a ready symbol script for IDA/Ghidra — the code is read almost at source level. If there IS obfuscation this gets harder but not impossible (names go away, the logic remains). This is why `--obfuscate` **should exist** (especially if critical business logic is on the client) but must not be counted as sufficient on its own.

## A7 — 🟡 Over-permissioning + Root/Jailbreak & Tamper Detection

**Scan:** `AndroidManifest.xml` and `Info.plist` permissions vs. the features the app actually uses.
- Is `ACCESS_FINE_LOCATION` really needed, or is `ACCESS_COARSE_LOCATION` (city/approximate) enough? (In a map/listing feature, **approximate location instead of exact GPS** is correct for both privacy and KVKK.)
- `READ_SMS`, `READ_CONTACTS`, `CAMERA`, `RECORD_AUDIO`, `MANAGE_EXTERNAL_STORAGE`, background location, `QUERY_ALL_PACKAGES` → remove if unused (the store may also reject).
- For each iOS permission, is there a description string in `Info.plist` (`NSLocationWhenInUseUsageDescription`, `NSCameraUsageDescription`, etc.)? If not, the app crashes + store rejection.
- Is there **root/jailbreak/emulator detection** (in payment/sensitive apps)? Play Integrity / App Attest / `freerasp`, etc. If not, 🟡 for sensitive flows.

> **Warning:** Root/JB detection *alone* is not security. It is routinely bypassed with Frida/objection and the "universal bypass" scripts circulating in multilingual communities; even libraries such as IOSSecuritySuite/freerasp can be patched. Its value: warning the honest user in a risky environment + making automated/mass fraud harder. A critical decision (payment approval, money transfer) must **never** rest on the assumption "the device looked clean"; server-side validation + a device-integrity signal (Play Integrity/App Attest, evaluated on the backend) must be the basis.

## A8 — 🟠 Missing Input Validation (OWASP M4)

**Rule:** Client validation can be bypassed — **every validation must be repeated on the server.** For UGC fields such as listing text, username, comment, message:
- Are length/format/type limits also present on the server (Firestore Rules `request.resource.data...` / Supabase policy / Edge Function)?
- If WebView is used, are `javascriptMode` and `onNavigationRequest` safe? Is raw HTML/URL injection rendered (XSS)?
- Are URLs opened via deep link / `url_launcher` validated (see A10)?
- SQL/NoSQL: is user input embedded directly into the query?
- **GraphQL (if `graphql`/`graphql_flutter`/`ferry`/`artemis` is in the project; otherwise NOT APPLICABLE):**
  - Is **introspection disabled in production** on the backend? If it is on, an attacker reads the whole schema, type and field names from outside and maps the attack surface → 🟠.
  - Is there a **query depth / complexity limit** against overly deep/nested queries? If not, a single deep query can cause DoS + a bill explosion → 🟠.
  - Is field-level authorization done **on the server**? The query comes from the client; which field returns to whom must be limited on the backend (hiding a field on the client is not authorization — see B4).

## A9 — 🟡 Third-Party Dependencies (OWASP M2: Supply Chain)

**Scan:**
- The `flutter pub outdated` / `dart pub outdated` output — packages with known CVEs/abandoned
- Packages in `pubspec.yaml` that are unmaintained, low-download, pulled via `git:`/`path:`
- Unpinned versions (`any`, very broad `^`/`>=` ranges)
- Is `pubspec.lock` committed? (Needed for reproducible builds.)
- Are there unpinned actions/packages that run the build in CI?
- **Typosquatting (fake/similarly-named package):** Is there a name in `pubspec.yaml` intentionally similar to an official/popular package (e.g. `url_launcher` → `url_launcer`, `firebase_core` → `firebase_cor`, `flutter_secure_storage` → `flutter_secure_storge`)? **Manual eyeballing is unreliable** — verify every package name against the real package on pub.dev: are the verified publisher, download/like counts, last publish date and repo address consistent? A newly added + low-download + unverified-publisher package that closely resembles an official name → 🟠, investigate.

**Recommendation:** Monthly dependency scan, automated static analysis with MobSF, `flutter analyze`/`dart analyze` mandatory in CI, `pubspec.lock` under version control.

## A10 — 🟠 Deep Link / URL Launcher / Share Security

This item is for projects using `url_launcher`, `app_links`/`uni_links`, `share_plus`, WebView.

**Scan:**
- The URL **source** in `launchUrl(...)` calls: if it comes from user content (a listing link, a profile link coming from Firestore/Supabase) and the scheme is not validated → an attacker can inject `javascript:`, `file:`, `intent:`, a phishing `https://` link → 🟠. There must be an allow-list of only `https`/`mailto`/`tel`.
- Screens opened via **deep link / App Link (Android) / Universal Link (iOS)**: is a privileged action performed directly with the incoming parameter? (E.g. if `myapp://reset?token=...` is processed without validation, that is an account-takeover path.)
- Is `AndroidManifest.xml` `intent-filter` `android:autoVerify="true"` and `assetlinks.json` correct? (Against App Link hijack.)
- Is there **PII/token leakage** in content shared via `share_plus`? ("End-of-game card" images are usually harmless; but shared text should not contain an email/phone/deep link + token.)
- WebView: `Uri` validation, `onNavigationRequest` allow-list, `allowFileAccess`/`allowUniversalAccessFromFileURLs` off.
- **WebView ↔ Dart JS bridge:** If a channel to send messages to the native/Dart side of web content is opened with `addJavaScriptChannel(...)` / `JavascriptChannel`, the JS running inside the WebView (potentially attacker-controlled) can call this channel → if the bridge function **triggers a sensitive operation/data access with unvalidated input** 🟠. Access to the bridge only from trusted origins + strict validation of the incoming message is mandatory.

## A11 — 🟠 Weak / Incorrect Cryptography (OWASP M10: Insufficient Cryptography)

If there is any encryption/hashing/signing in the project (packages: `crypto`, `encrypt`, `pointycastle`, `cryptography`, etc.):

**Red flags:**
```
md5(...) / sha1(...)                    → 🔴 for password/integrity (broken algorithms)
AESMode.ecb / 'AES/ECB'                 → 🔴 (leaks patterns; use GCM/CBC+HMAC)
IV/nonce is constant or reused          → 🔴
Encrypter(Key.fromUtf8('sabitanahtar')) → 🔴 encryption key embedded in code (same class as A1)
generating token/key/OTP with Random()  → 🟠 (Random.secure() is mandatory for cryptographic work)
base64/XOR/Caesar as "encryption"       → 🔴 (encoding ≠ encryption)
custom self-written encryption algorithm → 🔴 (never custom crypto)
password hash plaintext or unsalted on client/DB → 🔴 (if Firebase/Supabase Auth is used they already handle this; in custom auth use bcrypt/scrypt/Argon2)
```

**Rules:**
- The encryption key never resides in code/assets → derived from the Keystore/Keychain (`flutter_secure_storage`) or from the server.
- If hashing is needed, SHA-256+; for passwords bcrypt/scrypt/Argon2 (even SHA alone is not enough).
- The `HiveAesCipher` key must be generated with `Hive.generateSecureKey()` and stored in secure storage (linked to A3).
- Note: If there is no manual crypto in the project, this item is "NOT APPLICABLE" — the Firebase/Supabase SDKs manage their own TLS/token crypto correctly; **not writing your own crypto code is already the best practice**, write this not as a finding but as a positive note.

## A12 — 🟡 Data Leakage via Screen / Clipboard / Keyboard / Memory (OWASP M6: Inadequate Privacy Controls)

For apps with sensitive screens (payment, identity, private messages, balance):

**Scan:**
- **Screenshot/recording prevention:** On Android, is `FLAG_SECURE` (or a package such as `screen_protector`/`no_screenshot`) present on sensitive screens? If not, a malicious/accessibility service that records the screen sees everything → 🟡/🟠 in a sensitive app.
- **App switcher snapshot:** When going to the background, iOS takes a photo of the screen and shows it in the app switcher; Android "Recents" does the same. Is the sensitive screen masked when going to the background (overlay/blur on `AppLifecycleState.paused/inactive`)? → if not, 🟡.
- **Clipboard:** Is a token/password/IBAN/personal data copied with `Clipboard.setData(...)`? On Android, other apps (pre-13) can read the clipboard; if copying is necessary, a timed clear should be considered → 🟡.
- **Keyboard cache / autofill:** On sensitive fields (`TextField`), are `obscureText: true` + `enableSuggestions: false` + `autocorrect: false` set? Third-party keyboards learn and cache what is typed; on a password field, is `autofillHints: [AutofillHints.password]` correct? → if missing, 🟡.
- **Notification preview:** Is there a notification showing sensitive content on the lock screen (see C6 FCM)?
- **Sensitive state remaining in memory after logout (🟡):** When `signOut()` is called, is only a UI redirect done, or is the sensitive state in memory (e.g. `UserBloc`, `WalletState`, `ProfileNotifier`, `riverpod`/`provider`/`bloc` objects holding token/PII) **explicitly reset / `close()`-ed / `dispose()`-ed**? If not reset, on a **shared device** (switching accounts while the app is alive as a process) the next user may see the previous user's data on screen/in state → 🟡. *Realistic note:* In Dart it is not possible to wipe memory in a guaranteed way (GC + string immutability); so this is not a "protection against a memory dump" but a **session hygiene** item — the real threat is not a rooted attacker dumping RAM but the next person using the same device. What matters: clean the state tree on logout + `clearPersistence()` (A3) + delete the session data in secure storage.

## A13 — 🟠 MethodChannel / Native Bridge Vulnerabilities (Kotlin/Swift) — Native Injection & Path Traversal

This item applies if the project has custom `MethodChannel` / `EventChannel` / `BasicMessageChannel` or hand-written native (Kotlin/Java, Swift/Obj-C) code. In a project that uses only ready-made Firebase/Supabase SDKs and has no custom platform channel → **NOT APPLICABLE.**

**Scan:**
- Find `MethodChannel('...')`, `invokeMethod(...)`, `EventChannel(...)` calls on the Dart side; match their counterparts on the Android (`android/app/src/main/kotlin|java/.../*.kt|*.java`, `MethodChannel(...).setMethodCallHandler` inside `configureFlutterEngine`) and iOS (`ios/Runner/*.swift`, `FlutterMethodChannel(...)`) sides.
- Do arguments coming from the client (`call.arguments`, `call.argument("x")`) flow **without validation** on the native side into:
  - a SQL/SQLite query (string concatenation → **SQL injection**),
  - a file path (`File(path)`, `FileInputStream`, `NSFileManager` → **path traversal**; escaping the app sandbox with `../../`),
  - an `Intent` (Android) parameter / `startActivity` / `PendingIntent` (→ intent injection, triggering unauthorized components),
  - `Runtime.exec` / `ProcessBuilder` (→ command injection),
  - `WebView.loadUrl` / `evaluateJavascript` (→ XSS / JS bridge abuse).

**Red flags:**
```
// Android (Kotlin)
db.rawQuery("SELECT * FROM t WHERE id = " + call.argument<String>("id"), null)  → SQL injection
File(context.filesDir, call.argument<String>("name")!!)   // no ../ / canonical check → path traversal
Runtime.getRuntime().exec(call.argument<String>("cmd"))   → command injection
// iOS (Swift)
let p = docsDir.appendingPathComponent(args["name"] as! String)  // no prefix/canonical check
```

**Rule (with the report's core philosophy):** Native code is also **part of the client** — validation done there can also be bypassed with reverse engineering/hooking. So both layers are needed at once:
1. **Correct coding** against injection/path-traversal on the native side is still mandatory (parameterized query, path canonicalization + `startsWith` check against the sandbox root, explicit `Intent`, input allow-list) — this is intra-client security hygiene.
2. **But the authorization/ownership/payment decision is not made here;** that decision is validated on the backend. Do not mistake MethodChannel for a "secure internal API" — an attacker can call the native function directly (via hooking) too. A claim coming from the channel such as "this user is authorized" cannot be trusted.

---

# SECTION B — AUTHENTICATION FLOWS (Login / Register / Session)

This section is OWASP M3 (Insecure Authentication/Authorization). Regardless of Firebase Auth / Supabase Auth / custom, **the login flow itself** is the most exploited surface. The following apply depending on which provider the project has.

## B1 — 🟠 Email Enumeration Protection (Firebase Auth)

In Firebase Auth/Google Identity Platform, `createAuthUri` / `fetchSignInMethodsForEmail` leak whether an email is registered → an attacker filters a leaked email list for "who has an account on this platform" → targeted phishing + credential stuffing. Google turned on **email enumeration protection** by default for projects created after **September 15, 2023**; `fetchSignInMethodsForEmail` was deprecated.

**Scan:**
- Is `fetchSignInMethodsForEmail(...)` used in the code? If so → the auth flow depends on this deprecated/disabled behavior, 🟠 (it breaks + enumeration risk). If a "enter email first, decide whether it's login or register" pattern relies on this API, it must be redesigned.
- **MANUAL VERIFICATION:** Firebase Console → Authentication → Settings → is "Email enumeration protection" on? (In older projects it must be turned on manually.)

## B2 — 🟠 Social Login Silent Account Creation & Provider Confusion (Google/Apple)

Critical behavior: with `signInWithCredential` / `signInWithPopup`, on a Google/Apple/Facebook sign-in, if there is no matching account Firebase **silently creates a new account**. `isNewUser` (AdditionalUserInfo) is only returned on the first creation. In examples like Bumble: an attacker first creates an account with the victim's **not-yet-registered** email and then takes it over.

**Scan (if the project has `google_sign_in` / `sign_in_with_apple` / Firebase social):**
- In the Google/Apple sign-in flow, is the `account-exists-with-different-credential` error **handled**? If not, two different provider accounts for the same email confuse authorization → 🟠. (There should be a "One account per email address" setting + a credential-linking flow.)
- Sign in with Apple: Apple may provide a **relay/private email** and returns **name/email only the first time**. If the code expects this info again on later logins it breaks; it must be saved on the first attempt.
- **Sign in with Apple requirement:** If the app offers another social login (Google/Facebook), per App Store Guideline 4.8 / 5.1.1 **Sign in with Apple must also be offered** (otherwise store rejection). If the project has Google but not Apple → 🟡 (store risk).
- Newer versions of `google_sign_in` have a different API from older versions (`GoogleSignIn.instance`, `authenticate()`, `authorizationClient`). If migration from the old `signIn()` pattern is incomplete, it causes build/runtime errors + a broken flow — check the version used and the API compatibility.

## B3 — 🟠 Session / Token Management

**Scan:**
- **Token revocation on logout:** If on logout only local state is cleared but `signOut()` is not called / the refresh token is not revoked, the session can continue with the token remaining on the device → 🟠. In Firebase, `FirebaseAuth.instance.signOut()`, and in sensitive scenarios additional server-side token revocation. (Check together with A12 that the sensitive state in memory is also cleared on logout.)
- **Refresh token persistence:** A refresh token produces idTokens indefinitely until revoked. In a sensitive app it must be revoked on logout/password change.
- **Email verification:** Are sensitive operations tied to the `emailVerified == true` condition? If not, authorization is granted with an unverified email.
- **MFA:** Is there multi-factor authentication on high-value accounts (payment, admin)? (Requires an upgrade to Firebase → Identity Platform.)
- **Anonymous auth:** Should be used only for temporary state, not as a substitute for a persistent account; anonymous users must also be limited by security rules ("anyone can create anonymous account").
- **Brute force / rate limit:** Is there a limit on login attempts? (Firebase is partially automatic; in custom auth it must be done manually.)
- **Password policy:** Are minimum length/complexity and leaked-password checks (Identity Platform password policy) on?

## B4 — 🟠 Authorization ≠ Authentication (Broken Access Control)

**#1 Broken Access Control** in OWASP Web Top 10 2021/2025. On mobile, too, the real hole is here.
- Are "logged in" (authenticated) and "can perform this action" (authorized) being confused? Is the role/ownership check done **on the server** (rules/policy/function), or only by hiding the UI on the client? Client-side hiding is not security.
- A route guard (go_router `redirect` / auto_route guard) only protects the UI; **data access must be protected by a backend rule**.
- IDOR: Can another user's document/record be fetched with their `id`? (E.g. if `/users/{someoneElsesId}` can be requested from the client and the rule does not check ownership → 🔴.)

---

# SECTION C — BACKEND (FIREBASE + SUPABASE)

Most mobile breaches happen not on the client but in **misconfigured backend rules**. In 2024, a single piece of research found ~125 million records (19 million of them plaintext passwords) exposed across 916 Firebase sites. On the Supabase side, most breaches stem from leaving RLS off.

> Note: whichever of these services the project has, that subsection applies; the one that is not present is marked **"NOT APPLICABLE"**.

## C1 — 🔴 Firebase Firestore / Realtime DB / Storage Security Rules

**Claude Code should read these files:** `firestore.rules`, `database.rules.json`, `storage.rules`, `firebase.json` (`firebase.json` shows which rules file is deployed).

**The most dangerous pattern (test mode leaked into production):**
```
allow read, write: if true;         → 🔴 everyone reads/writes everything
match /{document=**} { allow read, write: if true; }  → 🔴 the entire DB is open
".read": true, ".write": true        → 🔴 (Realtime DB)
```
Firestore "test mode" rules **close after 30 days**, but the period may have been extended and moved to production; also a date-based temporary rule such as `allow read, write: if request.time < timestamp.date(2025,...)` in production → 🔴.

**The second most common mistake — "everyone who is logged in can access everything":**
```
allow read, write: if request.auth != null;   → 🟠
```
`request.auth != null` only says "is logged in", it does not restrict **which data** can be accessed. Another user is also logged in.

**The correct pattern (ownership + data validation):**
```
match /users/{userId} {
  allow read, update, delete: if request.auth != null && request.auth.uid == userId;
  allow create: if request.auth != null && request.auth.uid == userId;
}
match /posts/{postId} {
  allow read: if true;                                        // listings are public-readable
  allow create: if request.auth != null
                && request.resource.data.authorId == request.auth.uid
                && request.resource.data.title.size() < 200;  // server-side validation
  allow update, delete: if request.auth != null
                && resource.data.authorId == request.auth.uid; // owner only
}
```

**Claude Code should ask for each collection:**
1. Is there an `allow ... if true` or a date-based temporary rule? → 🔴
2. Is it only `auth != null`, or is there an ownership/UID match? → if not, 🟠
3. Are `update`/`delete` restricted to the owner? (Being able to modify someone else's listing/document = data integrity + an account-takeover path)
4. Is there **field validation** on `create`/`update` (`request.resource.data...` type/length, immutable fields `resource.data.x == request.resource.data.x`)? Can the user write `role: admin` or `isPremium: true` themselves → 🔴 privilege escalation.
5. **`get` and `list` (query) rules are separate.** A rule that allows reading a single document by ID does not automatically allow **querying (query/list)** the collection; where a query is made, the rule must be satisfiable for all documents the query could return. Is `list`/query access limited by ownership/filter, or is it "auth != null, everyone queries the whole collection"? → unrestricted query 🟠/🔴.
6. **Storage rules are separate** (`storage.rules`) — Firestore may be secure while Storage is open. Are there file size (`request.resource.size < 5*1024*1024`) and type (`request.resource.contentType.matches('image/.*')`) limits? If not, quota abuse/malicious files.
7. Is there a write **rate limit** / abuse protection? (An attacker can write junk data and blow up the bill/limit.)

**Extra layer — App Check (🟠, very important):** Is **App Check** on for Firestore/Storage/RealtimeDB/Functions/Auth? It validates that the request comes from your genuine, unmodified app (Android: Play Integrity, iOS: App Attest); it eliminates fake requests thrown by scripts. Check whether `FirebaseAppCheck.instance.activate(...)` is in the code. If not, even if the security rules are correct, `anon`/automated requests are unrestricted → 🟠.

## C2 — 🟡 Firebase Remote Config Security

If `firebase_remote_config` (e.g. required version / feature flags) is used:
- **Remote Config is NOT a secret store.** Values come down to the client and are readable; putting an API key/secret/URL there → 🔴.
- **Force-update / feature gate can be bypassed on the client.** "Lock those below this version" logic is for UX; a **security decision** (is it premium, can it be accessed) must not rest on it — it must be validated on the server.
- If Remote Config conditions target with PII (full email/phone), a privacy note.

## C3 — 🔴 Supabase Row Level Security (RLS)

In Supabase, the **golden rule: RLS must be on for every table in the `public` schema, without exception.** The `anon` key is public; if RLS is off, anyone with that key reads/writes/deletes the whole table.

**Claude Code should scan the following:**
- `supabase/migrations/*.sql`, `schema.sql`, `.sql` files
- A `CREATE TABLE` exists but is **not** followed by `ENABLE ROW LEVEL SECURITY` → 🔴
- Tables created via the SQL editor/migration **do not automatically turn on RLS** (unlike the dashboard). If it is not explicitly in the migration → 🔴

**Check query (MANUAL, the user should run it):**
```sql
SELECT tablename FROM pg_tables
WHERE schemaname = 'public' AND NOT rowsecurity;
```
Every table returned = unprotected = 🔴.

**RLS-on-but-flawed-policy patterns:**
```sql
-- 🔴 a policy that "looks like it grants access but opens everything"
CREATE POLICY "read" ON profiles FOR SELECT USING (true);

-- ✅ correct: only its own row
CREATE POLICY "read own" ON profiles FOR SELECT USING (auth.uid() = user_id);
```

**Other critical checks:**
1. **Is the `service_role` key on the client?** (`--dart-define`, in code, `NEXT_PUBLIC_*`) → 🔴, it fully bypasses RLS.
2. **A policy relying on the `user_metadata` claim** → 🔴. The user can change this field themselves; if used in an authorization check, that is privilege escalation. `app_metadata` should be used.
3. **If an UPDATE policy has no `WITH CHECK`** → the user can move a row to another user/tenant. In multi-tenant, `tenant_id`/`org_id` must be protected with `WITH CHECK`.
4. **Can RPCs / Edge Functions be called with `anon`?** If admin/license/payment functions are publicly callable → 🔴.
5. **Storage buckets:** A bucket marked `public` = everyone reads all the files. Bucket policies and file-path design (tenant/uid in the path) must be checked.
6. **`service_role` leakage in an Edge Function:** logging all `env` at startup, returning a connection/secret in an error message.

## C4 — 🟠 Firebase + Supabase (or dual-backend) Mixed-Use Risk

If an app has **both Firebase Auth and Supabase DB** (or vice versa): how do the Supabase RLS policies validate the Firebase identity? If the identity bridge between the two systems (JWT validation, custom claims) is wrong, someone authenticated on one side can gain unauthorized access on the other. **Identity must be managed from a single source of truth**; the JWT signature validation (audience/issuer) must be complete.

## C5 — 🟠 Cloud Functions / Edge Functions Security

If the project has `functions/` or `supabase/functions/`:
- Do the functions validate identity/authorization, or is it "runs if called"? In non-callable HTTP functions, the auth check must be done manually.
- Are input validation, rate limiting, and `maxInstances` (against a bill-exploding DoS) configured?
- **Are secrets in Vault/Secret Manager or in plain env?** Even an environment variable (env var) can leak into function logs, error traces, or a line like `console.log(process.env)`. Real secrets must be kept in **Google Cloud Secret Manager / Firebase secrets (`defineSecret`) / Supabase Vault** and passed to the function **by reference**; not in the source code or a plain `.env`. Code that logs all `env` at startup → 🔴.
- Is the CORS setting too loose (`*`)?
- Do error responses leak a stack trace / connection info?

## C6 — 🟠 Push Notification (FCM) Security

If the project has `firebase_messaging`:
- **Is there sensitive data in the payload?** The notification content is visible on the lock screen, passes through Google/Apple servers, and can be logged on the device. OTP/token/balance/private message content **must not be put in the payload**; the notification says "you have a new message", and the content is fetched over a secure channel when the app opens (data-only push + fetch pattern). → if violated, 🟠.
- **Topic security:** **Anyone can subscribe to FCM topics** — if the `subscribeToTopic('...')` name on the client is guessable (e.g. `user_12345`, `admin`, `premium`), an attacker subscribes too and receives every notification sent to that topic. **User/group-specific data should be sent not via a topic but token-based (from the server to a single device).** → if user-specific content is sent via a topic, 🟠.
- **Where the FCM token is stored:** The token is usually kept in Firestore as something like `users/{uid}/fcmToken`. If **someone else can read/write** this field (see C1 rules), an attacker can change the token and redirect the victim's notifications to their own device → 🟠.
- **Server-side sending:** Is the Legacy Server Key (permanent; if leaked, anyone can push to everyone) used, or the **HTTP v1 API + service account (short-lived OAuth token)**? If a legacy key is embedded in code/client → 🔴. (Google shut down the legacy FCM API; if it is still used it is already broken.)
- If a deep link is processed on notification tap, the A10/D2 validations apply.

## C7 — 🟠 In-App Purchase / Premium Verification + Billing & Abuse Monitoring

**IAP / premium (if the project has `in_app_purchase`, RevenueCat, `purchases_flutter` or an `isPremium`/`isPro`-like field):**
- **Where is the premium status determined?** If the client can say "I bought it" and write `isPremium: true` to Firestore/Supabase → 🔴 (same privilege escalation as C1/C3 item 4). The correct way: **the receipt (receipt/purchase token) is validated on the server** (Cloud Function → Google Play Developer API / App Store Server API or RevenueCat webhook) and **only the server** writes the premium field; the security rules prevent the client from writing this field.
- If product price/entitlement info is hardcoded on the client it can be manipulated; entitlement must always depend on server validation.

**Billing and abuse monitoring (MANUAL VERIFICATION):**
- Is a GCP **budget alert** defined? Is there a **usage alarm** for Firestore/Functions/Storage? An attack (quota abuse, junk writes, an infinite-loop function) often first shows up in the bill; without an alarm it goes unnoticed for weeks.
- A `maxInstances` limit in Cloud Functions (C5) + an anomalous read/write alert in Firestore should be considered.
- The official Firebase security checklist recommends contacting support on a suspected DoS — note this as an incident-response step.

## C8 — 🟠/🔴 Trusting "Claimed" Data Coming from the Client (Location / Score / Time / Counter Manipulation)

Applies to every location-based, gamification, loyalty/points, step-counter, "check-in", distance/reward flow. The fundamental mistake: **the backend blindly accepting values sent by the client such as lat/lng, score, timestamp, step count, distance.** All of these are produced on the client → **all can be faked** (Fake GPS / mock location, editing the packet to inflate the score, changing the device clock).

**Scan / questions:**
- Does the client send raw location/score/time to the backend and the backend writes it directly (`score`, `distance`, `location` to Firestore/Supabase)? → if no validation, 🟠; if premium/reward/ranking is determined by this value, 🔴.
- Is there a **velocity / jump check** on the server? Is the distance/time between two consecutive locations physically possible (e.g. 400 km in 2 minutes = impossible)? Is a sudden jump rejected?
- Is Fake GPS / location mocking being prevented **only with a client-side plugin** (the `isMockLocation` flag)? This client signal is **not sufficient on its own** (it can be bypassed) — server-side consistency checks (speed, geofence, server timestamp) must be the basis.
- Timestamp: in critical flows, is the **server time** (`FieldValue.serverTimestamp()` / DB `now()`) used, or the time sent by the client? Trusting the client time → ranking/reward manipulation.
- Score/progress: if the final score is computed on the client and sent, it is manipulated; if critical, server validation / recomputation is needed.

**Rule:** This is of the same family as A8 (input validation) and B4 (authorization): **no "fact" that the client states is accepted as real without validation.** The backend must audit the claimed data for consistency using its own independent signals (server time, previous state, physical logic).

---

# SECTION D — ANDROID-SPECIFIC

For every project with an `android/` folder. Files: `android/app/src/main/AndroidManifest.xml`, `android/app/build.gradle(.kts)`, `android/app/proguard-rules.pro`, `res/xml/network_security_config.xml`.

## D1 — 🟠 Manifest Configuration
- `android:debuggable="true"` in release → 🔴 (usually comes from the build type; check if set manually).
- `android:allowBackup="true"` (default) → app data can be pulled via ADB backup. If there is sensitive data, set it to `false` or exclude it with `fullBackupContent` → 🟡/🟠. **For Android 12+ (API 31), `android:dataExtractionRules` must also be defined** — `fullBackupContent` alone does not cover the new device-to-device transfer scenario; token/session files must be `exclude`d in both rule sets.
- **Tapjacking:** On screens with sensitive confirm buttons (payment, permission, deletion), taps can be tricked with an overlay. On critical native views, is there `filterTouchesWhenObscured` / on the Flutter side a second step (PIN/biometric) on sensitive confirmations? → if not present in a sensitive app, 🟡.
- **Task hijacking (StrandHogg):** Is `android:taskAffinity` set to an empty string (`""`) and is `launchMode` appropriate? With the default taskAffinity, a malicious app can place itself on top of your task and show a fake login screen → 🟡.
- `android:usesCleartextTraffic="true"` → 🟠 (see A4).
- `activity`/`service`/`receiver`/`provider` with `android:exported="true"`: should it really be externally exposed? An unnecessary exported component = being triggered from another app / data leakage. Those with an `intent-filter` are automatically exported; review each one.
- A custom `ContentProvider` that is `exported` and permission-less → data leakage.

## D2 — 🟠 Deep/App Link Verification
- In activities that contain an `intent-filter`, is `android:autoVerify="true"` and is `.well-known/assetlinks.json` on the correct host? If not, another app can steal your link (App Link hijack).
- Do incoming link parameters trigger a sensitive operation without validation (see A10, B)?

## D3 — 🟡 Build / Code Shrinking
- Are `minifyEnabled true` + `shrinkResources true` on in release?
- Do the ProGuard/R8 rules protect sensitive classes and not cancel obfuscation with an unnecessary `-keep`?
- Signing: is the keystore password in plaintext in `build.gradle` inside `signingConfigs` (→ 🔴), or does it come from `key.properties`/env? Are `key.properties` and `*.keystore`/`*.jks` **in `.gitignore`**?

## D4 — 🟡 Play Integrity
- In sensitive apps, is the Play Integrity API / App Check (Play Integrity provider) integrated? Does the emulator/root/tamper detection give a signal to the backend?

---

# SECTION E — iOS-SPECIFIC

For every project with an `ios/` folder. Files: `ios/Runner/Info.plist`, `ios/Runner/Runner.entitlements`, `ios/Podfile`.

## E1 — 🟠 App Transport Security (ATS)
- `NSAppTransportSecurity` → `NSAllowsArbitraryLoads = true` global ATS bypass → 🟠. If an exception is needed there must be a domain-based `NSExceptionDomains`, not global.

## E2 — 🟠 URL Schemes & Universal Links
- Are flows opened via `CFBundleURLTypes` (custom scheme) validated? A custom scheme is open to hijacking; for sensitive flows **Universal Links** should be preferred.
- `Runner.entitlements` → is `com.apple.developer.associated-domains` (`applinks:`) correct? Is the `apple-app-site-association` file on the host?

## E3 — 🟡 Keychain & Privacy
- Is sensitive data in the Keychain (secure_storage uses this), or in plaintext in `NSUserDefaults`/plist? (see A3.)
- Keychain accessibility level: is it `kSecAttrAccessibleWhenUnlocked` (or stricter, e.g. `...WhenPasscodeSetThisDeviceOnly`), or a loose one like `...Always`/`AfterFirstUnlock`? A loose level = easier to read on a jailbroken device via `objection > ios keychain dump`.
- **Biometric verification is not the sole defense:** with objection/Frida, the iOS biometric check (LocalAuthentication) can be bypassed at runtime. Biometric "approval" must *keep the critical data locked* (the data must be encrypted so it cannot be decrypted without biometrics, using `SecAccessControl` flags — e.g. the `biometric_storage` package uses `SecAccessControlWithFlags`); the logic of merely "if (authenticated) show" is not enough.
- Are the `Info.plist` permission description strings (`NS...UsageDescription`) complete (crash + store rejection)?
- Are the **Privacy Manifest** (`PrivacyInfo.xcprivacy`) and "required reason API" declarations up to date? (Mandatory by Apple since 2024; if missing, store rejection → 🟡.)

## E4 — 🟡 Pod Dependencies
- Is `Podfile.lock` committed? Are the pods pinned, is there a pod with a known CVE?

---

# SECTION F — KVKK / LEGAL COMPLIANCE (LOCAL REQUIREMENT)

These are not technical "vulnerabilities" but they bring an **administrative fine in an audit**; they carry store-rejection and lawsuit risk. KVKK art. 12: the data controller "is obliged to take all kinds of technical and administrative measures to ensure an appropriate level of security." The technical items above are the technical leg of this obligation. (For projects requiring GDPR, parallel obligations apply.)

**Claude Code should search for the following within the project (presence/absence check):**

## F1 — 🟠 Privacy Notice + Explicit Consent Flow
- Is there a privacy notice screen/file? KVKK art. 10: the identity of the data controller, which data is processed for what purpose, to whom it is transferred, the method of collection / legal basis, the rights of the data subject.
- If there is no condition for processing personal data, or if there is a transfer, is there an **explicit consent** screen?
- Is **separate consent** obtained for commercial messages (FCM push, email)? (Unsolicited commercial messages are a separate violation.)
- Is there consent/opt-out for analytics/Crashlytics/ad SDKs?

## F2 — 🟠 Data Minimization (Location, etc.)
The KVKK principle of "connected to the purpose, limited and proportionate":
- In the map/listing system, is **city/approximate location used instead of an exact GPS pin**? (Both A7 security and KVKK.)
- Is phone number/email visibility **opt-in and off by default**? (Verify in the code that the relevant visibility field defaults to `false`.)
- Is a field that is not truly necessary being collected?

## F3 — 🟡 UGC & User Rights (Listing/Comment/Message System)
- Is there a complaint + block + content-report mechanism? (Both store and KVKK.)
- Is there a terms-of-use + privacy-policy acceptance flow?
- Is **account deletion** (data-erasure request) available in the app? KVKK art. 7 + Apple Guideline 5.1.1(v) + Google Play policy make it **mandatory** → if absent, 🟠 store risk.
- Can data access/rectification/portability requests be fulfilled?

## F4 — 🔵 Data Retention + VERBİS
- Is personal data kept "for as long as necessary", or indefinitely? Is there automatic deletion/anonymization?
- Does the VERBİS registration obligation apply? (Confirm once based on the exemption criteria.)
- Has cross-border data transfer (the Firebase/Supabase server region) been evaluated for compliance?

**Note:** The KVKK approach of "not free unless permitted, only what is permitted is free" maps one-to-one with the **deny-by-default** rule logic in Section C — a correct security design also ensures compliance.

---

# OUTPUT FORMAT (Claude Code should produce this → `GUVENLIK_BULGULARI.md`)

**1) The Project Profile table at the very top (Step 0).**

**2) Summary table:**

| Severity | Count |
|--------|------|
| 🔴 Critical | ? |
| 🟠 High | ? |
| 🟡 Medium | ? |
| 🔵 Low | ? |
| ⚪ Manual Verification | ? |
| ➖ Not Applicable | ? |

**3) For each finding (criticals at the top):**

```
## [🔴/🟠/🟡/🔵] Short title
- **Category:** (A1 / B2 / C1 / D3 / F3 ...)
- **File:line:** lib/product/db/auth_storage.dart:42
- **Problem:** The refresh token is written to SharedPreferences in plaintext.
- **Evidence:** `prefs.setString('refreshToken', token);`
- **Impact:** On a rooted device the token is read → session takeover. (State whether it is client-side or a backend enforcement.)
- **Recommended fix:** Move to flutter_secure_storage. (Do NOT write code, get approval first.)
```

**4) A separate "MANUAL VERIFICATION" list** (out-of-code items such as Cloud Console / dashboard / store settings): A2 key restriction, is App Check on, email enumeration protection, Firestore/RLS dashboard status, GraphQL introspection, GCP budget alert, VERBİS, etc.

**5) A separate "NOT APPLICABLE" list** (if that technology is not in the project: e.g. if no Supabase, C3; if no iOS folder, Section E; if no GraphQL, A8-GraphQL; if no custom native code, A13).

**6) Self-audit (coverage) list:** Mark **every item** among A1–A13, B1–B4, C1–C8, D1–D4, E1–E4, F1–F4 one by one as "evaluated / NOT APPLICABLE / MANUAL VERIFICATION". This list is the proof that no item was silently skipped — do not leave any item out.

---

# PRIORITY ORDER (fix these first)

1. 🔴 Is a `service_role` / real secret / embedded encryption key / legacy FCM server key on the client? (A1, A11, C3, C6)
2. 🔴 Firestore/RLS `if true` or an RLS-off table / IDOR / unrestricted `list` query? (C1, C3, B4)
3. 🔴 Can a user write their own `role`/`isPremium` field — is the IAP receipt validated on the server? (C1, C3, C7)
4. 🔴 Plaintext storage of token/password/PII (including the Firestore offline cache)? (A3)
5. 🔴 Cleartext HTTP / certificate validation disabled / broken crypto (MD5-SHA1-ECB)? (A4, A11)
6. 🔴/🟠 Is location/score/time accepted as-is from the client, with no velocity/jump check? (C8)
7. 🟠 Injection / path traversal in the MethodChannel / native bridge (SQL, file path, Intent, exec)? (A13)
8. 🟠 Only `auth != null`, no ownership check? (C1, B4)
9. 🟠 No token revocation + `clearPersistence()` + sensitive-state cleanup on logout / social login silent account creation? (B2, B3, A3, A12)
10. 🟠 App Check + API key restriction + email enumeration protection? (C1, A2, B1)
11. 🟠 FCM: sensitive data in the payload, guessable topic, unprotected token field? (C6)
12. 🟠 Deep link / url_launcher / WebView JS bridge validation, exported components, tapjacking/task hijacking? (A10, D1, D2)
13. 🟠 GraphQL introspection on / no query depth limit? (A8)
14. 🟠 Are secrets in plain env, or in Secret Manager/Vault? (C5)
15. 🟠 KVKK privacy notice/consent/account-deletion flow? (F1–F3)
16. 🟡 Screen snapshot/clipboard/keyboard/memory leaks, typosquatting, absence of a budget alert? (A12, A9, C7)

---

### Sources (current as of 2026)
- OWASP Mobile Top 10 (2024 Final Release) — M1 Improper Credential Usage … M10 Insufficient Binary Protections
- OWASP MASVS & MASTG (Mobile App Security Verification Standard / Testing Guide)
- OWASP Top 10:2025 (Web) — Broken Access Control #1 (for authorization logic); OWASP GraphQL Cheat Sheet for GraphQL introspection/depth
- Firebase official: Security Checklist, Firestore/Storage Security Rules (get vs list), App Check, "Understand API keys", Email Enumeration Protection docs; Cloud Functions secrets (`defineSecret`) / Google Cloud Secret Manager
- FlutterFire: Auth error handling, `account-exists-with-different-credential`
- FCM: HTTP v1 API migration (legacy API shut down), topic messaging security notes
- Android: Backup & Restore (`dataExtractionRules`, API 31+), tapjacking (`filterTouchesWhenObscured`), StrandHogg/task affinity research
- Google Play Developer API / App Store Server API (server-side purchase validation)
- Flutter platform channels (MethodChannel/EventChannel) native security notes — input validation, path canonicalization, explicit Intent
- **Attacker-side tooling and research (to calibrate the defense expectation, multilingual — verify versions with the current release at audit time):**
  - reFlutter (OWASP MASTG tool) · Blutter — Flutter/Dart AOT reverse engineering
  - Frida · objection (SensePost) — pinning/keychain/biometric runtime bypass
  - Multilingual "universal" root/SSL/emulator bypass script repos (Frida CodeShare, GitHub — English & Chinese communities) · HackTricks anti-instrumentation
  - CyberCX & Appknox — Flutter/iOS jailbreak-detection (IOSSecuritySuite) bypass work
  - iOS pentest methodology (Frida, objection keychain dump, MASVS L1/L2)
  - Published production Flutter reverse-engineering case — embedded RSA key extraction + API re-invocation
  - Talsec freeRASP — Flutter RASP (protection side, counter-measure reference)
- Supabase official: Row Level Security & "Securing your API" docs · Supabase Vault
- KVKK Personal Data Security Guide (Technical and Administrative Measures) + Board Decision Summaries
- Apple App Store Review Guidelines (4.8 / 5.1.1) · Google Play Data Safety & account-deletion policies
