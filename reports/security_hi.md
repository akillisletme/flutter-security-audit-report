# Flutter एप्लिकेशन सुरक्षा ऑडिट रिपोर्ट
### (किसी भी Flutter + Firebase / Supabase प्रोजेक्ट में रखकर Claude Code को दिए जाने के लिए तैयार की गई स्कैन निर्देशिका)
**संस्करण:** 2026-07 (rev.4) · **संदर्भ:** OWASP Mobile Top 10 (2024 Final, 2026 में भी अद्यतन) · OWASP MASVS/MASTG · Firebase & Supabase आधिकारिक सुरक्षा दस्तावेज़ · हमलावर-पक्ष अनुसंधान (reFlutter/Blutter/Frida/objection, बहुभाषी स्रोत)

---

## इस फ़ाइल का उपयोग कैसे करोगे (Claude Code को दिया जाने वाला निर्देश)

यह फ़ाइल एक **ऑडिट निर्देशिका है**, कोई स्थिर चेकलिस्ट नहीं। अलग-अलग प्रोजेक्ट में अलग-अलग तकनीकें होती हैं (एक प्रोजेक्ट में Supabase, दूसरे में केवल Firebase, किसी और में दोनों एक साथ हो सकते हैं)। Claude Code को **पहले प्रोजेक्ट को समझना** चाहिए, कौन-सी मदें लागू होती हैं यह स्वयं निर्धारित करना चाहिए, जो लागू नहीं होतीं उन्हें "लागू नहीं (UYGULANMAZ)" के रूप में चिह्नित करना चाहिए।

एप्लिकेशन की रूट डायरेक्टरी में चल रहे Claude Code से यह कहो:

> "प्रोजेक्ट रूट डायरेक्टरी में `GUVENLIK_DENETIM_RAPORU.md` है। पहले प्रोजेक्ट को समझो:
> 1. `pubspec.yaml` पढ़ो, कौन-से पैकेज/सेवाएँ उपयोग हो रहे हैं वह निकालो (Firebase है, Supabase है, या दोनों; स्थानीय स्टोरेज क्या है; auth प्रदाता कौन-से हैं; नेटवर्क क्लाइंट क्या है)।
> 2. `android/` और `ios/` फ़ोल्डर संरचना, manifest/plist फ़ाइलों को स्कैन करो।
> 3. फिर इस रिपोर्ट की सभी नियंत्रण मदों को क्रमवार लागू करो। हर मद के लिए संबंधित फ़ाइलें पढ़ो, रेड-फ़्लैग pattern खोजो। प्रोजेक्ट में जिनका कोई समकक्ष नहीं है उन मदों को 'लागू नहीं (UYGULANMAZ)' के रूप में चिह्नित करो, मनगढ़ंत मत बनाओ।
> 4. तुम्हें मिली हर कमज़ोरी को अंत में दिए गए **आउटपुट फ़ॉर्मैट** के अनुसार `GUVENLIK_BULGULARI.md` नामक फ़ाइल में लिखो।
> **कोड मत बदलो। केवल पता लगाओ और रिपोर्ट करो।** सुधार तो जैसे-जैसे मैं मद-दर-मद स्वीकृति दूँ वैसे ही करेंगे।"

Claude Code को कोड लिखने से पहले पहचान करनी चाहिए। पहले सभी कमज़ोरियाँ देख लें, सुधार तो जैसे-जैसे उपयोगकर्ता स्वीकृति दे वैसे-वैसे करेंगे। जिस भी मद के बारे में तुम निश्चित नहीं हो (Cloud/dashboard सेटिंग जैसी कोड-बाहरी) उसे **"मैनुअल सत्यापन (MANUEL DOĞRULAMA)"** सूची में डालो।

**गहन स्कैन के लिए (LLM सतही/आलसी होकर न गुज़रे):** हर फ़ाइल या मॉड्यूल की जाँच करते समय सीधे निष्कर्ष पर मत कूदो। पहले एक `<analiz>` टैग खोलकर चरण-दर-चरण सोचो: *इस फ़ाइल में मैं क्या खोज रहा हूँ, कोड में मुझे क्या दिख रहा है, यह स्थिति रिपोर्ट के किस नियम (A1 / C3 / ...) से मेल खाती है, कोई रेड-फ़्लैग pattern मेल खाया क्या?* विश्लेषण समाप्त होते ही निष्कर्ष `GUVENLIK_BULGULARI.md` में लिखो। साथ ही **किसी भी मद को चुपचाप मत छोड़ो:** रिपोर्ट के अंत में A1–F4 के बीच की हर मद को "मूल्यांकित / लागू नहीं / मैनुअल सत्यापन" के रूप में चिह्नित करने वाली एक **स्व-ऑडिट (कवरेज) सूची** तैयार करो — इस तरह सिद्ध करो कि तुमने कोई भी मद बिना मूल्यांकन के नहीं छोड़ी। एक लंबे कोडबेस में खंड-दर-खंड आगे बढ़ो (हर मुख्य खंड समाप्त होते ही एक संक्षिप्त अंतरिम सारांश दो), एक ही बार में सतही रूप से मत गुज़रो।

---

## स्तर परिभाषाएँ

- 🔴 **क्रिटिकल** — यदि इसका शोषण हुआ तो सारा डेटा या सभी खाते हाथ से निकल जाते हैं, पैसा/प्रतिष्ठा की हानि निश्चित है। तुरंत ठीक करना चाहिए।
- 🟠 **उच्च** — गंभीर डेटा रिसाव या अनधिकृत पहुँच का मार्ग। अगले संस्करण से पहले ठीक करना चाहिए।
- 🟡 **मध्यम** — हमले की सतह को बढ़ाता है, अकेले क्रिटिकल नहीं पर श्रृंखला की एक कड़ी बन जाता है।
- 🔵 **निम्न / जानकारी** — अच्छी प्रथा का उल्लंघन, दीर्घकाल में जोखिम।

## चरण 0 — प्रोजेक्ट इन्वेंटरी (Claude Code पहले इसे तैयार करे)

रिपोर्ट शुरू करने से पहले एक "प्रोजेक्ट प्रोफ़ाइल" तालिका निकालो:

| क्षेत्र | पहचान |
|------|--------|
| Backend | Firebase / Supabase / दोनों / custom API |
| Auth प्रदाता | ई-मेल+पासवर्ड / Google / Apple / फ़ोन / अनाम / custom |
| स्थानीय स्टोरेज | Hive / SharedPreferences / secure_storage / SQLite / Isar |
| नेटवर्क क्लाइंट | Firebase SDK / dio / http / graphql |
| API paradigma | REST / GraphQL / gRPC / (न हो तो केवल SDK) |
| कस्टम native कोड | MethodChannel / EventChannel / Kotlin-Swift प्लगइन है/नहीं |
| State/DI | bloc / riverpod / provider / get_it ... |
| नेविगेशन | go_router / auto_route / Navigator ... |
| संवेदनशील विशेषताएँ | भुगतान / स्थान / कैमरा / स्वास्थ्य / मैसेजिंग / UGC (लिस्टिंग-टिप्पणी) |
| Cloud Functions / Edge Functions | है/नहीं |
| Firebase Storage / Supabase Storage | है/नहीं |
| एप्लिकेशन जोखिम वर्ग | वित्त-भुगतान / स्वास्थ्य / पहचान / UGC-सामाजिक / गेम-शौक (स्तरों को इसी के अनुसार कैलिब्रेट करो) |

यह प्रोफ़ाइल यह निर्धारित करती है कि कौन-से खंड लागू होते हैं और निष्कर्ष-स्तरों को कैसे कैलिब्रेट किया जाएगा। (एक ही "hardcoded secret" शौक वाले एप्लिकेशन में और एक fintech में एक जैसा वास्तविक जोखिम नहीं रखता — जोखिम वर्ग को हर निष्कर्ष के प्रभाव को तौलते समय उपयोग करो।)

---

# हमलावर परिप्रेक्ष्य — 2026 में क्लाइंट-साइड सुरक्षा की वास्तविकता

यह खंड कोई नियंत्रण मद नहीं, बल्कि **थ्रेट मॉडल है।** नीचे दी गई मदों का मूल्यांकन करते समय Claude Code (और डेवलपर) को यह सच्चाई पता होनी चाहिए: 2026 तक, एक दृढ़ हमलावर के पास Flutter एप्लिकेशन के विरुद्ध जो उपकरण हैं वे परिपक्व और बड़े पैमाने पर स्वचालित हैं। विभिन्न भाषाओं (अंग्रेज़ी, रूसी, चीनी, स्पेनिश, जर्मन) के अनुसंधान समुदायों द्वारा तैयार की गई तैयार टूल-चेन ये काम करती है:

**स्थैतिक / कोड निष्कर्षण:**
- **reFlutter** (OWASP MASTG में आधिकारिक उपकरण): `libapp.so` को patch करता है, Dart क्लास/फ़ंक्शन/नाम की जानकारी उगल देता है, कुछ Flutter cert-pinning implementation को **root की आवश्यकता के बिना** bypass करता है। (नोट: कुछ Flutter संस्करणों में एम्बेडेड proxy IP हटा दिया गया, अब डिवाइस पर proxy सेटिंग की आवश्यकता है — यानी रुकावट नहीं, केवल एक और कदम। संस्करण-विशिष्ट व्यवहार को ऑडिट के समय पुष्टि करो।)
- **Blutter**: `libapp.so` से Dart AOT metadata को हल करता है; IDA/Ghidra के लिए symbol नाम, object pool डंप और तैयार Frida टेम्पलेट तैयार करता है। obfuscation न होने वाले एप्लिकेशन में क्लास/मेथड नाम पठनीय हो जाते हैं।
- **Ghidra/IDA + Blutter script**: AES मोड, SHA-256 उपयोग, यहाँ तक कि एम्बेडेड RSA private key जैसे क्रिप्टोग्राफ़िक विवरण disassembly से निकाले जा सकते हैं।

**गतिशील / रनटाइम:**
- **Frida + objection**: एक ही कमांड से (`ios sslpinning disable`, `android sslpinning disable`) pinning bypass, **keychain dump**, फ़ाइल-सिस्टम में विचरण, बायोमेट्रिक सत्यापन bypass। **Jailbreak/root की आवश्यकता के बिना** `patchipa`/`patchapk` से frida-gadget एप्लिकेशन में एम्बेड किया जा सकता है। (टूल के संस्करणों को ऑडिट के समय अद्यतन संस्करण से पुष्टि करो; यहाँ बात वास्तविक संस्करण की नहीं, क्षमता की है।)
- **बहुभाषी तैयार script रिपॉज़िटरी**: root detection + SSL pinning + emulator + debug detection को एक साथ bypass करने वाले "universal" script सार्वजनिक रूप से उपलब्ध हैं (चीनी और अंग्रेज़ी समुदायों में विशेष रूप से व्यापक)। Root/jailbreak detection अकेले लगभग कभी भी पर्याप्त नहीं होता — bypass किया जा सकता है।
- **IOSSecuritySuite / freerasp जैसी jailbreak-detection लाइब्रेरियाँ भी** Frida hook या patched binary से bypass की जा सकती हैं (CyberCX, Appknox जैसी टीमों के हालिया कार्य यह दिखाते हैं)।

**वास्तविक-दुनिया परिणाम (प्रकाशित मामला):** उत्पादन में चल रहे एक Flutter (Android) एप्लिकेशन को चरण-दर-चरण पूरी तरह रिवर्स-इंजीनियर किया गया — **एम्बेडेड RSA private key निकाला गया, custom RSA-SHA256 signing फ़ॉर्मैट पुनर्निर्मित किया गया और उत्पादन API को एक स्वतंत्र Python client से सफलतापूर्वक कॉल किया गया।** यानी एप्लिकेशन की यह धारणा कि "मैं अपने client से न आने वाले अनुरोध को अस्वीकार करूँगा" ध्वस्त हो गई।

### इससे निकलने वाले तीन रक्षा नियम (पूरी रिपोर्ट जिन पर आधारित है)

1. **क्लाइंट पर जो कुछ भी रहता है उसे पठनीय मान लो।** रहस्य, कुंजी, signing लॉजिक, व्यावसायिक नियम — सब कुछ निकाला जा सकता है। इसीलिए A1 (रहस्य) और A11 (क्रिप्टो) मदें "मैंने obfuscate कर दिया, सुरक्षित हूँ" से बंद नहीं की जा सकतीं।
2. **क्लाइंट-साइड सुरक्षा (obfuscation, pinning, root/JB detection) हमले को *रोकती नहीं, महँगा बनाती हैं*।** ये बेकार नहीं हैं — बड़े पैमाने/स्वचालित हमले को छाँटती हैं, हमले की लागत बढ़ाती हैं — पर एकमात्र रक्षा-पंक्ति के रूप में कभी भी पर्याप्त नहीं। इन्हें परतदार होना चाहिए (defense-in-depth)।
3. **एकमात्र वास्तविक सुरक्षा सीमा backend है।** प्राधिकरण, डेटा स्वामित्व, भुगतान/premium सत्यापन, दर सीमा **सर्वर पर** (Firestore Rules / RLS / Cloud Function + App Check/Play Integrity) लागू होनी चाहिए। एक हमलावर तुम्हारे एप्लिकेशन को नहीं, सीधे तुम्हारे API को कॉल करता है — API को उस कॉल को स्वयं सत्यापित कर पाना चाहिए।

> **Claude Code इसे इस तरह उपयोग करे:** किसी नियंत्रण के "क्लाइंट-साइड सुरक्षा" है या "backend प्रवर्तन" है — यह हर निष्कर्ष में बताओ। यदि एक क्लाइंट-साइड सुरक्षा अकेले किसी क्रिटिकल निर्णय (premium, अधिकार, भुगतान, डेटा पहुँच) की रक्षा कर रही है और backend में उसका कोई समकक्ष नहीं है → इसे निष्कर्ष के "प्रभाव" भाग में **"क्लाइंट-साइड bypass किया जा सकता है, backend सत्यापन नहीं है"** के रूप में स्पष्ट रूप से लिखो।

---

# खंड A — क्लाइंट (FLUTTER / DART) पक्ष

मूल सच्चाई यह है: **संकलित APK/IPA एक सार्वजनिक फ़ाइल है।** `jadx`, `apktool`, `reFlutter`, `blutter` जैसे उपकरणों से `libapp.so` decompile किया जाता है; Dart AOT संकलन इसे कठिन बनाता है पर रोकता नहीं। क्लाइंट पर रहने वाली किसी भी चीज़ को "गुप्त" मत मानो। OWASP Mobile Top 10 2024 में **M7 (Insufficient Binary Protections)** और **M8 (Security Misconfiguration)** इस सच्चाई पर ज़ोर देते हैं।

> **OWASP 2024 श्रेणी मानचित्र (सही संख्याएँ):** M1 Improper Credential Usage · M2 Inadequate Supply Chain Security · M3 Insecure Authentication/Authorization · M4 Insufficient Input/Output Validation · M5 Insecure Communication · M6 Inadequate Privacy Controls · M7 Insufficient Binary Protections · M8 Security Misconfiguration · M9 Insecure Data Storage · M10 Insufficient Cryptography

## A1 — 🔴 एम्बेडेड रहस्य / API कुंजियाँ (OWASP M1: Improper Credential Usage)

OWASP 2024 सूची में **नंबर 1** और 2026 में भी पहले स्थान पर। सबसे बार-बार और सबसे महँगी गलती। Uber का 2017 का उल्लंघन (S3 कुंजी repo में), Strava API कुंजी रिसाव, 2024 में दर्जनों लोकप्रिय Android/iOS एप्लिकेशनों में मिले hardcoded credential — सब इसी श्रेणी से हैं।

**Claude Code ये स्कैन करे:**
- `lib/` के अंतर्गत सभी `.dart` फ़ाइलें, `assets/`, `.env`, `pubspec.yaml`
- `android/app/google-services.json`, `ios/Runner/GoogleService-Info.plist`
- `android/local.properties`, `android/app/build.gradle(.kts)`, `ios/Runner/Info.plist`
- `firebase_options.dart` (FlutterFire CLI द्वारा उत्पन्न)
- Git इतिहास: `git log -p | grep -iE '(apikey|secret|token|password|service_role)'` से बाद में हटाई गई पर history में रह गई कुंजियाँ

**रेड-फ़्लैग pattern (regex/grep):**
```
apiKey|api_key|apikey|secret|password|passwd|token|bearer|private_key|client_secret
AIza[0-9A-Za-z_\-]{35}        → Google/Firebase API key
sk_live_|sk_test_|pk_live_    → Stripe key
eyJ[A-Za-z0-9_\-]+\.eyJ       → JWT (service_role içeriyorsa kritik)
service_role                  → Supabase servis anahtarı ASLA istemcide olmamalı
supabaseKey|SUPABASE_SERVICE|SUPABASE_URL
ghp_|gho_|github_pat_         → GitHub token
xox[baprs]-                   → Slack token
-----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY-----
```

**महत्वपूर्ण भेद (false positive मत बनाओ):**
- **Firebase `apiKey`** (google-services.json / firebase_options.dart के अंदर वाला): तकनीकी रूप से "गुप्त नहीं", प्रोजेक्ट की पहचान करता है। Firebase आधिकारिक दस्तावेज़ कहता है: Firebase सेवाओं के लिए प्रतिबंधित कुंजियाँ रहस्य नहीं मानी जातीं। **लेकिन यदि प्रतिबंधित नहीं है** (देखो A2) या user enumeration के लिए खुली है तो दुरुपयोग की जाती है। निष्कर्ष को "प्रतिबंध जाँच आवश्यक" के रूप में चिह्नित करो, "लीक हुआ secret" मत कहो।
- **Supabase `anon` key**: जानबूझकर सार्वजनिक, अकेले कोई समस्या नहीं — **लेकिन यदि RLS बंद है तो आपदा** (देखो खंड C)। निष्कर्ष को RLS की स्थिति से जोड़ो।
- **Supabase `service_role` key**: यदि क्लाइंट में मिले तो **तुरंत 🔴 क्रिटिकल** — सारे RLS को bypass कर देती है, पूर्ण डेटाबेस पहुँच देती है।
- **Google Maps / Places / Gemini API key**: `AndroidManifest.xml`/`Info.plist`/कोड में होना कभी-कभी अनिवार्य है पर **पैकेज नाम + SHA-1 (Android) या bundle ID (iOS) से प्रतिबंधित न हो तो** कोई और तुम्हारा कोटा खर्च करता है → बिल तुम भरते हो। Firebase कुंजी से **अलग** कुंजी होनी चाहिए और संबंधित API तक प्रतिबंधित होनी चाहिए।

**सही दृष्टिकोण:** वास्तविक रहस्य backend में (Cloud Functions / Supabase Edge Functions) रहते हैं; क्लाइंट अस्थायी token माँगता है। `flutter_dotenv`, `--dart-define`, `--dart-define-from-file` केवल **गैर-संवेदनशील** config के लिए हैं — संकलित binary से पढ़े जा सकते हैं, सुरक्षा उपकरण नहीं हैं।

## A2 — 🟠 API कुंजी प्रतिबंध नहीं है (कोड-बाहरी, मैनुअल सत्यापन)

**जाँच:** Google Cloud Console में हर कुंजी के लिए:
- एप्लिकेशन प्रतिबंध है क्या? (Android: पैकेज नाम + SHA-1 signature; iOS: bundle ID; Web: HTTP referrer)
- API प्रतिबंध है क्या? (केवल उपयोग की जाने वाली API को अनुमति)
- Firebase कुंजी और Maps/Gemini कुंजी **अलग** हैं क्या?

Claude Code कोड पढ़कर Cloud Console सेटिंग नहीं देख सकता — इस मद को **"मैनुअल सत्यापन"** के रूप में चिह्नित करे और उपयोगकर्ता को याद दिलाए। अप्रतिबंधित कुंजी = कोटा दुरुपयोग, spam खाते, डेटा enumeration, अप्रत्याशित बिल।

## A3 — 🔴 असुरक्षित स्थानीय स्टोरेज (OWASP M9: Insecure Data Storage)

क्रिटिकल सच्चाई: **Hive (hive/hive_ce), SharedPreferences, SQLite डिफ़ॉल्ट रूप से बिना एन्क्रिप्शन के** सादा-पाठ हैं। rooted/jailbroken डिवाइस पर या ADB backup से `/data/data/<paket>/` के अंतर्गत से पढ़े जाते हैं। Talsec अनुसंधान: Firebase Auth का JWT भी `shared_prefs/com.google.firebase.auth.*.xml` के अंदर **सादा-पाठ** में रहता है और rooted डिवाइस पर चुराकर impersonation में उपयोग किया जा सकता है।

**Claude Code ये स्कैन करे:**
- `SharedPreferences`, `Hive`/`hive_ce`, `sqflite`, `isar` लिखने वाली सभी जगहें
- इन स्टोरों में **क्या लिखा जाता है**: token, पासवर्ड, ई-मेल, फ़ोन, TC पहचान, स्थान, session, कार्ड/IBAN → संवेदनशील
- प्रोजेक्ट में **`flutter_secure_storage` है क्या?** यदि नहीं और संवेदनशील डेटा स्थानीय रूप से लिखा जा रहा है → निष्कर्ष।

**रेड फ़्लैग:**
```
prefs.setString(...token...) / setString(...password...) / setString(...jwt...)
Hive box / SharedPreferences alanları: authToken, refreshToken, accessToken,
    session, jwt, credit, card, iban, phone, email, tckn, otp, pin, apiKey
writeAsString(...) ile dosyaya token/PII yazımı (path_provider dizinlerine)
```

**नियम:**
- संवेदनशील डेटा (token, session, पहचान, भुगतान) → **`flutter_secure_storage`** (iOS Keychain / Android Keystore + EncryptedSharedPreferences)।
- यदि Hive में रहने वाला संवेदनशील डेटा है → **AES से एन्क्रिप्टेड box** (`HiveAesCipher`), एन्क्रिप्शन कुंजी **`flutter_secure_storage`** में रखी जाती है (कोड में एम्बेडेड नहीं होती)।
- स्कोर/पासा/थीम/भाषा/अंतिम टैब जैसा **गैर-संवेदनशील** ऑफ़लाइन डेटा → सादा Hive/SharedPreferences **ठीक है**।
- नोट: Firebase SDK का token को सादा-पाठ में रखना SDK का व्यवहार है; इसके विरुद्ध रक्षा **डिवाइस अखंडता** (App Check / Play Integrity) + **छोटी token अवधि + logout पर token revoke** है (देखो B और C)।
- **Firestore offline persistence cache भी बिना एन्क्रिप्शन के है** (`Settings(persistenceEnabled: true)` — Flutter में डिफ़ॉल्ट चालू)। एप्लिकेशन जो भी Firestore डेटा पढ़ता है वह डिवाइस पर सादी फ़ाइल में जमा होता है। बहुत संवेदनशील collection (संदेश, भुगतान, पहचान) के लिए persistence बंद करना या डिवाइस-लॉक/अतिरिक्त एन्क्रिप्शन परत पर विचार करो; कम-से-कम **logout पर `clearPersistence()`** कॉल हो रहा है या नहीं जाँचो (यदि कॉल नहीं हो रहा तो पिछले उपयोगकर्ता का डेटा डिवाइस पर रह जाता है → साझा डिवाइस पर रिसाव, 🟠)।

Claude Code, सभी स्थानीय स्टोरों में लिखे गए हर फ़ील्ड को सूचीबद्ध करे और **संवेदनशील / गैर-संवेदनशील** के रूप में वर्गीकृत करे।

## A4 — 🔴/🟠 असुरक्षित संचार (OWASP M5) — HTTPS और सर्टिफ़िकेट Pinning

**स्कैन:**
- `http://` से शुरू होने वाले (HTTPS न होने वाले) सभी endpoint → 🔴
- `AndroidManifest.xml` में `android:usesCleartextTraffic="true"` → 🟠
- `res/xml/network_security_config.xml` में `cleartextTrafficPermitted="true"` या ढीले `trust-anchors` → 🟠
- iOS `Info.plist`: `NSAppTransportSecurity` के अंतर्गत `NSAllowsArbitraryLoads = true` (ATS bypass) → 🟠
- Dio/http client में `badCertificateCallback` हमेशा `true` लौटाता है या `HttpClient` `onBadCertificate = (_,__,___) => true` → 🔴 (सर्टिफ़िकेट सत्यापन बंद, MITM के लिए खुला)

**सुझाव:** TLS 1.2+ अनिवार्य, cleartext निषिद्ध। संवेदनशील डेटा संसाधित करने वाले प्रवाहों में **certificate pinning** (सार्वजनिक Wi-Fi पर MITM के विरुद्ध)। Firebase SDK ट्रैफ़िक पहले से ही TLS है; यदि अतिरिक्त `dio`/`http` कॉल हैं तो pinning + interceptor से token जोड़ना, error लॉग में संवेदनशील डेटा न लीक करना।

> **यथार्थवादी अपेक्षा:** Pinning बड़े पैमाने/स्वचालित MITM और नौसिखिए हमलावर को छाँट देता है, पर reFlutter (`libapp.so` patch) और objection (`ios/android sslpinning disable`) तैयार कमांडों से कई pinning implementation को **root/jailbreak के बिना** bypass कर सकते हैं। यानी pinning "मेरा ट्रैफ़िक कभी नहीं देखा जा सकता" की गारंटी नहीं है; **सर्वर को फिर भी हर अनुरोध को स्वयं सत्यापित करना चाहिए** (App Check/Play Integrity + auth)। Pinning जोड़ना अच्छा है, उस पर *भरोसा करना* गलती है।

## A5 — 🟡 संवेदनशील डेटा लॉगिंग / Debug मोड चालू (OWASP M8)

**स्कैन:**
- `print(...)`, `debugPrint(...)`, `developer.log(...)`, `logger.*` में token/पासवर्ड/PII/पूरा response body गुज़रने वाली पंक्तियाँ
- `kReleaseMode`/`kDebugMode` जाँच के बिना production में चलने वाला verbose logging
- Crashlytics/Sentry को custom key/log के रूप में PII भेजना (`setCustomKey`, `recordError`, `log`) — ई-मेल/फ़ोन/token भेजना
- `android/app/build.gradle`: release build में `debuggable true` रह गया हो तो → 🟠
- Dio `LogInterceptor` release में चालू रह गया हो तो (सभी request/response लॉग करता है) → 🟠

**नियम:** Release में लॉग बंद/मास्क किए हों। Crashlytics को कच्चा PII मत भेजो।

## A6 — 🟡 Reverse Engineering / Obfuscation नहीं है (OWASP M7: Insufficient Binary Protections)

**स्कैन:**
- Build script में / CI में `flutter build ... --obfuscate --split-debug-info=<dir>` उपयोग हो रहा है क्या?
- Android `android/app/build.gradle`: `minifyEnabled true` + `shrinkResources true` + R8/ProGuard नियम हैं क्या?

**नोट:** Obfuscation 100% सुरक्षा नहीं है, केवल हमले की लागत बढ़ाता है। **असली नियम: संवेदनशील व्यावसायिक लॉजिक और प्राधिकरण को backend पर ले जाओ, क्लाइंट को अविश्वसनीय मानो।** "101 तक पहुँचा क्या", "कौन-सी ध्वनि बजे" जैसा जोखिम-रहित गेम/UI लॉजिक क्लाइंट पर हो सकता है; पर "यह उपयोगकर्ता premium है क्या / यह लेनदेन वैध है क्या / यह स्कोर वास्तविक है क्या" backend पर सत्यापित होना चाहिए। क्रिटिकल नियंत्रणों को केवल obfuscation पर नहीं, सर्वर सत्यापन पर आधारित करो।

> **ठोस उदाहरण:** यदि obfuscation नहीं है, तो Blutter `libapp.so` से क्लास/मेथड नामों को पठनीय बना देता है और IDA/Ghidra के लिए तैयार symbol script तैयार करता है — कोड लगभग स्रोत-स्तर पर पढ़ा जाता है। यदि obfuscation है तो यह काम कठिन होता है पर असंभव नहीं (नाम चले जाते हैं, लॉजिक रहता है)। इसीलिए `--obfuscate` **होना चाहिए** (विशेषकर यदि क्रिटिकल व्यावसायिक लॉजिक क्लाइंट पर है) पर अकेले पर्याप्त नहीं माना जाना चाहिए।

## A7 — 🟡 अत्यधिक अनुमति (Over-permissioning) + Root/Jailbreak व Tamper Detection

**स्कैन:** `AndroidManifest.xml` और `Info.plist` की अनुमतियाँ बनाम एप्लिकेशन वास्तव में जिन विशेषताओं का उपयोग करता है।
- `ACCESS_FINE_LOCATION` वास्तव में आवश्यक है या `ACCESS_COARSE_LOCATION` (शहर/अनुमानित) पर्याप्त है? (Map/लिस्टिंग विशेषता में **पूर्ण GPS के बजाय अनुमानित स्थान** गोपनीयता और KVKK दोनों दृष्टि से सही है।)
- `READ_SMS`, `READ_CONTACTS`, `CAMERA`, `RECORD_AUDIO`, `MANAGE_EXTERNAL_STORAGE`, बैकग्राउंड स्थान, `QUERY_ALL_PACKAGES` → उपयोग न हो तो हटाओ (स्टोर भी अस्वीकार कर सकता है)।
- हर iOS अनुमति के लिए `Info.plist` में विवरण string (`NSLocationWhenInUseUsageDescription`, `NSCameraUsageDescription` आदि) है क्या? यदि नहीं तो एप्लिकेशन crash होता है + स्टोर अस्वीकृति।
- **Root/jailbreak/emulator detection** (भुगतान/संवेदनशील एप्लिकेशनों में) है क्या? Play Integrity / App Attest / `freerasp` जैसा। यदि नहीं तो संवेदनशील प्रवाहों के लिए 🟡।

> **चेतावनी:** Root/JB detection *अकेले* सुरक्षा नहीं है। Frida/objection से और बहुभाषी समुदायों में घूमने वाले "universal bypass" script से नियमित रूप से bypass किया जाता है; IOSSecuritySuite/freerasp जैसी लाइब्रेरियाँ भी patch की जा सकती हैं। इसका मूल्य: ईमानदार उपयोगकर्ता को जोखिमपूर्ण परिवेश में चेतावनी देना + स्वचालित/बड़े पैमाने के fraud को कठिन बनाना। क्रिटिकल निर्णय (भुगतान अनुमोदन, धन हस्तांतरण) **कभी भी** "डिवाइस साफ़ दिखा" की धारणा पर आधारित नहीं होना चाहिए; सर्वर-साइड सत्यापन + डिवाइस अखंडता संकेत (Play Integrity/App Attest, backend में मूल्यांकित) मुख्य होना चाहिए।

## A8 — 🟠 इनपुट सत्यापन की कमी (OWASP M4)

**नियम:** क्लाइंट सत्यापन bypass किया जा सकता है — **हर सत्यापन सर्वर पर दोहराया जाना चाहिए।** लिस्टिंग टेक्स्ट, उपयोगकर्ता नाम, टिप्पणी, संदेश जैसे UGC फ़ील्ड में:
- लंबाई/फ़ॉर्मैट/प्रकार की सीमा सर्वर पर भी (Firestore Rules `request.resource.data...` / Supabase policy / Edge Function) है क्या?
- यदि WebView उपयोग हो रहा है तो `javascriptMode` और `onNavigationRequest` सुरक्षित हैं क्या? कच्चा HTML/URL इंजेक्शन render हो रहा है क्या (XSS)?
- Deep link / `url_launcher` से खोले गए URL सत्यापित हो रहे हैं क्या (देखो A10)?
- SQL/NoSQL: उपयोगकर्ता इनपुट सीधे query में एम्बेड हो रहा है क्या?
- **GraphQL (यदि प्रोजेक्ट में `graphql`/`graphql_flutter`/`ferry`/`artemis` है; न हो तो लागू नहीं):**
  - Backend में **introspection production में बंद** है क्या? यदि चालू है तो हमलावर पूरे schema, प्रकार और फ़ील्ड नामों को बाहर से पढ़कर हमले की सतह का मानचित्र बना लेता है → 🟠।
  - अत्यधिक गहरे/नेस्टेड (nested) query के विरुद्ध **query depth / complexity limit** है क्या? यदि नहीं तो एक ही गहरे query से DoS + बिल विस्फोट → 🟠।
  - फ़ील्ड-आधारित प्राधिकरण **सर्वर पर** है क्या? Query क्लाइंट से आती है; कौन-सा फ़ील्ड किसे लौटेगा वह backend में सीमित होना चाहिए (क्लाइंट में फ़ील्ड छिपाना प्राधिकरण नहीं है — देखो B4)।

## A9 — 🟡 तृतीय-पक्ष निर्भरताएँ (OWASP M2: Supply Chain)

**स्कैन:**
- `flutter pub outdated` / `dart pub outdated` आउटपुट — ज्ञात CVE वाले/परित्यक्त पैकेज
- `pubspec.yaml` में रखरखाव छोड़ दिए गए, कम डाउनलोड वाले, `git:`/`path:` से खींचे गए पैकेज
- अनपिन किए गए संस्करण (`any`, बहुत विस्तृत `^`/`>=` श्रेणियाँ)
- `pubspec.lock` commit किया गया है क्या? (दोहराने योग्य build के लिए आवश्यक।)
- CI में build चलाने वाला पर अनपिन किया गया action/पैकेज है क्या?
- **Typosquatting (नकली/समान नाम वाला पैकेज):** `pubspec.yaml` के पैकेज नामों में आधिकारिक/लोकप्रिय पैकेजों से जानबूझकर मिलता-जुलता कोई नाम है क्या (जैसे `url_launcher` → `url_launcer`, `firebase_core` → `firebase_cor`, `flutter_secure_storage` → `flutter_secure_storge`)? **हाथ से आँख द्वारा स्कैन अविश्वसनीय है** — हर पैकेज नाम को pub.dev के वास्तविक पैकेज के विरुद्ध सत्यापित करो: verified publisher, डाउनलोड/like संख्या, अंतिम प्रकाशन तिथि और repo पता संगत हैं क्या? नया जोड़ा गया + कम डाउनलोड वाला + असत्यापित publisher वाला + किसी आधिकारिक नाम से बहुत मिलता-जुलता पैकेज → 🟠 जाँचो।

**सुझाव:** मासिक निर्भरता स्कैन, MobSF से स्वचालित स्थैतिक विश्लेषण, `flutter analyze`/`dart analyze` CI में अनिवार्य, `pubspec.lock` version control में।

## A10 — 🟠 Deep Link / URL Launcher / Share सुरक्षा

यह मद `url_launcher`, `app_links`/`uni_links`, `share_plus`, WebView उपयोग करने वाले प्रोजेक्टों के लिए है।

**स्कैन:**
- `launchUrl(...)` कॉल में URL का **स्रोत**: यदि उपयोगकर्ता सामग्री से (Firestore/Supabase से आने वाला लिस्टिंग लिंक, प्रोफ़ाइल लिंक) आ रहा है और scheme सत्यापित नहीं होता → हमलावर `javascript:`, `file:`, `intent:`, फ़िशिंग `https://` लिंक इंजेक्ट कर सकता है → 🟠। केवल `https`/`mailto`/`tel` की allow-list होनी चाहिए।
- **Deep link / App Link (Android) / Universal Link (iOS)** से खुलने वाली स्क्रीन: आने वाले पैरामीटर से सीधे प्राधिकृत कार्रवाई की जाती है क्या? (उदाहरण `myapp://reset?token=...` बिना सत्यापन के संसाधित होता है तो खाता अधिग्रहण का मार्ग।)
- `AndroidManifest.xml` `intent-filter` `android:autoVerify="true"` और `assetlinks.json` सही हैं क्या? (App Link hijack के विरुद्ध।)
- `share_plus` से साझा की गई सामग्री में **PII/token रिसाव** है क्या? ("गेम अंत कार्ड" जैसी छवियाँ आमतौर पर जोखिम-रहित होती हैं; पर साझा टेक्स्ट में ई-मेल/फ़ोन/deep link + token न हो।)
- WebView: `Uri` सत्यापन, `onNavigationRequest` allow-list, `allowFileAccess`/`allowUniversalAccessFromFileURLs` बंद।
- **WebView ↔ Dart JS ब्रिज:** यदि `addJavaScriptChannel(...)` / `JavascriptChannel` से web सामग्री को native/Dart पक्ष पर संदेश भेजने का चैनल खुलता है, तो WebView के अंदर चलने वाला (संभावित रूप से हमलावर-नियंत्रित) JS इस चैनल को कॉल कर सकता है → यदि ब्रिज फ़ंक्शन **असत्यापित इनपुट से संवेदनशील कार्रवाई/डेटा पहुँच trigger करता है** तो 🟠। ब्रिज तक केवल विश्वसनीय origin से पहुँच + आने वाले संदेश का कड़ा सत्यापन आवश्यक है।

## A11 — 🟠 कमज़ोर / गलत क्रिप्टोग्राफ़ी (OWASP M10: Insufficient Cryptography)

यदि प्रोजेक्ट में कोई भी एन्क्रिप्शन/hash/signing है (पैकेज: `crypto`, `encrypt`, `pointycastle`, `cryptography` आदि):

**रेड फ़्लैग:**
```
md5(...) / sha1(...)                    → parola/bütünlük için 🔴 (kırık algoritmalar)
AESMode.ecb / 'AES/ECB'                 → 🔴 (desenleri sızdırır; GCM/CBC+HMAC kullan)
IV/nonce sabit veya tekrar kullanılıyor → 🔴
Encrypter(Key.fromUtf8('sabitanahtar')) → 🔴 koda gömülü şifreleme anahtarı (A1 ile aynı sınıf)
Random() ile token/anahtar/OTP üretimi  → 🟠 (kriptografik iş için Random.secure() şart)
base64/XOR/Caesar "şifreleme" olarak    → 🔴 (encoding ≠ encryption)
kendi yazılmış özel şifreleme algoritması → 🔴 (asla custom crypto)
parola hash'i istemcide/DB'de düz veya salt'sız → 🔴 (Firebase/Supabase Auth kullanılıyorsa zaten onlar halleder; custom auth'ta bcrypt/scrypt/Argon2)
```

**नियम:**
- एन्क्रिप्शन कुंजी कभी कोड/asset में नहीं रहती → Keystore/Keychain (`flutter_secure_storage`) या सर्वर से व्युत्पन्न की जाती है।
- यदि hash आवश्यक है तो SHA-256+; पासवर्ड के लिए bcrypt/scrypt/Argon2 (अकेला SHA भी पर्याप्त नहीं)।
- `HiveAesCipher` कुंजी `Hive.generateSecureKey()` से उत्पन्न करके secure storage में संग्रहीत होनी चाहिए (A3 से संबंधित)।
- नोट: यदि प्रोजेक्ट में कोई मैनुअल क्रिप्टो नहीं है तो यह मद "लागू नहीं" — Firebase/Supabase SDK अपने TLS/token क्रिप्टो को स्वयं सही प्रबंधित करते हैं; **अपना क्रिप्टो कोड न लिखना ही सबसे अच्छी प्रथा है**, इसे निष्कर्ष के रूप में नहीं बल्कि सकारात्मक टिप्पणी के रूप में लिखो।

## A12 — 🟡 स्क्रीन / क्लिपबोर्ड / कीबोर्ड / मेमोरी के माध्यम से डेटा रिसाव (OWASP M6: Inadequate Privacy Controls)

संवेदनशील स्क्रीन वाले (भुगतान, पहचान, निजी संदेश, बैलेंस) एप्लिकेशनों में:

**स्कैन:**
- **स्क्रीनशॉट/रिकॉर्डिंग रोक:** Android में संवेदनशील स्क्रीनों पर `FLAG_SECURE` (या `screen_protector`/`no_screenshot` जैसा पैकेज) है क्या? यदि नहीं तो स्क्रीन रिकॉर्ड करने वाली दुर्भावनापूर्ण/एक्सेसिबिलिटी सेवा सब कुछ देखती है → संवेदनशील एप्लिकेशन में 🟡/🟠।
- **App switcher snapshot:** iOS बैकग्राउंड में जाते समय स्क्रीन की फ़ोटो लेता है और app switcher में दिखाता है; Android "Recents" वही करता है। संवेदनशील स्क्रीन बैकग्राउंड में जाते समय मास्क होती है क्या (`AppLifecycleState.paused/inactive` में overlay/blur)? → न हो तो 🟡।
- **क्लिपबोर्ड (clipboard):** `Clipboard.setData(...)` से token/पासवर्ड/IBAN/व्यक्तिगत डेटा कॉपी हो रहा है क्या? Android में अन्य एप्लिकेशन (13 से पहले) क्लिपबोर्ड पढ़ सकते हैं; यदि कॉपी करना आवश्यक है तो समयबद्ध सफ़ाई पर विचार किया जाना चाहिए → 🟡।
- **कीबोर्ड कैश / autofill:** संवेदनशील फ़ील्ड में (`TextField`) `obscureText: true` + `enableSuggestions: false` + `autocorrect: false` सेट किया गया है क्या? तृतीय-पक्ष कीबोर्ड टाइप किए गए को सीखकर कैश करते हैं; पासवर्ड फ़ील्ड में `autofillHints: [AutofillHints.password]` सही है क्या? → कमी हो तो 🟡।
- **नोटिफ़िकेशन पूर्वावलोकन:** लॉक स्क्रीन पर संवेदनशील सामग्री दिखाने वाला नोटिफ़िकेशन है क्या (देखो C6 FCM)?
- **Logout के बाद मेमोरी में शेष संवेदनशील state (🟡):** `signOut()` कॉल होने पर केवल UI redirect किया जाता है, या मेमोरी में संवेदनशील state (जैसे `UserBloc`, `WalletState`, `ProfileNotifier`, token/PII रखने वाले `riverpod`/`provider`/`bloc` object) **स्पष्ट रूप से रीसेट / `close()` / `dispose()` किए जाते हैं** क्या? यदि रीसेट नहीं होते, तो **साझा डिवाइस पर** (एप्लिकेशन प्रोसेस के रूप में जीवित रहते हुए खाता बदलने पर) अगला उपयोगकर्ता पिछले उपयोगकर्ता का डेटा स्क्रीन/state में देख सकता है → 🟡। *यथार्थवादी नोट:* Dart में मेमोरी को गारंटीशुदा तरीके से मिटाना संभव नहीं है (GC + string immutability); इसीलिए यह "memory dump के विरुद्ध सुरक्षा" नहीं, बल्कि **session स्वच्छता** की मद है — असली खतरा RAM dump करने वाला rooted हमलावर नहीं, बल्कि उसी डिवाइस का उपयोग करने वाला अगला व्यक्ति है। महत्वपूर्ण: logout पर state tree साफ़ करो + `clearPersistence()` (A3) + secure storage का session डेटा मिटाओ।

## A13 — 🟠 MethodChannel / Native ब्रिज कमज़ोरियाँ (Kotlin/Swift) — Native Injection व Path Traversal

यह मद तब लागू होती है जब प्रोजेक्ट में कस्टम `MethodChannel` / `EventChannel` / `BasicMessageChannel` या हाथ से लिखा गया native (Kotlin/Java, Swift/Obj-C) कोड है। केवल तैयार Firebase/Supabase SDK उपयोग करने वाले, बिना किसी कस्टम platform चैनल वाले प्रोजेक्ट में → **लागू नहीं।**

**स्कैन:**
- Dart पक्ष पर `MethodChannel('...')`, `invokeMethod(...)`, `EventChannel(...)` कॉल खोजो; उनके समकक्ष Android (`android/app/src/main/kotlin|java/.../*.kt|*.java`, `configureFlutterEngine` के अंदर `MethodChannel(...).setMethodCallHandler`) और iOS (`ios/Runner/*.swift`, `FlutterMethodChannel(...)`) पक्ष पर मिलाओ।
- क्लाइंट से आने वाले arguments (`call.arguments`, `call.argument("x")`) native पक्ष पर **बिना सत्यापन के** इनमें बहते हैं क्या:
  - SQL/SQLite query (string जोड़ → **SQL injection**),
  - फ़ाइल पथ (`File(path)`, `FileInputStream`, `NSFileManager` → **path traversal**; `../../` से एप्लिकेशन sandbox के बाहर निकलना),
  - `Intent` (Android) पैरामीटर / `startActivity` / `PendingIntent` (→ intent injection, अनधिकृत component trigger),
  - `Runtime.exec` / `ProcessBuilder` (→ command injection),
  - `WebView.loadUrl` / `evaluateJavascript` (→ XSS / JS ब्रिज दुरुपयोग)।

**रेड फ़्लैग:**
```
// Android (Kotlin)
db.rawQuery("SELECT * FROM t WHERE id = " + call.argument<String>("id"), null)  → SQL injection
File(context.filesDir, call.argument<String>("name")!!)   // ../ / canonical kontrolü yok → path traversal
Runtime.getRuntime().exec(call.argument<String>("cmd"))   → komut enjeksiyonu
// iOS (Swift)
let p = docsDir.appendingPathComponent(args["name"] as! String)  // prefix/canonical kontrolü yok
```

**नियम (रिपोर्ट के मूल दर्शन के साथ):** Native कोड भी **क्लाइंट का हिस्सा है** — वहाँ किया गया सत्यापन भी रिवर्स-इंजीनियरिंग/hook से bypass किया जा सकता है। यानी दोनों परतें एक साथ आवश्यक हैं:
1. Native पक्ष पर injection/path-traversal के विरुद्ध **सही लेखन** फिर भी आवश्यक है (parameterized query, path canonicalization + sandbox root `startsWith` जाँच, explicit `Intent`, इनपुट allow-list) — यह क्लाइंट-आंतरिक सुरक्षा स्वच्छता है।
2. **लेकिन अधिकार/स्वामित्व/भुगतान निर्णय यहाँ नहीं लिया जाता;** वह निर्णय backend में सत्यापित होता है। MethodChannel को "सुरक्षित आंतरिक API" मत समझो — हमलावर native फ़ंक्शन को सीधे भी (hook से) कॉल कर सकता है। चैनल से आने वाले "यह उपयोगकर्ता प्राधिकृत है" जैसे दावे पर भरोसा नहीं किया जाता।

---

# खंड B — प्रमाणीकरण प्रवाह (Login / पंजीकरण / Session)

यह खंड OWASP M3 (Insecure Authentication/Authorization) है। Firebase Auth / Supabase Auth / custom चाहे जो भी हो, **login प्रवाह स्वयं** सबसे अधिक शोषित सतह है। नीचे दिए गए बिंदु प्रोजेक्ट में जो भी provider हो उसी के अनुसार लागू होते हैं।

## B1 — 🟠 ई-मेल Enumeration सुरक्षा (Firebase Auth)

Firebase Auth/Google Identity Platform में `createAuthUri` / `fetchSignInMethodsForEmail` किसी ई-मेल के पंजीकृत होने या न होने की जानकारी लीक कर देते हैं → हमलावर लीक हुई ई-मेल सूची को "इस platform पर किसका खाता है" के हिसाब से छाँट लेता है → लक्षित फ़िशिंग + credential stuffing। Google ने **15 सितंबर 2023** के बाद बनाए गए प्रोजेक्टों में **email enumeration protection** को डिफ़ॉल्ट चालू किया; `fetchSignInMethodsForEmail` को deprecate कर दिया गया।

**स्कैन:**
- कोड में `fetchSignInMethodsForEmail(...)` का उपयोग है क्या? यदि है → auth प्रवाह इस deprecate/बंद किए गए व्यवहार पर निर्भर है, 🟠 (टूटता है + enumeration जोखिम)। "पहले ई-मेल दर्ज करो, login है या पंजीकरण यह तय करो" pattern यदि इस API पर आधारित है तो पुनः डिज़ाइन किया जाना चाहिए।
- **मैनुअल सत्यापन:** Firebase Console → Authentication → Settings → "Email enumeration protection" चालू है क्या? (पुराने प्रोजेक्टों में मैनुअल रूप से चालू करना होगा।)

## B2 — 🟠 सामाजिक लॉगिन में मौन खाता निर्माण व Provider मिश्रण (Google/Apple)

क्रिटिकल व्यवहार: `signInWithCredential` / `signInWithPopup` से Google/Apple/Facebook लॉगिन में, यदि मेल खाता खाता नहीं है तो Firebase **मौन रूप से नया खाता बना देता है**। `isNewUser` (AdditionalUserInfo) केवल पहली बार निर्माण पर लौटता है। Bumble/समान उदाहरणों में: हमलावर पीड़ित की **अभी तक अपंजीकृत** ई-मेल से पहले खाता खोलकर बाद में उसे अधिग्रहीत कर सकता है।

**स्कैन (यदि प्रोजेक्ट में `google_sign_in` / `sign_in_with_apple` / Firebase social है):**
- Google/Apple लॉगिन प्रवाह में `account-exists-with-different-credential` error को **हैंडल किया जाता है** क्या? यदि नहीं तो एक ही ई-मेल के दो अलग provider खातों का प्राधिकरण भ्रम पैदा करता है → 🟠। (Firebase की "One account per email address" सेटिंग + credential linking प्रवाह होना चाहिए।)
- Apple से लॉगिन: Apple **relay/private ई-मेल** दे सकता है और **नाम/ई-मेल केवल पहली बार** लौटाता है। यदि कोड बाद के लॉगिन में इस जानकारी की फिर से अपेक्षा करता है तो टूटता है; पहली बार में ही सहेजा जाना चाहिए।
- Apple Sign-In की **अनिवार्यता:** यदि एप्लिकेशन कोई अन्य सामाजिक लॉगिन (Google/Facebook) प्रदान करता है तो App Store Guideline 4.8 / 5.1.1 के अनुसार **Apple से Sign-In भी प्रदान किया जाना चाहिए** (न हो तो स्टोर अस्वीकृति)। प्रोजेक्ट में Google है Apple नहीं → 🟡 (स्टोर जोखिम)।
- `google_sign_in` के नए संस्करण पुराने संस्करणों से भिन्न API रखते हैं (`GoogleSignIn.instance`, `authenticate()`, `authorizationClient`)। यदि पुराने `signIn()` pattern से migration अधूरा है तो compile/runtime error + प्रवाह टूटना होता है — उपयोग किए गए संस्करण और API संगतता को जाँचो।

## B3 — 🟠 Session / Token प्रबंधन

**स्कैन:**
- **Logout पर token revoke:** यदि निकास पर केवल स्थानीय state साफ़ होती है पर `signOut()` कॉल नहीं होता / refresh token रद्द नहीं होता, तो डिवाइस पर बचे token से session जारी रखी जा सकती है → 🟠। Firebase में `FirebaseAuth.instance.signOut()`, संवेदनशील परिदृश्य में अतिरिक्त रूप से सर्वर-साइड token revoke। (Logout पर मेमोरी में मौजूद संवेदनशील state भी साफ़ हुई है या नहीं यह A12 के साथ जाँचो।)
- **Refresh token स्थायित्व:** Refresh token, रद्द होने तक अनंत idToken उत्पन्न करता रहता है। संवेदनशील एप्लिकेशन में निकास/पासवर्ड परिवर्तन पर revoke होना चाहिए।
- **ई-मेल सत्यापन:** संवेदनशील कार्रवाइयाँ `emailVerified == true` की शर्त से बंधी हैं क्या? यदि नहीं तो असत्यापित ई-मेल से अधिकार मिल जाता है।
- **MFA:** उच्च-मूल्य वाले खातों (भुगतान, admin) में बहु-कारक प्रमाणीकरण है क्या? (Firebase → Identity Platform अपग्रेड आवश्यक है।)
- **अनाम auth:** केवल अस्थायी state के लिए उपयोग होना चाहिए; स्थायी खाते का स्थान नहीं लेना चाहिए; अनाम उपयोगकर्ता भी security rules से सीमित होने चाहिए ("anyone can create anonymous account")।
- **Brute force / rate limit:** Login प्रयासों पर सीमा है क्या? (Firebase आंशिक रूप से स्वचालित; custom auth में मैनुअल आवश्यक।)
- **पासवर्ड नीति:** न्यूनतम लंबाई/जटिलता, लीक हुए पासवर्ड की जाँच (Identity Platform password policy) चालू है क्या?

## B4 — 🟠 प्राधिकरण ≠ प्रमाणीकरण (Broken Access Control)

OWASP Web Top 10 2021/2025 में **#1 Broken Access Control**। मोबाइल में भी असली छेद यहीं है।
- "लॉगिन किया" (authenticated) और "यह कार्रवाई कर सकता है" (authorized) में गड़बड़ी हो रही है क्या? भूमिका/स्वामित्व जाँच **सर्वर पर** (rules/policy/function) हो रही है, या केवल क्लाइंट में UI छिपाकर? क्लाइंट में छिपाना सुरक्षा नहीं है।
- Route guard (go_router `redirect` / auto_route guard) केवल UI की रक्षा करता है; **डेटा पहुँच को backend नियम से** सुरक्षित करना चाहिए।
- IDOR: दूसरे उपयोगकर्ता की `id` से document/record खींचा जा सकता है क्या? (जैसे `/users/{başkasınınId}` क्लाइंट से माँगा जा सकता है और rule स्वामित्व जाँच नहीं करता → 🔴।)

---

# खंड C — BACKEND (FIREBASE + SUPABASE)

मोबाइल उल्लंघनों का बड़ा हिस्सा क्लाइंट में नहीं, बल्कि **गलत कॉन्फ़िगर किए गए backend नियमों में** होता है। 2024 में एक ही अनुसंधान ने 916 Firebase साइटों पर ~12.5 करोड़ record (जिनमें से 1.9 करोड़ सादा-पाठ पासवर्ड) खुले पाए। Supabase पक्ष पर उल्लंघनों का बड़ा हिस्सा RLS के बंद छोड़ दिए जाने से उत्पन्न होता है।

> नोट: प्रोजेक्ट में इन सेवाओं में से जो भी हो उसका उप-खंड लागू होता है; जो न हो उसे **"लागू नहीं"** चिह्नित किया जाता है।

## C1 — 🔴 Firebase Firestore / Realtime DB / Storage सुरक्षा नियम

**Claude Code ये फ़ाइलें पढ़े:** `firestore.rules`, `database.rules.json`, `storage.rules`, `firebase.json` (कौन-सी rules फ़ाइल deploy की गई है यह `firebase.json` दिखाता है)।

**सबसे खतरनाक pattern (test mode production में भाग गया):**
```
allow read, write: if true;         → 🔴 herkes her şeyi okur/yazar
match /{document=**} { allow read, write: if true; }  → 🔴 tüm DB açık
".read": true, ".write": true        → 🔴 (Realtime DB)
```
Firestore की "test mode" rules **30 दिन बाद बंद हो जाती हैं** पर अवधि बढ़ाकर production में ले जाई गई हो सकती है; साथ ही `allow read, write: if request.time < timestamp.date(2025,...)` जैसा तिथि-आधारित अस्थायी नियम production में → 🔴।

**दूसरी सबसे आम गलती — "लॉगिन करने वाला हर कोई हर चीज़ तक पहुँचता है":**
```
allow read, write: if request.auth != null;   → 🟠
```
`request.auth != null` केवल "लॉगिन किया है" कहता है, **किस डेटा** तक पहुँच सकता है यह सीमित नहीं करता। दूसरे उपयोगकर्ता ने भी लॉगिन किया होता है।

**सही pattern (स्वामित्व + डेटा सत्यापन):**
```
match /users/{userId} {
  allow read, update, delete: if request.auth != null && request.auth.uid == userId;
  allow create: if request.auth != null && request.auth.uid == userId;
}
match /posts/{postId} {
  allow read: if true;                                        // ilanlar herkese açık okunur
  allow create: if request.auth != null
                && request.resource.data.authorId == request.auth.uid
                && request.resource.data.title.size() < 200;  // sunucu tarafı validation
  allow update, delete: if request.auth != null
                && resource.data.authorId == request.auth.uid; // sadece sahibi
}
```

**Claude Code हर collection के लिए पूछे:**
1. `allow ... if true` या तिथि-आधारित अस्थायी नियम है क्या? → 🔴
2. केवल `auth != null` है, या स्वामित्व/UID मिलान है? → न हो तो 🟠
3. `update`/`delete` मालिक तक सीमित है क्या? (दूसरे की लिस्टिंग/document बदल पाना = डेटा अखंडता + खाता अधिग्रहण का मार्ग)
4. `create`/`update` में **फ़ील्ड सत्यापन** है क्या (`request.resource.data...` प्रकार/लंबाई, अपरिवर्तनीय फ़ील्ड `resource.data.x == request.resource.data.x`)? उपयोगकर्ता `role: admin` या `isPremium: true` स्वयं लिख सकता है क्या → 🔴 अधिकार वृद्धि।
5. **`get` और `list` (query) के नियम अलग हैं।** एक दस्तावेज़ को ID से पढ़ने की अनुमति देने वाला नियम, collection को **query (query/list) करने** की स्वतः अनुमति नहीं देता; जहाँ query की जाती है वहाँ नियम को query द्वारा लौटाए जा सकने वाले सभी दस्तावेज़ों के लिए संतुष्ट होना चाहिए। `list`/query पहुँच स्वामित्व/filter से सीमित है, या "auth != null वाला हर कोई पूरी collection query करता है"? → असीमित query 🟠/🔴।
6. **Storage नियम अलग हैं** (`storage.rules`) — Firestore सुरक्षित होने पर भी Storage खुला हो सकता है। फ़ाइल आकार (`request.resource.size < 5*1024*1024`) और प्रकार (`request.resource.contentType.matches('image/.*')`) की सीमा है क्या? यदि नहीं तो कोटा दुरुपयोग/दुर्भावनापूर्ण फ़ाइल।
7. लेखन **दर सीमा** / abuse सुरक्षा है क्या? (हमलावर कचरा डेटा लिखकर बिल/सीमा को उड़ा सकता है।)

**अतिरिक्त परत — App Check (🟠, बहुत महत्वपूर्ण):** Firestore/Storage/RealtimeDB/Functions/Auth के लिए **App Check** चालू है क्या? यह सत्यापित करता है कि अनुरोध तुम्हारे वास्तविक, अपरिवर्तित एप्लिकेशन से आया है (Android: Play Integrity, iOS: App Attest); script से भेजे गए नकली अनुरोधों को छाँट देता है। कोड में `FirebaseAppCheck.instance.activate(...)` है क्या देखो। यदि नहीं तो security rules सही होने पर भी `anon`/स्वचालित अनुरोध खुले हैं → 🟠।

## C2 — 🟡 Firebase Remote Config सुरक्षा

यदि `firebase_remote_config` (जैसे अनिवार्य संस्करण / feature flag) उपयोग हो रहा है:
- **Remote Config एक रहस्य भंडार नहीं है।** मान क्लाइंट तक पहुँचते हैं और पढ़े जा सकते हैं; API कुंजी/secret/URL रखना → 🔴।
- **Force-update / feature gate क्लाइंट में bypass किया जा सकता है।** "इस संस्करण से नीचे वालों को लॉक करो" का लॉजिक UX के लिए है; **सुरक्षा निर्णय** (premium है क्या, पहुँच सकता है क्या) इस पर आधारित नहीं होना चाहिए — सर्वर पर सत्यापित होना चाहिए।
- यदि Remote Config की शर्तें PII (पूर्ण ई-मेल/फ़ोन) से लक्ष्यीकरण करती हैं तो गोपनीयता नोट।

## C3 — 🔴 Supabase Row Level Security (RLS)

Supabase में **स्वर्णिम नियम: `public` schema की हर table में RLS चालू होना चाहिए, बिना अपवाद।** `anon` key सार्वजनिक है; यदि RLS बंद है तो उस key से हर कोई पूरी table पढ़ता/लिखता/मिटाता है।

**Claude Code ये स्कैन करे:**
- `supabase/migrations/*.sql`, `schema.sql`, `.sql` फ़ाइलें
- `CREATE TABLE` है पर उसके बाद `ENABLE ROW LEVEL SECURITY` **नहीं है** → 🔴
- SQL editor/migration से बनाई गई tables RLS को **स्वतः चालू नहीं करतीं** (dashboard के विपरीत)। यदि migration में स्पष्ट रूप से नहीं है → 🔴

**जाँच query (मैनुअल, उपयोगकर्ता चलाए):**
```sql
SELECT tablename FROM pg_tables
WHERE schemaname = 'public' AND NOT rowsecurity;
```
लौटने वाली हर table = असुरक्षित = 🔴।

**RLS चालू पर policy गलत pattern:**
```sql
-- 🔴 "izin veriyormuş gibi görünen ama her şeyi açan" politika
CREATE POLICY "read" ON profiles FOR SELECT USING (true);

-- ✅ doğru: sadece kendi satırı
CREATE POLICY "read own" ON profiles FOR SELECT USING (auth.uid() = user_id);
```

**अन्य क्रिटिकल जाँचें:**
1. **`service_role` key क्लाइंट में है क्या?** (`--dart-define`, कोड में, `NEXT_PUBLIC_*`) → 🔴, RLS को पूरी तरह bypass कर देती है।
2. **`user_metadata` claim पर आधारित policy** → 🔴। इस फ़ील्ड को उपयोगकर्ता स्वयं बदल सकता है; यदि अधिकार जाँच में उपयोग हो तो अधिकार वृद्धि। `app_metadata` उपयोग होना चाहिए।
3. **यदि UPDATE policy में `WITH CHECK` नहीं है** → उपयोगकर्ता row को दूसरे उपयोगकर्ता/tenant में स्थानांतरित कर सकता है। Multi-tenant में `tenant_id`/`org_id` को `WITH CHECK` से सुरक्षित करना चाहिए।
4. **RPC / Edge Function `anon` से कॉल किए जा सकते हैं क्या?** यदि Admin/license/भुगतान फ़ंक्शन सार्वजनिक रूप से कॉल किए जा सकते हैं → 🔴।
5. **Storage bucket:** `public` चिह्नित bucket = हर कोई सभी फ़ाइलें पढ़ता है। Bucket policy और फ़ाइल-पथ डिज़ाइन (path में tenant/uid) जाँचे जाने चाहिए।
6. **Edge Function में `service_role` रिसाव:** शुरुआत में सारा `env` लॉग करना, error संदेश में connection/secret लौटाना।

## C4 — 🟠 Firebase + Supabase (या दोहरा backend) मिश्रित उपयोग जोखिम

यदि एक एप्लिकेशन में **Firebase Auth और Supabase DB दोनों** (या इसका उलटा) हैं: Supabase RLS policy Firebase पहचान को कैसे सत्यापित करती हैं? यदि दो सिस्टमों के बीच पहचान-सेतु (JWT सत्यापन, custom claims) गलत है, तो एक ओर authenticated उपयोगकर्ता दूसरी ओर अनधिकृत रूप से पहुँच सकता है। **पहचान एकल स्रोत (single source of truth) से** प्रबंधित होनी चाहिए; JWT signature सत्यापन (audience/issuer) पूर्ण होना चाहिए।

## C5 — 🟠 Cloud Functions / Edge Functions सुरक्षा

यदि प्रोजेक्ट में `functions/` या `supabase/functions/` है:
- फ़ंक्शन पहचान/अधिकार सत्यापित करते हैं, या "कॉल हुआ तो चलेगा"? Callable न होने वाले HTTP फ़ंक्शन में auth जाँच हाथ से करनी होती है।
- इनपुट सत्यापन, rate limit, `maxInstances` (बिल उड़ाने वाले DOS के विरुद्ध) सेट है क्या?
- **रहस्य Vault/Secret Manager में हैं या सादे env में?** पर्यावरण चर (env var) भी फ़ंक्शन लॉग, error trace या `console.log(process.env)` जैसी पंक्ति में लीक हो सकता है। वास्तविक रहस्य **Google Cloud Secret Manager / Firebase secrets (`defineSecret`) / Supabase Vault** में रखकर फ़ंक्शन को **संदर्भ द्वारा** दिए जाने चाहिए; स्रोत कोड में या सादे `.env` में नहीं। शुरुआत में सारा `env` लॉग करने वाला कोड → 🔴।
- CORS सेटिंग बहुत ढीली है क्या (`*`)?
- Error response में stack trace / connection जानकारी लीक हो रही है क्या?

## C6 — 🟠 Push Notification (FCM) सुरक्षा

यदि प्रोजेक्ट में `firebase_messaging` है:
- **Payload में संवेदनशील डेटा है क्या?** नोटिफ़िकेशन सामग्री लॉक स्क्रीन पर दिखती है, Google/Apple सर्वरों से गुज़रती है और डिवाइस पर लॉग की जा सकती है। OTP/token/बैलेंस/निजी संदेश सामग्री **payload में नहीं रखी जाती**; नोटिफ़िकेशन "तुम्हारा नया संदेश है" कहता है, सामग्री एप्लिकेशन खुलने पर सुरक्षित चैनल से खींची जाती है (data-only push + fetch pattern)। → उल्लंघन हो तो 🟠।
- **Topic सुरक्षा:** FCM topic को **कोई भी subscribe कर सकता है** — यदि क्लाइंट में `subscribeToTopic('...')` का नाम अनुमान-योग्य है (जैसे `user_12345`, `admin`, `premium`) तो हमलावर भी subscribe कर लेता है और उस topic पर जाने वाला हर नोटिफ़िकेशन प्राप्त करता है। **उपयोगकर्ता/समूह-विशिष्ट डेटा topic से नहीं, token-आधारित (सर्वर से एकल डिवाइस को) भेजा जाना चाहिए।** → यदि topic के माध्यम से व्यक्ति-विशिष्ट सामग्री भेजी जा रही है तो 🟠।
- **FCM token के संग्रहण का स्थान:** Token आमतौर पर Firestore में `users/{uid}/fcmToken` जैसा रखा जाता है। यदि इस फ़ील्ड को **कोई और पढ़/लिख सकता है** (C1 नियम देखो) तो हमलावर token बदलकर पीड़ित के नोटिफ़िकेशन अपने डिवाइस पर मोड़ सकता है → 🟠।
- **सर्वर-साइड भेजना:** Legacy Server Key (स्थायी, लीक हो तो सबको push भेजा जा सकता है) उपयोग हो रहा है, या **HTTP v1 API + service account (छोटी अवधि का OAuth token)**? यदि Legacy key कोड/क्लाइंट में एम्बेडेड है → 🔴। (Google ने legacy FCM API बंद कर दिया; यदि अभी भी उपयोग हो रहा है तो वह पहले से ही टूटा हुआ है।)
- यदि नोटिफ़िकेशन पर क्लिक होने पर deep link संसाधित होता है तो A10/D2 सत्यापन लागू हैं।

## C7 — 🟠 इन-ऐप खरीद / Premium सत्यापन + बिल व दुरुपयोग निगरानी

**IAP / premium (यदि प्रोजेक्ट में `in_app_purchase`, RevenueCat, `purchases_flutter` या `isPremium`/`isPro` जैसा फ़ील्ड है):**
- Premium स्थिति **कहाँ निर्धारित होती है?** यदि क्लाइंट "मैंने खरीदा" कहकर Firestore/Supabase में `isPremium: true` लिख सकता है → 🔴 (C1/C3 मद 4 वाली अधिकार वृद्धि)। सही तरीका: **रसीद (receipt/purchase token) सर्वर पर सत्यापित होती है** (Cloud Function → Google Play Developer API / App Store Server API या RevenueCat webhook) और premium फ़ील्ड को **केवल सर्वर** लिखता है; security rules क्लाइंट को इस फ़ील्ड को लिखने से रोकती हैं।
- यदि उत्पाद मूल्य/अधिकार जानकारी क्लाइंट में स्थिर है तो उसे बदला जा सकता है; अधिकार-प्रदान हमेशा सर्वर सत्यापन पर आधारित होना चाहिए।

**बिल और दुरुपयोग निगरानी (मैनुअल सत्यापन):**
- GCP **बजट चेतावनी** (budget alert) परिभाषित है क्या? Firestore/Functions/Storage के लिए **उपयोग अलार्म** है क्या? हमला (कोटा दुरुपयोग, कचरा लेखन, अनंत लूप फ़ंक्शन) अधिकांशतः पहले बिल में दिखता है; यदि अलार्म नहीं है तो सप्ताहों तक पता नहीं चलता।
- Cloud Functions में `maxInstances` सीमा (C5) + Firestore में असामान्य read/write चेतावनी पर विचार किया जाना चाहिए।
- Firebase आधिकारिक सुरक्षा checklist DOS संदेह पर सपोर्ट से संपर्क की सिफ़ारिश करती है — घटना प्रतिक्रिया चरण के रूप में नोट करो।

## C8 — 🟠/🔴 क्लाइंट से आए "दावा किए गए" डेटा पर भरोसा करना (स्थान / स्कोर / समय / काउंटर हेरफेर)

स्थान-आधारित, gamification, loyalty/अंक, step-counter, "check-in", दूरी/पुरस्कार वाले हर प्रवाह में लागू। मूल गलती: **क्लाइंट द्वारा भेजे गए lat/lng, स्कोर, timestamp, step संख्या, दूरी जैसे मानों को backend द्वारा आँख बंद करके स्वीकार करना।** ये सब क्लाइंट में उत्पन्न होते हैं → **इन सबको जाली बनाया जा सकता है** (Fake GPS / mock location, packet संपादित करके स्कोर फुलाना, डिवाइस घड़ी बदलना)।

**स्कैन / प्रश्न:**
- क्लाइंट backend को कच्चा स्थान/स्कोर/समय भेजता है और backend इसे सीधे लिख देता है क्या (Firestore/Supabase में `score`, `distance`, `location`)? → सत्यापन न हो तो 🟠; यदि इस मान से premium/पुरस्कार/रैंकिंग निर्धारित होती है तो 🔴।
- **Velocity / jump check** सर्वर पर है क्या? दो क्रमागत स्थानों के बीच की दूरी/समय भौतिक रूप से संभव है क्या (जैसे 2 मिनट में 400 किमी = असंभव)? अचानक छलाँग अस्वीकृत होती है क्या?
- Fake GPS / location mocking को **केवल क्लाइंट के plugin से** (`isMockLocation` flag) रोकने का प्रयास किया जा रहा है क्या? यह क्लाइंट संकेत **अकेले पर्याप्त नहीं है** (bypass किया जा सकता है) — सर्वर-साइड संगति जाँच (गति, भौगोलिक सीमा, सर्वर timestamp) मुख्य होनी चाहिए।
- Timestamp: क्रिटिकल प्रवाहों में **सर्वर समय** (`FieldValue.serverTimestamp()` / DB `now()`) उपयोग हो रहा है, या क्लाइंट द्वारा भेजा गया समय? क्लाइंट समय पर भरोसा करना → रैंकिंग/पुरस्कार हेरफेर।
- स्कोर/प्रगति: यदि अंतिम स्कोर क्लाइंट में गणना करके भेजा जाता है तो उसे बदला जा सकता है; यदि क्रिटिकल है तो सर्वर सत्यापन / पुनर्गणना आवश्यक है।

**नियम:** यह A8 (इनपुट सत्यापन) और B4 (प्राधिकरण) के साथ एक ही परिवार से है: **क्लाइंट द्वारा कहा गया कोई भी "तथ्य" बिना सत्यापन के वास्तविक नहीं माना जाता।** Backend को दावा किए गए डेटा को अपने स्वतंत्र संकेतों (सर्वर समय, पिछली स्थिति, भौतिक तर्क) से संगति की दृष्टि से जाँचना चाहिए।

---

# खंड D — ANDROID-विशिष्ट

`android/` फ़ोल्डर वाले हर प्रोजेक्ट में। फ़ाइलें: `android/app/src/main/AndroidManifest.xml`, `android/app/build.gradle(.kts)`, `android/app/proguard-rules.pro`, `res/xml/network_security_config.xml`।

## D1 — 🟠 Manifest कॉन्फ़िगरेशन
- `android:debuggable="true"` release में → 🔴 (आमतौर पर build type से आता है, यदि हाथ से सेट है तो देखो)।
- `android:allowBackup="true"` (डिफ़ॉल्ट) → ADB backup से एप्लिकेशन डेटा खींचा जा सकता है। यदि संवेदनशील डेटा है तो `false` या `fullBackupContent` से बाहर रखो → 🟡/🟠। **Android 12+ (API 31) के लिए `android:dataExtractionRules` अलग से परिभाषित होना चाहिए** — `fullBackupContent` अकेले नए डिवाइस-से-डिवाइस स्थानांतरण (device transfer) परिदृश्य को कवर नहीं करता; token/session फ़ाइलों को दोनों नियम-सेटों में `exclude` किया जाना चाहिए।
- **Tapjacking:** संवेदनशील पुष्टि बटन (भुगतान, अनुमति, हटाना) वाली स्क्रीनों पर स्पर्श overlay से धोखा दिया जा सकता है। क्रिटिकल native view में `filterTouchesWhenObscured` / Flutter पक्ष पर संवेदनशील पुष्टियों में दूसरा चरण (PIN/बायोमेट्रिक) है क्या? → संवेदनशील एप्लिकेशन में न हो तो 🟡।
- **Task hijacking (StrandHogg):** `android:taskAffinity` खाली string (`""`) के रूप में सेट है क्या और `launchMode` उपयुक्त है क्या? डिफ़ॉल्ट taskAffinity के साथ दुर्भावनापूर्ण एप्लिकेशन स्वयं को तुम्हारे task के ऊपर बैठाकर नकली login स्क्रीन दिखा सकता है → 🟡।
- `android:usesCleartextTraffic="true"` → 🟠 (देखो A4)।
- `android:exported="true"` वाले `activity`/`service`/`receiver`/`provider`: क्या वास्तव में बाहर के लिए खुले होने चाहिए? अनावश्यक exported component = दूसरे एप्लिकेशन से trigger/डेटा रिसाव। `intent-filter` वाले स्वतः exported होते हैं; हर एक की समीक्षा करो।
- कस्टम `ContentProvider` `exported` और बिना अनुमति के हो → डेटा रिसाव।

## D2 — 🟠 Deep/App Link सत्यापन
- `intent-filter` वाली activity में `android:autoVerify="true"` और `.well-known/assetlinks.json` सही host पर है क्या? यदि नहीं तो दूसरा एप्लिकेशन लिंक चुरा सकता है (App Link hijack)।
- आने वाले link पैरामीटर बिना सत्यापन के संवेदनशील कार्रवाई trigger करते हैं क्या (देखो A10, B)।

## D3 — 🟡 Build / कोड छँटाई (Minification)
- `minifyEnabled true` + `shrinkResources true` release में चालू हैं क्या?
- ProGuard/R8 नियम संवेदनशील क्लासों की रक्षा करते हैं, अनावश्यक `-keep` से obfuscation को रद्द तो नहीं कर रहे?
- Signing: `signingConfigs` में keystore पासवर्ड सादा-पाठ में `build.gradle` में है (→ 🔴), या `key.properties`/env से आता है? `key.properties` और `*.keystore`/`*.jks` **`.gitignore` में हैं क्या**?

## D4 — 🟡 Play Integrity
- संवेदनशील एप्लिकेशनों में Play Integrity API / App Check (Play Integrity provider) एकीकृत है क्या? Emulator/root/tamper detection backend को संकेत देता है क्या?

---

# खंड E — iOS-विशिष्ट

`ios/` फ़ोल्डर वाले हर प्रोजेक्ट में। फ़ाइलें: `ios/Runner/Info.plist`, `ios/Runner/Runner.entitlements`, `ios/Podfile`।

## E1 — 🟠 App Transport Security (ATS)
- `NSAppTransportSecurity` → `NSAllowsArbitraryLoads = true` वैश्विक ATS bypass → 🟠। यदि अपवाद आवश्यक है तो domain-आधारित `NSExceptionDomains` होना चाहिए, वैश्विक नहीं।

## E2 — 🟠 URL Scheme व Universal Links
- `CFBundleURLTypes` (custom scheme) से खुलने वाले प्रवाह सत्यापित होते हैं क्या? Custom scheme hijack के लिए खुला है; संवेदनशील प्रवाह के लिए **Universal Links** को प्राथमिकता दी जानी चाहिए।
- `Runner.entitlements` → `com.apple.developer.associated-domains` (`applinks:`) सही है क्या? `apple-app-site-association` फ़ाइल host पर है क्या?

## E3 — 🟡 Keychain व गोपनीयता
- संवेदनशील डेटा Keychain में है (secure_storage इसका उपयोग करता है), या `NSUserDefaults`/plist में सादा? (देखो A3।)
- Keychain पहुँच स्तर: `kSecAttrAccessibleWhenUnlocked` (या अधिक कड़ा, जैसे `...WhenPasscodeSetThisDeviceOnly`) है, या `...Always`/`AfterFirstUnlock` जैसा ढीला? ढीला स्तर = jailbroken डिवाइस पर `objection > ios keychain dump` से पढ़ना आसान हो जाता है।
- **बायोमेट्रिक सत्यापन अकेली रक्षा नहीं है:** objection/Frida से iOS बायोमेट्रिक जाँच (LocalAuthentication) रनटाइम पर bypass की जा सकती है। बायोमेट्रिक "पुष्टि" को क्रिटिकल डेटा को *लॉक रखना* चाहिए (डेटा बायोमेट्रिक के बिना डिक्रिप्ट न हो सके इस तरह `SecAccessControl` flag से एन्क्रिप्ट होना चाहिए — जैसे `biometric_storage` पैकेज `SecAccessControlWithFlags` उपयोग करता है), केवल "if (authenticated) दिखाओ" का लॉजिक पर्याप्त नहीं है।
- `Info.plist` अनुमति विवरण string (`NS...UsageDescription`) पूर्ण हैं क्या (crash + स्टोर अस्वीकृति)।
- **Privacy Manifest** (`PrivacyInfo.xcprivacy`) और "required reason API" घोषणाएँ अद्यतन हैं क्या? (Apple ने 2024 से अनिवार्य किया; कमी हो तो स्टोर अस्वीकृति → 🟡।)

## E4 — 🟡 Pod निर्भरताएँ
- `Podfile.lock` commit किया गया है क्या? Pod पिन किए गए हैं क्या, कोई ज्ञात CVE वाला pod है क्या?

---

# खंड F — KVKK / कानूनी अनुपालन (स्थानीय अनिवार्यता)

ये तकनीकी "छेद" नहीं हैं पर **ऑडिट में प्रशासनिक जुर्माना** लाते हैं; स्टोर अस्वीकृति और मुकदमे का जोखिम रखते हैं। KVKK m.12: डेटा नियंत्रक "उपयुक्त सुरक्षा स्तर सुनिश्चित करने वाले हर प्रकार के तकनीकी और प्रशासनिक उपाय अपनाने के लिए बाध्य है।" ऊपर की तकनीकी मदें इस दायित्व का तकनीकी पहलू हैं। (GDPR आवश्यक प्रोजेक्टों में समानांतर दायित्व लागू होते हैं।)

**Claude Code प्रोजेक्ट के अंदर ये खोजे (अस्तित्व/अनुपस्थिति जाँच):**

## F1 — 🟠 सूचना पाठ (Aydınlatma metni) + स्पष्ट सहमति प्रवाह
- सूचना पाठ स्क्रीन/फ़ाइल है क्या? KVKK m.10: डेटा नियंत्रक की पहचान, कौन-सा डेटा किस उद्देश्य से संसाधित होता है, किसे स्थानांतरित किया जाता है, संग्रह विधि/कानूनी आधार, संबंधित व्यक्ति के अधिकार।
- यदि व्यक्तिगत डेटा प्रसंस्करण की शर्त नहीं है या स्थानांतरण है तो **स्पष्ट सहमति** स्क्रीन है क्या?
- व्यावसायिक संदेश (FCM push, ई-मेल) के लिए **अलग अनुमति** ली जाती है क्या? (बिना अनुमति व्यावसायिक संदेश अलग उल्लंघन है।)
- Analytics/Crashlytics/विज्ञापन SDK के लिए सहमति/opt-out है क्या?

## F2 — 🟠 डेटा न्यूनीकरण (स्थान आदि)
KVKK का "उद्देश्य से संबद्ध, सीमित, संयमित" सिद्धांत:
- Map/लिस्टिंग सिस्टम में **पूर्ण GPS pin के बजाय शहर/अनुमानित स्थान** उपयोग हो रहा है क्या? (A7 सुरक्षा और KVKK दोनों।)
- फ़ोन नंबर/ई-मेल दृश्यता **opt-in और डिफ़ॉल्ट रूप से बंद** है क्या? (कोड में संबंधित दृश्यता फ़ील्ड का डिफ़ॉल्ट `false` होना सत्यापित करो।)
- वास्तव में अनावश्यक फ़ील्ड एकत्र हो रहा है क्या?

## F3 — 🟡 UGC व उपयोगकर्ता अधिकार (लिस्टिंग/टिप्पणी/संदेश सिस्टम)
- शिकायत + ब्लॉक + सामग्री रिपोर्ट करने का तंत्र है क्या? (स्टोर और KVKK दोनों।)
- उपयोग शर्तें + गोपनीयता नीति स्वीकृति प्रवाह है क्या?
- **खाता हटाना** (डेटा नष्ट करने का अनुरोध) एप्लिकेशन के अंदर है क्या? KVKK m.7 + Apple Guideline 5.1.1(v) + Google Play नीति इसे **अनिवार्य** बनाती है → न हो तो 🟠 स्टोर जोखिम।
- डेटा पहुँच/सुधार/पोर्टेबिलिटी अनुरोध पूरा किया जा सकता है क्या?

## F4 — 🔵 डेटा प्रतिधारण + VERBİS
- व्यक्तिगत डेटा "आवश्यक अवधि तक" रखा जाता है, या अनिश्चित काल तक? स्वचालित हटाना/अनामकरण है क्या?
- VERBİS पंजीकरण दायित्व लागू है क्या? (अपवाद मानदंडों के अनुसार एक बार पुष्टि।)
- विदेश में डेटा स्थानांतरण (Firebase/Supabase सर्वर क्षेत्र) अनुपालन की दृष्टि से मूल्यांकित किया गया है क्या?

**नोट:** KVKK का "जो निषिद्ध नहीं है वह मुक्त नहीं, जो अनुमत है वह मुक्त है" दृष्टिकोण, तकनीकी पक्ष पर खंड C के **deny-by-default** नियम-तर्क से बिल्कुल मेल खाता है — सही सुरक्षा संरचना साथ-साथ अनुपालन भी सुनिश्चित करती है।

---

# आउटपुट फ़ॉर्मैट (Claude Code इसे तैयार करे → `GUVENLIK_BULGULARI.md`)

**1) सबसे ऊपर प्रोजेक्ट प्रोफ़ाइल तालिका (चरण 0)।**

**2) सारांश तालिका:**

| स्तर | संख्या |
|--------|------|
| 🔴 क्रिटिकल | ? |
| 🟠 उच्च | ? |
| 🟡 मध्यम | ? |
| 🔵 निम्न | ? |
| ⚪ मैनुअल सत्यापन | ? |
| ➖ लागू नहीं | ? |

**3) हर निष्कर्ष के लिए (क्रिटिकल सबसे ऊपर):**

```
## [🔴/🟠/🟡/🔵] Kısa başlık
- **Kategori:** (A1 / B2 / C1 / D3 / F3 ...)
- **Dosya:satır:** lib/product/db/auth_storage.dart:42
- **Sorun:** Refresh token SharedPreferences'a düz metin yazılıyor.
- **Kanıt:** `prefs.setString('refreshToken', token);`
- **Etki:** Root'lu cihazda token okunur → oturum ele geçirme. (İstemci tarafı mı / backend zorlaması mı olduğunu belirt.)
- **Önerilen düzeltme:** flutter_secure_storage'a taşı. (Kod YAZMA, önce onay al.)
```

**4) अलग "मैनुअल सत्यापन" सूची** (Cloud Console / dashboard / स्टोर सेटिंग जैसी कोड-बाहरी मदें): A2 कुंजी प्रतिबंध, App Check चालू है क्या, email enumeration protection, Firestore/RLS dashboard स्थिति, GraphQL introspection, GCP बजट चेतावनी, VERBİS आदि।

**5) अलग "लागू नहीं" सूची** (प्रोजेक्ट में वह तकनीक न हो तो: जैसे Supabase न हो तो C3, iOS फ़ोल्डर न हो तो खंड E, GraphQL न हो तो A8-GraphQL, कस्टम native कोड न हो तो A13)।

**6) स्व-ऑडिट (कवरेज) सूची:** A1–A13, B1–B4, C1–C8, D1–D4, E1–E4, F1–F4 के बीच की **हर मद** को एक-एक करके "मूल्यांकित / लागू नहीं / मैनुअल सत्यापन" के रूप में चिह्नित करो। यह सूची इस बात का प्रमाण है कि कोई भी मद चुपचाप नहीं छोड़ी गई — कोई मद अधूरी मत छोड़ो।

---

# प्राथमिकता क्रम (पहले इन्हें ठीक करो)

1. 🔴 `service_role` / वास्तविक secret / एम्बेडेड एन्क्रिप्शन कुंजी / legacy FCM server key क्लाइंट में है क्या? (A1, A11, C3, C6)
2. 🔴 Firestore/RLS `if true` या RLS बंद table / IDOR / असीमित `list` query? (C1, C3, B4)
3. 🔴 उपयोगकर्ता अपना `role`/`isPremium` फ़ील्ड लिख सकता है क्या — IAP रसीद सर्वर पर सत्यापित होती है क्या? (C1, C3, C7)
4. 🔴 Token/पासवर्ड/PII सादा-पाठ संग्रहण (Firestore offline cache सहित)? (A3)
5. 🔴 Cleartext HTTP / सर्टिफ़िकेट सत्यापन बंद / टूटा क्रिप्टो (MD5-SHA1-ECB)? (A4, A11)
6. 🔴/🟠 स्थान/स्कोर/समय क्लाइंट से आए हुए रूप में ही स्वीकार किया जा रहा है, velocity/jump check नहीं है क्या? (C8)
7. 🟠 MethodChannel / native ब्रिज में injection / path traversal (SQL, फ़ाइल पथ, Intent, exec)? (A13)
8. 🟠 केवल `auth != null`, स्वामित्व जाँच नहीं है? (C1, B4)
9. 🟠 Logout पर token revoke + `clearPersistence()` + संवेदनशील state सफ़ाई नहीं है / सामाजिक लॉगिन में मौन खाता निर्माण? (B2, B3, A3, A12)
10. 🟠 App Check + API कुंजी प्रतिबंध + email enumeration protection? (C1, A2, B1)
11. 🟠 FCM: payload में संवेदनशील डेटा, अनुमान-योग्य topic, असुरक्षित token फ़ील्ड? (C6)
12. 🟠 Deep link / url_launcher / WebView JS ब्रिज सत्यापन, exported components, tapjacking/task hijacking? (A10, D1, D2)
13. 🟠 GraphQL introspection चालू / query depth limit नहीं? (A8)
14. 🟠 रहस्य सादे env में हैं या Secret Manager/Vault में? (C5)
15. 🟠 KVKK सूचना/सहमति/खाता हटाने का प्रवाह? (F1–F3)
16. 🟡 स्क्रीन snapshot/क्लिपबोर्ड/कीबोर्ड/मेमोरी रिसाव, typosquatting, बजट चेतावनी की अनुपस्थिति? (A12, A9, C7)

---

### स्रोत (2026 तक अद्यतन)
- OWASP Mobile Top 10 (2024 Final Release) — M1 Improper Credential Usage … M10 Insufficient Binary Protections
- OWASP MASVS & MASTG (Mobile App Security Verification Standard / Testing Guide)
- OWASP Top 10:2025 (Web) — Broken Access Control #1 (प्राधिकरण लॉजिक के लिए); GraphQL introspection/depth के लिए OWASP GraphQL Cheat Sheet
- Firebase आधिकारिक: Security Checklist, Firestore/Storage Security Rules (get vs list), App Check, "Understand API keys", Email Enumeration Protection दस्तावेज़; Cloud Functions secrets (`defineSecret`) / Google Cloud Secret Manager
- FlutterFire: Auth error handling, `account-exists-with-different-credential`
- FCM: HTTP v1 API migration (legacy API बंद कर दिया गया), topic messaging सुरक्षा नोट
- Android: Backup & Restore (`dataExtractionRules`, API 31+), tapjacking (`filterTouchesWhenObscured`), StrandHogg/task affinity अनुसंधान
- Google Play Developer API / App Store Server API (सर्वर-साइड खरीद सत्यापन)
- Flutter platform channels (MethodChannel/EventChannel) native सुरक्षा नोट — इनपुट सत्यापन, path canonicalization, explicit Intent
- **हमलावर-पक्ष उपकरण और अनुसंधान (रक्षा अपेक्षा को कैलिब्रेट करने के लिए, बहुभाषी — संस्करणों को ऑडिट के समय अद्यतन संस्करण से पुष्टि करो):**
  - reFlutter (OWASP MASTG उपकरण) · Blutter — Flutter/Dart AOT रिवर्स इंजीनियरिंग
  - Frida · objection (SensePost) — pinning/keychain/बायोमेट्रिक runtime bypass
  - बहुभाषी "universal" root/SSL/emulator bypass script रिपॉज़िटरी (Frida CodeShare, GitHub — अंग्रेज़ी व चीनी समुदाय) · HackTricks anti-instrumentation
  - CyberCX & Appknox — Flutter/iOS jailbreak-detection (IOSSecuritySuite) bypass कार्य
  - iOS pentest पद्धति (Frida, objection keychain dump, MASVS L1/L2)
  - प्रकाशित उत्पादन Flutter रिवर्स इंजीनियरिंग मामला — एम्बेडेड RSA key निष्कर्षण + API पुनः कॉल
  - Talsec freeRASP — Flutter RASP (सुरक्षा पक्ष, प्रति-उपाय संदर्भ)
- Supabase आधिकारिक: Row Level Security & "Securing your API" दस्तावेज़ · Supabase Vault
- KVKK व्यक्तिगत डेटा सुरक्षा गाइड (तकनीकी और प्रशासनिक उपाय) + बोर्ड निर्णय सारांश
- Apple App Store Review Guidelines (4.8 / 5.1.1) · Google Play Data Safety & खाता हटाने की नीतियाँ
