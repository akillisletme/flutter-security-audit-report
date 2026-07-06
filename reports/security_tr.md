# Flutter Uygulama Güvenlik Denetim Raporu
### (Herhangi bir Flutter + Firebase / Supabase projesine konulup Claude Code'a verilmek üzere hazırlanmış tarama talimatı)
**Sürüm:** 2026-07 (rev.4) · **Referans:** OWASP Mobile Top 10 (2024 Final, 2026'da hâlâ güncel) · OWASP MASVS/MASTG · Firebase & Supabase resmî güvenlik dokümanları · saldırgan tarafı araştırma (reFlutter/Blutter/Frida/objection, çok dilli kaynaklar)

---

## BU DOSYAYI NASIL KULLANACAKSIN (Claude Code'a verilecek komut)

Bu dosya bir **denetim talimatıdır**, sabit bir kontrol listesi değil. Farklı projelerde farklı teknolojiler bulunur (bir projede Supabase, diğerinde sadece Firebase, başka birinde ikisi birden olabilir). Claude Code **önce projeyi tanımalı**, hangi maddelerin geçerli olduğunu kendisi belirlemeli, geçerli olmayanları "UYGULANMAZ" diye işaretlemeli.

Uygulamanın kök dizininde çalışan Claude Code'a şunu söyle:

> "Proje kök dizininde `GUVENLIK_DENETIM_RAPORU.md` var. Önce projeyi tanı:
> 1. `pubspec.yaml`'ı oku, hangi paketlerin/servislerin kullanıldığını çıkar (Firebase mı, Supabase mı, ikisi mi; yerel depolama ne; auth sağlayıcıları neler; ağ istemcisi ne).
> 2. `android/` ve `ios/` klasör yapısını, manifest/plist dosyalarını tara.
> 3. Sonra bu rapordaki tüm kontrol maddelerini sırayla uygula. Her madde için ilgili dosyaları oku, kırmızı bayrak pattern'lerini ara. Projede karşılığı olmayan maddeleri 'UYGULANMAZ' olarak işaretle, uydurma.
> 4. Bulduğun her açığı en sondaki **Çıktı Formatı**'na göre `GUVENLIK_BULGULARI.md` adlı dosyaya yaz.
> **Kod DEĞİŞTİRME. Sadece tespit et ve raporla.** Düzeltmeleri ben madde madde onayladıkça yaparız."

Claude Code kod yazmadan önce tespit yapmalı. Önce tüm açıkları görelim, düzeltmeyi kullanıcı onayladıkça yaparız. Emin olamadığın (Cloud/dashboard ayarı gibi kod-dışı) her maddeyi **"MANUEL DOĞRULAMA"** listesine koy.

**Derinlemesine tarama için (LLM yüzeysel/tembel geçmesin):** Her dosyayı veya modülü incelerken doğrudan sonuca atlama. Önce bir `<analiz>` etiketi açıp adım adım düşün: *bu dosyada ne arıyorum, kodda ne görüyorum, bu durum rapordaki hangi kurala (A1 / C3 / ...) denk geliyor, bir kırmızı bayrak pattern'i eşleşti mi?* Analiz bitince bulguyu `GUVENLIK_BULGULARI.md`'ye yaz. Ayrıca **hiçbir maddeyi sessizce atlama:** raporun sonunda A1–F4 arasındaki her maddeyi "değerlendirildi / UYGULANMAZ / MANUEL DOĞRULAMA" olarak işaretleyen bir **öz-denetim (kapsama) listesi** üret — değerlendirmediğin madde kalmadığını böyle kanıtla. Uzun bir kod tabanında bölüm bölüm ilerle (her ana bölümü bitirince kısa bir ara özet ver), tek seferde yüzeysel geçme.

---

## SEVİYE TANIMLARI

- 🔴 **KRİTİK** — Sömürülürse tüm veri veya hesaplar ele geçer, para/itibar kaybı kesin. Hemen düzeltilmeli.
- 🟠 **YÜKSEK** — Ciddi veri sızıntısı veya yetkisiz erişim yolu. Bir sonraki sürümden önce düzeltilmeli.
- 🟡 **ORTA** — Saldırı yüzeyini büyütür, tek başına kritik değil ama zincirin halkası olur.
- 🔵 **DÜŞÜK / BİLGİ** — İyi uygulama ihlali, uzun vadede risk.

## ADIM 0 — PROJE ENVANTERİ (Claude Code önce bunu üretsin)

Rapora başlamadan bir "Proje Profili" tablosu çıkar:

| Alan | Tespit |
|------|--------|
| Backend | Firebase / Supabase / ikisi / custom API |
| Auth sağlayıcıları | e-posta+şifre / Google / Apple / telefon / anonim / custom |
| Yerel depolama | Hive / SharedPreferences / secure_storage / SQLite / Isar |
| Ağ istemcisi | Firebase SDK / dio / http / graphql |
| API paradigması | REST / GraphQL / gRPC / (yoksa yalnız SDK) |
| Özel native kod | MethodChannel / EventChannel / Kotlin-Swift eklenti var/yok |
| State/DI | bloc / riverpod / provider / get_it ... |
| Navigasyon | go_router / auto_route / Navigator ... |
| Hassas özellikler | ödeme / konum / kamera / sağlık / mesajlaşma / UGC (ilan-yorum) |
| Cloud Functions / Edge Functions | var/yok |
| Firebase Storage / Supabase Storage | var/yok |
| Uygulama risk sınıfı | finans-ödeme / sağlık / kimlik / UGC-sosyal / oyun-hobi (seviyeleri buna göre kalibre et) |

Bu profil, hangi bölümlerin geçerli olduğunu ve bulgu seviyelerinin nasıl kalibre edileceğini belirler. (Aynı "hardcoded secret" bir hobi uygulamasında ile bir fintech'te aynı gerçek riski taşımaz — risk sınıfını her bulgunun etkisini tartarken kullan.)

---

# SALDIRGAN PERSPEKTİFİ — 2026'DA İSTEMCİ TARAFI KORUMALARIN GERÇEĞİ

Bu bölüm bir kontrol maddesi değil, **tehdit modelidir.** Aşağıdaki maddeleri değerlendirirken Claude Code (ve geliştirici) şu gerçeği bilmeli: 2026 itibarıyla, kararlı bir saldırganın Flutter uygulamasına karşı elindeki araçlar olgun ve büyük ölçüde otomatik. Çeşitli dillerdeki (İngilizce, Rusça, Çince, İspanyolca, Almanca) araştırma topluluklarının ürettiği hazır araç zinciri şunları yapar:

**Statik / kod çıkarma:**
- **reFlutter** (OWASP MASTG'de resmî araç): `libapp.so`'yu patch'ler, Dart sınıf/fonksiyon/isim bilgisini döker, bazı Flutter cert-pinning implementasyonlarını **root gerektirmeden** bypass eder. (Not: Bazı Flutter sürümlerinde gömülü proxy IP kaldırıldı, artık cihazda proxy ayarı gerekiyor — yani engelleme değil, sadece bir adım daha. Sürüme özgü davranışı denetim anında teyit et.)
- **Blutter**: `libapp.so`'dan Dart AOT metadata'sını çözer; IDA/Ghidra için sembol isimlerini, object pool dökümünü ve hazır Frida şablonu üretir. Obfuscation olmayan uygulamada sınıf/metod isimleri okunur hale gelir.
- **Ghidra/IDA + Blutter script'i**: AES modu, SHA-256 kullanımı, hatta gömülü RSA private key gibi kriptografik detaylar disassembly'den çıkarılabilir.

**Dinamik / çalışma zamanı:**
- **Frida + objection**: tek komutla (`ios sslpinning disable`, `android sslpinning disable`) pinning bypass, **keychain dump**, dosya sistemi gezme, biyometrik doğrulama bypass. **Jailbreak/root gerektirmeden** `patchipa`/`patchapk` ile frida-gadget uygulamaya gömülebilir. (Araç sürümlerini denetim anında güncel sür ile teyit et; buradaki gerçek sürüme değil yeteneğe bağlı.)
- **Çok dilli hazır script depoları**: root tespiti + SSL pinning + emülatör + debug tespitini birlikte bypass eden "universal" script'ler herkese açık (Çince ve İngilizce topluluklarda özellikle yaygın). Root/jailbreak tespiti tek başına neredeyse hiçbir zaman yeterli değildir — atlanabilir.
- **IOSSecuritySuite / freerasp gibi jailbreak-detection kütüphaneleri bile** Frida hook'u veya yamalı binary ile atlatılabiliyor (CyberCX, Appknox gibi ekiplerin son dönem çalışmaları bunu gösteriyor).

**Gerçek dünya sonucu (yayınlanmış vaka):** Üretimdeki bir Flutter (Android) uygulaması aşama aşama tamamen tersine mühendislik edildi — **gömülü RSA private key çıkarıldı, custom RSA-SHA256 imzalama formatı yeniden kuruldu ve üretim API'si bağımsız bir Python client'tan başarıyla çağrıldı.** Yani uygulamanın "kendi client'ımdan gelmeyen isteği reddederim" varsayımı çöktü.

### Bundan çıkan üç savunma kuralı (raporun tamamının dayandığı temel)

1. **İstemcide duran her şey okunabilir kabul edilir.** Sır, anahtar, imzalama mantığı, iş kuralı — hepsi çıkarılabilir. Bu yüzden A1 (sırlar) ve A11 (kripto) maddeleri "obfuscate ettim, güvende" ile kapatılamaz.
2. **İstemci tarafı korumalar (obfuscation, pinning, root/JB tespiti) saldırıyı *engellemez, pahalılaştırır*.** Değersiz değiller — kitlesel/otomatik saldırıyı eler, saldırı maliyetini yükseltir — ama tek savunma hattı olarak asla yeterli değiller. Katmanlı olmalılar (defense-in-depth).
3. **Tek gerçek güvenlik sınırı backend'dir.** Yetkilendirme, veri sahipliği, ödeme/premium doğrulaması, hız sınırı **sunucuda** (Firestore Rules / RLS / Cloud Function + App Check/Play Integrity) uygulanmalı. Bir saldırgan senin uygulamanı değil, doğrudan senin API'ni çağırır — API o çağrıyı kendi başına doğrulayabilmeli.

> **Claude Code bunu şöyle kullansın:** Bir kontrolün "istemci tarafı koruma" mı yoksa "backend zorlaması" mı olduğunu her bulguda belirt. İstemci tarafı bir koruma tek başına kritik bir kararı (premium, yetki, ödeme, veri erişimi) koruyor ve backend'de karşılığı yoksa → bunu bulgunun "Etki" kısmında **"istemci tarafı bypass edilebilir, backend doğrulaması yok"** diye açıkça yaz.

---

# BÖLÜM A — İSTEMCİ (FLUTTER / DART) TARAFI

Temel gerçek şu: **derlenmiş APK/IPA herkese açık bir dosyadır.** `jadx`, `apktool`, `reFlutter`, `blutter` gibi araçlarla `libapp.so` decompile edilir; Dart AOT derlemesi bunu zorlaştırır ama engellemez. İstemcide duran hiçbir şeyi "gizli" kabul etme. OWASP Mobile Top 10 2024'te **M7 (Insufficient Binary Protections)** ve **M8 (Security Misconfiguration)** bu gerçeği vurgular.

> **OWASP 2024 kategori haritası (doğru numaralar):** M1 Improper Credential Usage · M2 Inadequate Supply Chain Security · M3 Insecure Authentication/Authorization · M4 Insufficient Input/Output Validation · M5 Insecure Communication · M6 Inadequate Privacy Controls · M7 Insufficient Binary Protections · M8 Security Misconfiguration · M9 Insecure Data Storage · M10 Insufficient Cryptography

## A1 — 🔴 Gömülü Sırlar / API Anahtarları (OWASP M1: Improper Credential Usage)

OWASP 2024 listesinde **1 numara** ve 2026'da hâlâ birinci. En sık ve en pahalı hata. Uber'in 2017 ihlali (S3 anahtarı repo'da), Strava API anahtarı sızıntısı, 2024'te onlarca popüler Android/iOS uygulamasında bulunan hardcoded credential'lar hep bu kategoriden.

**Claude Code şunları tarasın:**
- `lib/` altındaki tüm `.dart` dosyaları, `assets/`, `.env`, `pubspec.yaml`
- `android/app/google-services.json`, `ios/Runner/GoogleService-Info.plist`
- `android/local.properties`, `android/app/build.gradle(.kts)`, `ios/Runner/Info.plist`
- `firebase_options.dart` (FlutterFire CLI üretimi)
- Git geçmişi: `git log -p | grep -iE '(apikey|secret|token|password|service_role)'` ile sonradan silinmiş ama history'de kalmış anahtarlar

**Kırmızı bayrak pattern'leri (regex/grep):**
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

**Önemli ayrımlar (yanlış pozitif üretme):**
- **Firebase `apiKey`** (google-services.json / firebase_options.dart içindeki): teknik olarak "gizli değil", projeyi tanımlar. Firebase resmî dokümanı der ki: Firebase servislerine kısıtlı anahtarlar sır sayılmaz. **AMA kısıtlanmamışsa** (bkz. A2) veya user enumeration'a açıksa kötüye kullanılır. Bulguyu "kısıtlama kontrolü gerekli" olarak işaretle, "sızmış secret" deme.
- **Supabase `anon` key**: bilerek public, tek başına sorun değil — **AMA RLS kapalıysa felaket** (bkz. C bölümü). Bulguyu RLS durumuna bağla.
- **Supabase `service_role` key**: istemcide bulunursa **anında 🔴 KRİTİK** — tüm RLS'i bypass eder, tam veritabanı erişimi verir.
- **Google Maps / Places / Gemini API key**: `AndroidManifest.xml`/`Info.plist`/kod içinde olması bazen zorunlu ama **paket adı + SHA-1 (Android) veya bundle ID (iOS) ile kısıtlanmamışsa** başkası senin kotanı harcar → faturayı sen ödersin. Firebase anahtarından **ayrı** anahtar olmalı ve ilgili API'ye kısıtlanmalı.

**Doğru yaklaşım:** Gerçek sırlar backend'de (Cloud Functions / Supabase Edge Functions) durur; istemci geçici token ister. `flutter_dotenv`, `--dart-define`, `--dart-define-from-file` yalnızca **hassas olmayan** config içindir — derlenmiş binary'den okunabilir, güvenlik aracı değildir.

## A2 — 🟠 API Anahtarı Kısıtlaması Yok (kod-dışı, MANUEL DOĞRULAMA)

**Kontrol:** Google Cloud Console'da her anahtar için:
- Uygulama kısıtlaması var mı? (Android: paket adı + SHA-1 imza; iOS: bundle ID; Web: HTTP referrer)
- API kısıtlaması var mı? (Sadece kullanılan API'lere izin)
- Firebase anahtarı ile Maps/Gemini anahtarı **ayrı** mı?

Claude Code kodu okuyarak Cloud Console ayarını göremez — bu maddeyi **"MANUEL DOĞRULAMA"** olarak işaretlesin ve kullanıcıya hatırlatsın. Kısıtlanmamış anahtar = kota istismarı, spam hesap, veri enumerasyonu, sürpriz fatura.

## A3 — 🔴 Güvensiz Yerel Depolama (OWASP M9: Insecure Data Storage)

Kritik gerçek: **Hive (hive/hive_ce), SharedPreferences, SQLite varsayılan olarak şifresiz** düz metindir. Root'lu/jailbreak'li cihazda veya ADB backup ile `/data/data/<paket>/` altından okunur. Talsec araştırması: Firebase Auth JWT'si bile `shared_prefs/com.google.firebase.auth.*.xml` içinde **düz metin** durur ve root'lu cihazda çalınıp impersonation'da kullanılabilir.

**Claude Code şunları tarasın:**
- `SharedPreferences`, `Hive`/`hive_ce`, `sqflite`, `isar` yazan tüm yerler
- Bu depolara **ne yazıldığı**: token, şifre, e-posta, telefon, TC kimlik, konum, oturum, kart/IBAN → hassas
- Projede **`flutter_secure_storage` var mı?** Yoksa ve hassas veri yerel yazılıyorsa → bulgu.

**Kırmızı bayrak:**
```
prefs.setString(...token...) / setString(...password...) / setString(...jwt...)
Hive box / SharedPreferences alanları: authToken, refreshToken, accessToken,
    session, jwt, credit, card, iban, phone, email, tckn, otp, pin, apiKey
writeAsString(...) ile dosyaya token/PII yazımı (path_provider dizinlerine)
```

**Kurallar:**
- Hassas veri (token, oturum, kimlik, ödeme) → **`flutter_secure_storage`** (iOS Keychain / Android Keystore + EncryptedSharedPreferences).
- Hive'da kalması gereken hassas veri varsa → **AES ile şifrelenmiş box** (`HiveAesCipher`), şifreleme anahtarı **`flutter_secure_storage`**'da tutulur (kodda gömülü olmaz).
- Skor/zar/tema/dil/son sekme gibi **hassas olmayan** offline veri → düz Hive/SharedPreferences **tamam**.
- Not: Firebase SDK'nın token'ı düz metin tutması SDK davranışıdır; buna karşı savunma **cihaz bütünlüğü** (App Check / Play Integrity) + **kısa token ömrü + logout'ta token revoke**'tur (bkz. B ve C).
- **Firestore offline persistence cache'i de şifresizdir** (`Settings(persistenceEnabled: true)` — Flutter'da varsayılan açık). Uygulamanın okuduğu her Firestore verisi cihazda düz dosyada birikir. Çok hassas koleksiyonlar (mesaj, ödeme, kimlik) için persistence'ı kapatmayı veya cihaz kilidi/ek şifreleme katmanını değerlendir; en azından **logout'ta `clearPersistence()`** çağrılıyor mu kontrol et (çağrılmıyorsa önceki kullanıcının verisi cihazda kalır → paylaşılan cihazda sızıntı, 🟠).

Claude Code, tüm yerel depolara yazılan her alanı listelesin ve **hassas / hassas-değil** diye sınıflandırsın.

## A4 — 🔴/🟠 Güvensiz İletişim (OWASP M5) — HTTPS ve Sertifika Pinning

**Tarama:**
- `http://` ile başlayan (HTTPS olmayan) tüm endpoint'ler → 🔴
- `AndroidManifest.xml` içinde `android:usesCleartextTraffic="true"` → 🟠
- `res/xml/network_security_config.xml` içinde `cleartextTrafficPermitted="true"` veya gevşek `trust-anchors` → 🟠
- iOS `Info.plist`: `NSAppTransportSecurity` altında `NSAllowsArbitraryLoads = true` (ATS bypass) → 🟠
- Dio/http client'ta `badCertificateCallback` her zaman `true` dönüyorsa veya `HttpClient` `onBadCertificate = (_,__,___) => true` → 🔴 (sertifika doğrulama kapalı, MITM'e açık)

**Öneri:** TLS 1.2+ zorunlu, cleartext yasak. Hassas veri işleyen akışlarda **certificate pinning** (halka açık Wi-Fi'de MITM'e karşı). Firebase SDK trafiği zaten TLS'tir; ek `dio`/`http` çağrıları varsa pinning + interceptor ile token ekleme, hata loglarında hassas veri sızdırmama.

> **Gerçekçi beklenti:** Pinning kitlesel/otomatik MITM'i ve acemi saldırganı eler, ama reFlutter (`libapp.so` yaması) ve objection (`ios/android sslpinning disable`) hazır komutlarla birçok pinning implementasyonunu **root/jailbreak olmadan** atlayabiliyor. Yani pinning "trafiğim asla görülemez" garantisi değildir; **sunucu yine de her isteği kendi başına doğrulamalı** (App Check/Play Integrity + auth). Pinning'i eklemek iyidir, ona *güvenmek* hatadır.

## A5 — 🟡 Hassas Veri Loglama / Debug Modu Açık (OWASP M8)

**Tarama:**
- `print(...)`, `debugPrint(...)`, `developer.log(...)`, `logger.*` içinde token/şifre/PII/tam yanıt gövdesi geçen satırlar
- `kReleaseMode`/`kDebugMode` kontrolü olmadan production'da çalışan verbose loglama
- Crashlytics/Sentry'ye custom key/log olarak PII gönderme (`setCustomKey`, `recordError`, `log`) — e-posta/telefon/token gönderme
- `android/app/build.gradle`: release build'de `debuggable true` kalmışsa → 🟠
- Dio `LogInterceptor` release'de açık kalmışsa (tüm istek/yanıtı loglar) → 🟠

**Kural:** Release'de loglar kapalı/maskeli. Crashlytics'e ham PII gönderme.

## A6 — 🟡 Reverse Engineering / Obfuscation Yok (OWASP M7: Insufficient Binary Protections)

**Tarama:**
- Build script'lerinde / CI'da `flutter build ... --obfuscate --split-debug-info=<dir>` kullanılıyor mu?
- Android `android/app/build.gradle`: `minifyEnabled true` + `shrinkResources true` + R8/ProGuard kuralları var mı?

**Not:** Obfuscation %100 koruma değil, sadece saldırı maliyetini yükseltir. **Asıl kural: hassas iş mantığını ve yetkilendirmeyi backend'e taşı, istemciyi güvenilmez kabul et.** "101'e ulaştı mı", "hangi ses çalsın" gibi risksiz oyun/UI mantığı istemcide olabilir; ama "bu kullanıcı premium mi / bu işlem geçerli mi / bu skor gerçek mi" backend'de doğrulanmalı. Kritik kontrolleri sadece obfuscation'a değil, sunucu doğrulamasına dayandır.

> **Somut:** Obfuscation YOKSA, Blutter `libapp.so`'dan sınıf/metod isimlerini okunur hale getirir ve IDA/Ghidra'ya hazır sembol script'i üretir — kod neredeyse kaynak seviyesinde okunur. Obfuscation VARSA bu iş zorlaşır ama imkânsız olmaz (isimler gider, mantık kalır). Bu yüzden `--obfuscate` **olmalı** (özellikle kritik iş mantığı istemcideyse) ama tek başına yeterli sayılmamalı.

## A7 — 🟡 Aşırı İzin (Over-permissioning) + Root/Jailbreak & Tamper Tespiti

**Tarama:** `AndroidManifest.xml` ve `Info.plist` izinleri vs. uygulamanın gerçekten kullandığı özellikler.
- `ACCESS_FINE_LOCATION` gerçekten gerekli mi, yoksa `ACCESS_COARSE_LOCATION` (şehir/yaklaşık) yeterli mi? (Harita/ilan özelliğinde **tam GPS yerine yaklaşık konum** hem gizlilik hem KVKK açısından doğru.)
- `READ_SMS`, `READ_CONTACTS`, `CAMERA`, `RECORD_AUDIO`, `MANAGE_EXTERNAL_STORAGE`, arka plan konumu, `QUERY_ALL_PACKAGES` → kullanılmıyorsa kaldır (mağaza da reddedebilir).
- Her iOS izni için `Info.plist`'te açıklama string'i (`NSLocationWhenInUseUsageDescription`, `NSCameraUsageDescription` vb.) var mı? Yoksa uygulama çöker + mağaza reddi.
- **Root/jailbreak/emülatör tespiti** (ödeme/hassas uygulamalarda) var mı? Play Integrity / App Attest / `freerasp` gibi. Yoksa hassas akışlar için 🟡.

> **Uyarı:** Root/JB tespiti *tek başına* güvenlik değildir. Frida/objection ile ve çok dilli topluluklarda dolaşan "universal bypass" script'leriyle rutin olarak atlatılıyor; IOSSecuritySuite/freerasp gibi kütüphaneler bile yamalanabiliyor. Değeri: dürüst kullanıcıyı riskli ortamda uyarmak + otomatik/kitlesel fraud'u zorlaştırmak. Kritik karar (ödeme onayı, para transferi) **asla** "cihaz temiz göründü" varsayımına dayanmamalı; sunucu tarafı doğrulama + cihaz bütünlüğü sinyali (Play Integrity/App Attest, backend'de değerlendirilir) esas olmalı.

## A8 — 🟠 Girdi Doğrulama Eksikliği (OWASP M4)

**Kural:** İstemci doğrulaması bypass edilebilir — **her doğrulama sunucuda tekrarlanmalı.** İlan metni, kullanıcı adı, yorum, mesaj gibi UGC alanlarında:
- Uzunluk/format/tip sınırı sunucuda da (Firestore Rules `request.resource.data...` / Supabase policy / Edge Function) var mı?
- WebView kullanılıyorsa `javascriptMode` ve `onNavigationRequest` güvenli mi? Ham HTML/URL enjeksiyonu render ediliyor mu (XSS)?
- Deep link / `url_launcher` ile açılan URL'ler doğrulanıyor mu (bkz. A10)?
- SQL/NoSQL: kullanıcı girdisi doğrudan sorguya gömülüyor mu?
- **GraphQL (projede `graphql`/`graphql_flutter`/`ferry`/`artemis` varsa; yoksa UYGULANMAZ):**
  - Backend'de **introspection production'da kapalı** mı? Açıksa saldırgan tüm şemayı, tip ve alan isimlerini dışarıdan okuyup saldırı yüzeyinin haritasını çıkarır → 🟠.
  - Aşırı derin/iç içe (nested) sorgulara karşı **query depth / complexity limiti** var mı? Yoksa tek bir derin sorgu ile DoS + fatura patlatma → 🟠.
  - Alan bazlı yetkilendirme **sunucuda** mı? Sorgu istemciden gelir; hangi alanın kime döneceği backend'de sınırlanmalı (istemcide alanı gizlemek yetki değildir — bkz. B4).

## A9 — 🟡 Üçüncü Parti Bağımlılıklar (OWASP M2: Supply Chain)

**Tarama:**
- `flutter pub outdated` / `dart pub outdated` çıktısı — bilinen CVE'li/terk edilmiş paketler
- `pubspec.yaml`'da bakımı bırakılmış, düşük indirmeli, `git:`/`path:` ile çekilen paketler
- Pinlenmemiş versiyonlar (`any`, çok geniş `^`/`>=` aralıkları)
- `pubspec.lock` commit'lenmiş mi? (Tekrarlanabilir build için gerekli.)
- CI'da build çalıştıran ama pinlenmemiş action/paket var mı?
- **Typosquatting (sahte/benzer isimli paket):** `pubspec.yaml`'daki paket adlarında resmî/popüler paketlere kasıtlı benzeyen bir isim var mı (ör. `url_launcher` → `url_launcer`, `firebase_core` → `firebase_cor`, `flutter_secure_storage` → `flutter_secure_storge`)? **Elle göz taraması güvenilmez** — her paket adını pub.dev'deki gerçek pakete karşı doğrula: doğrulanmış yayıncı (verified publisher), indirme/beğeni sayısı, son yayın tarihi ve repo adresi tutarlı mı? Yeni eklenmiş + düşük indirmeli + doğrulanmamış yayıncılı + resmî bir isme çok benzeyen paket → 🟠 incele.

**Öneri:** Aylık bağımlılık taraması, MobSF ile otomatik statik analiz, `flutter analyze`/`dart analyze` CI'da zorunlu, `pubspec.lock` versiyon kontrolünde.

## A10 — 🟠 Deep Link / URL Launcher / Share Güvenliği

Bu madde `url_launcher`, `app_links`/`uni_links`, `share_plus`, WebView kullanan projeler için.

**Tarama:**
- `launchUrl(...)` çağrılarında URL **kaynağı**: kullanıcı içeriğinden (Firestore/Supabase'den gelen ilan linki, profil linki) geliyorsa ve şema doğrulanmıyorsa → saldırgan `javascript:`, `file:`, `intent:`, kimlik avı `https://` linki enjekte edebilir → 🟠. Sadece `https`/`mailto`/`tel` allow-list'i olmalı.
- **Deep link / App Link (Android) / Universal Link (iOS)** ile açılan ekranlar: gelen parametreyle doğrudan yetkili işlem yapılıyor mu? (Örn. `myapp://reset?token=...` doğrulama olmadan işleniyorsa hesap devralma yolu.)
- `AndroidManifest.xml` `intent-filter` `android:autoVerify="true"` ve `assetlinks.json` doğru mu? (App Link hijack'e karşı.)
- `share_plus` ile paylaşılan içerikte **PII/token sızıntısı** var mı? ("Oyun sonu kartı" gibi görseller genelde risksiz; ama paylaşılan metinde e-posta/telefon/derin link + token olmasın.)
- WebView: `Uri` doğrulama, `onNavigationRequest` allow-list, `allowFileAccess`/`allowUniversalAccessFromFileURLs` kapalı.
- **WebView ↔ Dart JS köprüsü:** `addJavaScriptChannel(...)` / `JavascriptChannel` ile web içeriğine native/Dart tarafına mesaj yollama kanalı açılıyorsa, WebView içinde çalışan (potansiyel olarak saldırgan kontrolündeki) JS bu kanalı çağırabilir → köprü fonksiyonu **doğrulanmamış girdiyle hassas işlem/veri erişimi tetikliyorsa** 🟠. Köprüye yalnızca güvenilen origin'lerden erişim + gelen mesajı sıkı doğrulama şart.

## A11 — 🟠 Zayıf / Yanlış Kriptografi (OWASP M10: Insufficient Cryptography)

Projede herhangi bir şifreleme/hash/imzalama varsa (paket: `crypto`, `encrypt`, `pointycastle`, `cryptography` vb.):

**Kırmızı bayraklar:**
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

**Kurallar:**
- Şifreleme anahtarı asla kodda/asset'te durmaz → Keystore/Keychain (`flutter_secure_storage`) veya sunucudan türetilir.
- Hash gerekiyorsa SHA-256+; parola için bcrypt/scrypt/Argon2 (tek başına SHA bile yetmez).
- `HiveAesCipher` anahtarı `Hive.generateSecureKey()` ile üretilip secure storage'da saklanmalı (A3 ile bağlantılı).
- Not: Projede hiç manuel kripto yoksa bu madde "UYGULANMAZ" — Firebase/Supabase SDK'ları kendi TLS/token kriptosunu doğru yönetir; **kendi kripto kodu yazmamak zaten en iyi pratiktir**, bunu bulgu olarak değil olumlu not olarak yaz.

## A12 — 🟡 Ekran / Pano / Klavye / Bellek Üzerinden Veri Sızıntısı (OWASP M6: Inadequate Privacy Controls)

Hassas ekranı olan (ödeme, kimlik, özel mesaj, bakiye) uygulamalarda:

**Tarama:**
- **Ekran görüntüsü/kayıt engeli:** Android'de hassas ekranlarda `FLAG_SECURE` (veya `screen_protector`/`no_screenshot` gibi paket) var mı? Yoksa ekran kaydı yapan zararlı/yardımcı erişim servisi her şeyi görür → hassas uygulamada 🟡/🟠.
- **Uygulama değiştirici (app switcher) snapshot'ı:** iOS arka plana geçerken ekranın fotoğrafını çeker ve app switcher'da gösterir; Android "Recents" aynısını yapar. Hassas ekran arka plana geçerken maskeleniyor mu (`AppLifecycleState.paused/inactive`'de overlay/blur)? → yoksa 🟡.
- **Pano (clipboard):** `Clipboard.setData(...)` ile token/şifre/IBAN/kişisel veri kopyalanıyor mu? Android'de diğer uygulamalar (13 öncesi) panoyu okuyabilir; kopyalama gerekliyse süreli temizleme düşünülmeli → 🟡.
- **Klavye önbelleği / autofill:** Hassas alanlarda (`TextField`) `obscureText: true` + `enableSuggestions: false` + `autocorrect: false` set edilmiş mi? Üçüncü parti klavyeler yazılanı öğrenip önbellekler; şifre alanında `autofillHints: [AutofillHints.password]` doğru mu? → eksikse 🟡.
- **Bildirim önizlemesi:** Kilit ekranında hassas içerik gösteren bildirim var mı (bkz. C6 FCM)?
- **Logout sonrası bellekte kalan hassas state (🟡):** `signOut()` çağrıldığında sadece UI yönlendirmesi mi yapılıyor, yoksa hafızadaki hassas state (ör. `UserBloc`, `WalletState`, `ProfileNotifier`, token/PII tutan `riverpod`/`provider`/`bloc` nesneleri) **açıkça sıfırlanıyor / `close()` / `dispose()` ediliyor** mu? Sıfırlanmıyorsa, **paylaşılan cihazda** (uygulama süreç olarak canlıyken hesap değiştirme) bir sonraki kullanıcı önceki kullanıcının verisini ekranda/state'te görebilir → 🟡. *Gerçekçi not:* Dart'ta belleği garantili biçimde silmek mümkün değildir (GC + string immutability); bu yüzden bu bir "memory dump'a karşı koruma" değil, **oturum hijyeni** maddesidir — asıl tehdit RAM dump eden root'lu saldırgan değil, aynı cihazı kullanan bir sonraki kişidir. Kritik olan: logout'ta state ağacını temizle + `clearPersistence()` (A3) + secure storage'daki oturum verisini sil.

## A13 — 🟠 MethodChannel / Native Köprü Zafiyetleri (Kotlin/Swift) — Native Injection & Path Traversal

Bu madde projede özel `MethodChannel` / `EventChannel` / `BasicMessageChannel` veya elle yazılmış native (Kotlin/Java, Swift/Obj-C) kod varsa uygulanır. Yalnızca hazır Firebase/Supabase SDK kullanan, hiç özel platform kanalı olmayan projede → **UYGULANMAZ.**

**Tarama:**
- Dart tarafında `MethodChannel('...')`, `invokeMethod(...)`, `EventChannel(...)` çağrılarını bul; karşılıklarını Android (`android/app/src/main/kotlin|java/.../*.kt|*.java`, `configureFlutterEngine` içinde `MethodChannel(...).setMethodCallHandler`) ve iOS (`ios/Runner/*.swift`, `FlutterMethodChannel(...)`) tarafında eşleştir.
- İstemciden gelen argümanlar (`call.arguments`, `call.argument("x")`) native tarafta **doğrulanmadan** şuraya akıyor mu:
  - SQL/SQLite sorgusu (string birleştirme → **SQL injection**),
  - dosya yolu (`File(path)`, `FileInputStream`, `NSFileManager` → **path traversal**; `../../` ile uygulama sandbox'ı dışına çıkma),
  - `Intent` (Android) parametresi / `startActivity` / `PendingIntent` (→ intent injection, yetkisiz bileşen tetikleme),
  - `Runtime.exec` / `ProcessBuilder` (→ komut enjeksiyonu),
  - `WebView.loadUrl` / `evaluateJavascript` (→ XSS / JS köprü kötüye kullanımı).

**Kırmızı bayrak:**
```
// Android (Kotlin)
db.rawQuery("SELECT * FROM t WHERE id = " + call.argument<String>("id"), null)  → SQL injection
File(context.filesDir, call.argument<String>("name")!!)   // ../ / canonical kontrolü yok → path traversal
Runtime.getRuntime().exec(call.argument<String>("cmd"))   → komut enjeksiyonu
// iOS (Swift)
let p = docsDir.appendingPathComponent(args["name"] as! String)  // prefix/canonical kontrolü yok
```

**Kural (raporun temel felsefesiyle):** Native kod da **istemcinin parçasıdır** — orada yapılan doğrulama da tersine mühendislik/hook ile bypass edilebilir. Yani iki katman birden gerekir:
1. Native tarafta injection/path-traversal'a karşı **doğru yazım** yine de şarttır (parametreli sorgu, path canonicalization + sandbox kökü `startsWith` kontrolü, explicit `Intent`, giriş allow-list'i) — bu istemci içi güvenlik hijyeni.
2. **Ama yetki/sahiplik/ödeme kararı burada verilmez;** o karar backend'de doğrulanır. MethodChannel'ı "güvenli iç API" sanma — saldırgan native fonksiyonu doğrudan da (hook ile) çağırabilir. Kanaldan gelen "şu kullanıcı yetkili" gibi bir iddiaya güvenilmez.

---

# BÖLÜM B — KİMLİK DOĞRULAMA AKIŞLARI (Login / Kayıt / Oturum)

Bu bölüm OWASP M3 (Insecure Authentication/Authorization). Firebase Auth / Supabase Auth / custom fark etmeksizin **login akışının kendisi** en çok istismar edilen yüzeydir. Aşağıdakiler projede hangi sağlayıcı varsa ona göre uygulanır.

## B1 — 🟠 E-posta Enumerasyonu Koruması (Firebase Auth)

Firebase Auth/Google Identity Platform'da `createAuthUri` / `fetchSignInMethodsForEmail` bir e-postanın kayıtlı olup olmadığını sızdırır → saldırgan sızmış e-posta listesini "bu platformda kimin hesabı var" diye eler → hedefli oltalama + credential stuffing. Google **15 Eylül 2023**'ten sonra oluşturulan projelerde **email enumeration protection**'ı varsayılan açtı; `fetchSignInMethodsForEmail` deprecate edildi.

**Tarama:**
- Kodda `fetchSignInMethodsForEmail(...)` kullanımı var mı? Varsa → auth akışı bu deprecate/kapatılmış davranışa bağımlı, 🟠 (kırılır + enumeration riski). "Önce e-posta gir, login mi kayıt mı olduğuna karar ver" deseni bu API'ye dayanıyorsa yeniden tasarlanmalı.
- **MANUEL DOĞRULAMA:** Firebase Console → Authentication → Settings → "Email enumeration protection" açık mı? (Eski projelerde manuel açılmalı.)

## B2 — 🟠 Sosyal Giriş Sessiz Hesap Oluşturma & Sağlayıcı Karışması (Google/Apple)

Kritik davranış: `signInWithCredential` / `signInWithPopup` ile Google/Apple/Facebook girişinde, eşleşen hesap yoksa Firebase **sessizce yeni hesap oluşturur**. `isNewUser` (AdditionalUserInfo) yalnızca ilk oluşturmada döner. Bumble/benzeri örneklerinde: saldırgan mağdurun **henüz kayıtsız** e-postasıyla önce hesap açıp sonra devralabilir.

**Tarama (projede `google_sign_in` / `sign_in_with_apple` / Firebase social varsa):**
- Google/Apple giriş akışında `account-exists-with-different-credential` hatası **ele alınıyor mu**? Alınmıyorsa aynı e-postanın iki farklı sağlayıcı hesabı yetkilendirme kafası karıştırır → 🟠. (Firebase "One account per email address" ayarı + credential linking akışı olmalı.)
- Apple ile giriş: Apple **relay/private e-posta** verebilir ve **ad/e-postayı yalnızca ilk sefer** döner. Kod sonraki girişlerde bu bilgiyi tekrar bekliyorsa bozulur; ilk seferde kaydedilmeli.
- Apple Sign-In **zorunluluğu:** Uygulama başka bir sosyal giriş (Google/Facebook) sunuyorsa App Store Guideline 4.8 / 5.1.1 gereği **Apple ile Giriş de sunulmalı** (yoksa mağaza reddi). Projede Google var Apple yoksa → 🟡 (mağaza riski).
- `google_sign_in` yeni sürümleri eski sürümlerden farklı API'ye sahip (`GoogleSignIn.instance`, `authenticate()`, `authorizationClient`). Eski `signIn()` deseninden migrasyon eksikse derleme/çalışma hatası + akış kırılması olur — kullanılan sürümü ve API uyumunu kontrol et.

## B3 — 🟠 Oturum / Token Yönetimi

**Tarama:**
- **Logout'ta token revoke:** Çıkışta sadece yerel state temizleniyor ama `signOut()` çağrılmıyor / refresh token iptal edilmiyorsa, cihazda kalan token'la oturum sürdürülebilir → 🟠. Firebase'de `FirebaseAuth.instance.signOut()`, hassas senaryoda ayrıca sunucu tarafı token revoke. (Logout'ta bellekteki hassas state'in de temizlendiğini A12 ile birlikte kontrol et.)
- **Refresh token kalıcılığı:** Refresh token, iptal edilene kadar sonsuz idToken üretir. Hassas uygulamada çıkış/şifre değişiminde revoke edilmeli.
- **E-posta doğrulaması:** Hassas işlemler `emailVerified == true` şartına bağlı mı? Değilse doğrulanmamış e-postayla yetki alınır.
- **MFA:** Yüksek değerli hesaplarda (ödeme, admin) çok faktörlü doğrulama var mı? (Firebase → Identity Platform yükseltmesi gerekir.)
- **Anonim auth:** Sadece geçici state için kullanılmalı, kalıcı hesap yerine geçmemeli; anonim kullanıcılar da security rules ile sınırlanmalı ("anyone can create anonymous account").
- **Brute force / rate limit:** Login denemelerine sınır var mı? (Firebase kısmen otomatik; custom auth'ta manuel gerekir.)
- **Şifre politikası:** Minimum uzunluk/karmaşıklık, sızmış şifre kontrolü (Identity Platform password policy) açık mı?

## B4 — 🟠 Yetkilendirme ≠ Kimlik Doğrulama (Broken Access Control)

OWASP Web Top 10 2021/2025'te **#1 Broken Access Control**. Mobilde de asıl açık burada.
- "Giriş yaptı" (authenticated) ile "bu işlemi yapabilir" (authorized) karıştırılıyor mu? Rol/sahiplik kontrolü **sunucuda** (rules/policy/function) mı yapılıyor, yoksa sadece istemcide UI gizlenerek mi? İstemci gizleme güvenlik değildir.
- Route guard (go_router `redirect` / auto_route guard) sadece UI'yi koruyor; **veri erişimi backend kuralıyla** korunmalı.
- IDOR: Başka kullanıcının `id`'siyle döküman/kayıt çekilebiliyor mu? (Örn. `/users/{başkasınınId}` istemciden istenebiliyorsa ve rule sahiplik kontrol etmiyorsa → 🔴.)

---

# BÖLÜM C — BACKEND (FIREBASE + SUPABASE)

Mobil ihlallerin büyük kısmı istemcide değil, **yanlış yapılandırılmış backend kurallarında** olur. 2024'te tek bir araştırma 916 Firebase sitesinde ~125 milyon kaydın (19 milyonu düz metin şifre) açıkta olduğunu buldu. Supabase tarafında ihlallerin büyük kısmı RLS'in kapalı bırakılmasından kaynaklanıyor.

> Not: Projede bu servislerden hangisi varsa o alt bölüm uygulanır; olmayan **"UYGULANMAZ"** işaretlenir.

## C1 — 🔴 Firebase Firestore / Realtime DB / Storage Güvenlik Kuralları

**Claude Code şu dosyaları okusun:** `firestore.rules`, `database.rules.json`, `storage.rules`, `firebase.json` (hangi rules dosyasının deploy edildiğini `firebase.json` gösterir).

**En tehlikeli pattern (test modu production'a kaçmış):**
```
allow read, write: if true;         → 🔴 herkes her şeyi okur/yazar
match /{document=**} { allow read, write: if true; }  → 🔴 tüm DB açık
".read": true, ".write": true        → 🔴 (Realtime DB)
```
Firestore "test mode" kuralları **30 gün sonra kapanır** ama süre uzatılıp production'a taşınmış olabilir; ayrıca `allow read, write: if request.time < timestamp.date(2025,...)` gibi tarih bazlı geçici kural production'da → 🔴.

**İkinci en yaygın hata — "giriş yapan herkes her şeye erişir":**
```
allow read, write: if request.auth != null;   → 🟠
```
`request.auth != null` sadece "giriş yapmış" der, **hangi veriye** erişebileceğini sınırlamaz. Başka bir kullanıcı da giriş yapmıştır.

**Doğru desen (sahiplik + veri doğrulama):**
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

**Claude Code her collection için sorsun:**
1. `allow ... if true` veya tarih-bazlı geçici kural var mı? → 🔴
2. Sadece `auth != null` mı, sahiplik/UID eşleşmesi var mı? → yoksa 🟠
3. `update`/`delete` sahibiyle sınırlı mı? (Başkasının ilanını/dökümanını değiştirebilme = veri bütünlüğü + hesap devralma yolu)
4. `create`/`update`'te **alan doğrulama** var mı (`request.resource.data...` tip/uzunluk, değiştirilemez alanlar `resource.data.x == request.resource.data.x`)? Kullanıcı `role: admin` veya `isPremium: true`'yu kendisi yazabiliyor mu → 🔴 yetki yükseltme.
5. **`get` ile `list` (query) kuralları ayrıdır.** Tek bir dökümanı ID ile okumaya izin veren bir kural, koleksiyonu **sorgulamaya (query/list)** otomatik izin vermez; query yapılan yerlerde kuralın query'nin döndürebileceği tüm dökümanlar için sağlanması gerekir. `list`/query erişimi sahiplik/filtre ile sınırlı mı, yoksa "auth != null herkes tüm koleksiyonu sorgular" mı? → sınırsız query 🟠/🔴.
6. **Storage kuralları ayrı** (`storage.rules`) — Firestore güvenli olsa da Storage açık olabilir. Dosya boyutu (`request.resource.size < 5*1024*1024`) ve tip (`request.resource.contentType.matches('image/.*')`) sınırı var mı? Yoksa kota istismarı/zararlı dosya.
7. Yazma **hız sınırı** / abuse koruması var mı? (Saldırgan çöp veri yazıp faturayı/limiti patlatabilir.)

**Ek katman — App Check (🟠, çok önemli):** Firestore/Storage/RealtimeDB/Functions/Auth için **App Check** açık mı? İsteğin gerçek, değiştirilmemiş uygulamandan geldiğini doğrular (Android: Play Integrity, iOS: App Attest); script'lerle atılan sahte istekleri eler. Kodda `FirebaseAppCheck.instance.activate(...)` var mı bak. Yoksa security rules doğru olsa bile `anon`/otomatik istekler serbesttir → 🟠.

## C2 — 🟡 Firebase Remote Config Güvenliği

`firebase_remote_config` (örn. zorunlu versiyon / feature flag) kullanılıyorsa:
- **Remote Config bir sır deposu DEĞİLDİR.** Değerler istemciye iner ve okunabilir; API anahtarı/secret/URL koymak → 🔴.
- **Force-update / feature gate istemcide bypass edilebilir.** "Şu versiyonun altındakini kilitle" mantığı UX içindir; **güvenlik kararı** (premium mü, erişebilir mi) buna dayandırılmamalı — sunucuda doğrulanmalı.
- Remote Config koşulları PII (tam e-posta/telefon) ile hedefleme yapıyorsa gizlilik notu.

## C3 — 🔴 Supabase Row Level Security (RLS)

Supabase'de **altın kural: `public` şemasındaki her tabloda RLS açık olmalı, istisnasız.** `anon` key public'tir; RLS kapalıysa o key ile herkes tüm tabloyu okur/yazar/siler.

**Claude Code şunları tarasın:**
- `supabase/migrations/*.sql`, `schema.sql`, `.sql` dosyaları
- `CREATE TABLE` var ama takibinde `ENABLE ROW LEVEL SECURITY` **yok** → 🔴
- SQL editör/migration ile oluşturulan tablolar RLS'i **otomatik açmaz** (dashboard'un aksine). Migration'da açıkça yoksa → 🔴

**Kontrol sorgusu (MANUEL, kullanıcı çalıştırsın):**
```sql
SELECT tablename FROM pg_tables
WHERE schemaname = 'public' AND NOT rowsecurity;
```
Dönen her tablo = korumasız = 🔴.

**RLS açık ama politika hatalı desenler:**
```sql
-- 🔴 "izin veriyormuş gibi görünen ama her şeyi açan" politika
CREATE POLICY "read" ON profiles FOR SELECT USING (true);

-- ✅ doğru: sadece kendi satırı
CREATE POLICY "read own" ON profiles FOR SELECT USING (auth.uid() = user_id);
```

**Diğer kritik kontroller:**
1. **`service_role` key istemcide mi?** (`--dart-define`, kod içinde, `NEXT_PUBLIC_*`) → 🔴, RLS'i tamamen bypass eder.
2. **`user_metadata` claim'ine dayanan politika** → 🔴. Bu alanı kullanıcı kendi değiştirebilir; yetki kontrolünde kullanılırsa yetki yükseltme. `app_metadata` kullanılmalı.
3. **UPDATE politikası `WITH CHECK` içermiyorsa** → kullanıcı satırı başka kullanıcıya/tenant'a taşıyabilir. Multi-tenant'ta `tenant_id`/`org_id` `WITH CHECK` ile korunmalı.
4. **RPC / Edge Function'lar `anon` ile çağrılabiliyor mu?** Admin/lisans/ödeme fonksiyonları public çağrılabiliyorsa → 🔴.
5. **Storage bucket'ları:** `public` işaretli bucket = herkes tüm dosyaları okur. Bucket policy'leri ve dosya yolu tasarımı (path'te tenant/uid) kontrol edilmeli.
6. **Edge Function'da `service_role` sızıntısı:** başlangıçta tüm `env` loglama, hata mesajında bağlantı/secret döndürme.

## C4 — 🟠 Firebase + Supabase (veya çift backend) Karışık Kullanım Riski

Bir uygulamada **hem Firebase Auth hem Supabase DB** (ya da tersi) varsa: Supabase RLS politikaları Firebase kimliğini nasıl doğruluyor? İki sistem arası kimlik köprüsü (JWT doğrulama, custom claims) yanlışsa, bir tarafta authenticated olan diğer tarafta yetkisiz erişebilir. **Kimlik tek kaynaktan (single source of truth)** yönetilmeli; JWT imza doğrulaması (audience/issuer) eksiksiz olmalı.

## C5 — 🟠 Cloud Functions / Edge Functions Güvenliği

Projede `functions/` veya `supabase/functions/` varsa:
- Fonksiyonlar kimlik/yetki doğruluyor mu, yoksa "çağrıldıysa çalışır" mı? Callable olmayan HTTP fonksiyonlarında auth kontrolü elle yapılmalı.
- Girdi doğrulama, rate limit, `maxInstances` (fatura patlatan DOS'a karşı) ayarlı mı?
- **Sırlar Vault/Secret Manager'da mı, yalın env'de mi?** Çevre değişkeni (env var) bile fonksiyon loglarına, hata izlerine veya `console.log(process.env)` benzeri bir satıra sızabilir. Gerçek sırlar **Google Cloud Secret Manager / Firebase secrets (`defineSecret`) / Supabase Vault** içinde tutulup fonksiyona **referansla** verilmeli; kaynak kodda veya düz `.env`'de değil. Başlangıçta tüm `env`'i loglayan kod → 🔴.
- CORS ayarı çok gevşek mi (`*`)?
- Hata yanıtlarında stack trace / bağlantı bilgisi sızıyor mu?

## C6 — 🟠 Push Bildirim (FCM) Güvenliği

Projede `firebase_messaging` varsa:
- **Payload'da hassas veri var mı?** Bildirim içeriği kilit ekranında görünür, Google/Apple sunucularından geçer ve cihazda loglanabilir. OTP/token/bakiye/özel mesaj içeriği **payload'a konmaz**; bildirim "yeni mesajın var" der, içerik uygulama açılınca güvenli kanaldan çekilir (data-only push + fetch deseni). → İhlal varsa 🟠.
- **Topic güvenliği:** FCM topic'lerine **herkes abone olabilir** — istemcideki `subscribeToTopic('...')` adı tahmin edilebilirse (örn. `user_12345`, `admin`, `premium`) saldırgan da abone olur ve o topic'e giden her bildirimi alır. **Kullanıcıya/gruba özel veri topic ile değil, token bazlı (sunucudan tek cihaza) gönderilmeli.** → Topic üzerinden kişiye özel içerik gönderiliyorsa 🟠.
- **FCM token'ın saklandığı yer:** Token genelde Firestore'da `users/{uid}/fcmToken` gibi tutulur. Bu alanı **başkası okuyabiliyor/yazabiliyorsa** (C1 kurallarına bak) saldırgan token'ı değiştirip mağdurun bildirimlerini kendi cihazına yönlendirebilir → 🟠.
- **Sunucu tarafı gönderim:** Legacy Server Key (kalıcı, sızarsa herkese push atılır) mı, yoksa **HTTP v1 API + servis hesabı (kısa ömürlü OAuth token)** mı kullanılıyor? Legacy key koda/istemciye gömülüyse → 🔴. (Google legacy FCM API'yi kapattı; hâlâ kullanılıyorsa zaten kırıktır.)
- Bildirime tıklanınca deep link işleniyorsa A10/D2 doğrulamaları geçerli.

## C7 — 🟠 Uygulama İçi Satın Alma / Premium Doğrulaması + Fatura & Kötüye Kullanım İzleme

**IAP / premium (projede `in_app_purchase`, RevenueCat, `purchases_flutter` veya `isPremium`/`isPro` benzeri alan varsa):**
- Premium durumu **nerede belirleniyor?** İstemci "satın aldım" deyip Firestore/Supabase'e `isPremium: true` yazabiliyorsa → 🔴 (C1/C3 madde 4 ile aynı yetki yükseltme). Doğrusu: **makbuz (receipt/purchase token) sunucuda doğrulanır** (Cloud Function → Google Play Developer API / App Store Server API veya RevenueCat webhook) ve premium alanını **yalnızca sunucu** yazar; security rules istemcinin bu alanı yazmasını engeller.
- Ürün fiyat/hak bilgisi istemcide sabitse manipüle edilebilir; hak tanıma her zaman sunucu doğrulamasına bağlı olmalı.

**Fatura ve kötüye kullanım izleme (MANUEL DOĞRULAMA):**
- GCP **bütçe uyarısı** (budget alert) tanımlı mı? Firestore/Functions/Storage için **kullanım alarmı** var mı? Saldırı (kota istismarı, çöp yazma, sonsuz döngü fonksiyon) çoğu zaman ilk olarak faturada görünür; alarm yoksa haftalarca fark edilmez.
- Cloud Functions'ta `maxInstances` sınırı (C5) + Firestore'da anormal okuma/yazma uyarısı düşünülmeli.
- Firebase resmî güvenlik checklist'i DOS şüphesinde destek ile iletişimi önerir — olay müdahale adımı olarak not et.

## C8 — 🟠/🔴 İstemciden Gelen "İddia Edilen" Veriye Güvenme (Konum / Skor / Zaman / Sayaç Manipülasyonu)

Konum tabanlı, oyunlaştırma, sadakat/puan, adım-sayar, "check-in", mesafe/ödül içeren her akışta geçerli. Temel hata: **istemcinin gönderdiği lat/lng, skor, zaman damgası, adım sayısı, mesafe gibi değerleri backend'in körü körüne kabul etmesi.** Bunların hepsi istemcide üretilir → **hepsi sahtelenebilir** (Fake GPS / mock location, paketi düzenleyip skoru şişirme, cihaz saatini değiştirme).

**Tarama / sorular:**
- İstemci backend'e ham konum/skor/zaman gönderip backend bunu doğrudan mı yazıyor (Firestore/Supabase'e `score`, `distance`, `location`)? → doğrulama yoksa 🟠; bu değerle premium/ödül/sıralama belirleniyorsa 🔴.
- **Velocity / jump check** sunucuda var mı? İki ardışık konum arasındaki mesafe/süre fiziksel olarak mümkün mü (ör. 2 dakikada 400 km = imkânsız)? Ani sıçrama reddediliyor mu?
- Fake GPS / location mocking **sadece istemcideki eklentiyle** (`isMockLocation` flag'i) mi engellenmeye çalışılıyor? Bu istemci sinyali **tek başına yeterli değildir** (bypass edilebilir) — sunucu tarafı tutarlılık kontrolü (hız, coğrafi sınır, sunucu zaman damgası) esas olmalı.
- Zaman damgası: kritik akışlarda **sunucu zamanı** (`FieldValue.serverTimestamp()` / DB `now()`) mı kullanılıyor, yoksa istemcinin gönderdiği zaman mı? İstemci zamanına güvenmek → sıralama/ödül manipülasyonu.
- Skor/ilerleme: nihai skor istemcide hesaplanıp gönderiliyorsa manipüle edilir; kritikse sunucu doğrulaması / yeniden hesaplama gerekir.

**Kural:** Bu, A8 (girdi doğrulama) ve B4 (yetkilendirme) ile aynı aileden: **istemcinin söylediği hiçbir "olgu" doğrulanmadan gerçek kabul edilmez.** Backend, iddia edilen veriyi kendi bağımsız sinyalleriyle (sunucu zamanı, önceki durum, fiziksel mantık) tutarlılık açısından denetlemeli.

---

# BÖLÜM D — ANDROID'E ÖZEL

`android/` klasörü olan her projede. Dosyalar: `android/app/src/main/AndroidManifest.xml`, `android/app/build.gradle(.kts)`, `android/app/proguard-rules.pro`, `res/xml/network_security_config.xml`.

## D1 — 🟠 Manifest Yapılandırması
- `android:debuggable="true"` release'de → 🔴 (genelde build type'tan gelir, elle set edilmişse bak).
- `android:allowBackup="true"` (varsayılan) → ADB backup ile uygulama verisi çekilebilir. Hassas veri varsa `false` veya `fullBackupContent` ile hariç tut → 🟡/🟠. **Android 12+ (API 31) için `android:dataExtractionRules` ayrıca tanımlanmalı** — `fullBackupContent` tek başına yeni cihaz-cihaza aktarım (device transfer) senaryosunu kapsamaz; token/oturum dosyaları her iki kural setinde de `exclude` edilmeli.
- **Tapjacking:** Hassas onay butonları (ödeme, izin, silme) olan ekranlarda dokunuşlar overlay ile kandırılabilir. Kritik native view'larda `filterTouchesWhenObscured` / Flutter tarafında hassas onaylarda ikinci adım (PIN/biometrik) var mı? → hassas uygulamada yoksa 🟡.
- **Task hijacking (StrandHogg):** `android:taskAffinity` boş string (`""`) olarak set edilmiş mi ve `launchMode` uygun mu? Varsayılan taskAffinity ile zararlı uygulama kendini senin task'inin üstüne oturtup sahte login ekranı gösterebilir → 🟡.
- `android:usesCleartextTraffic="true"` → 🟠 (bkz. A4).
- `android:exported="true"` olan `activity`/`service`/`receiver`/`provider`: gerçekten dışa açık olmalı mı? Gereksiz exported bileşen = başka uygulamadan tetiklenme/veri sızıntısı. `intent-filter` olanlar otomatik exported'dır; her birini gözden geçir.
- Özel `ContentProvider` `exported` ve izinsizse → veri sızıntısı.

## D2 — 🟠 Deep/App Link Doğrulama
- `intent-filter` içeren activity'lerde `android:autoVerify="true"` ve `.well-known/assetlinks.json` doğru host'ta mı? Yoksa başka uygulama linkini çalabilir (App Link hijack).
- Gelen link parametreleri doğrulanmadan hassas işlem tetikliyor mu (bkz. A10, B).

## D3 — 🟡 Build / Kod Küçültme
- `minifyEnabled true` + `shrinkResources true` release'de açık mı?
- ProGuard/R8 kuralları hassas sınıfları koruyor mu, gereksiz `-keep` ile obfuscation'ı iptal etmiyor mu?
- İmzalama: `signingConfigs` içinde keystore şifresi düz metin `build.gradle`'da mı (→ 🔴), yoksa `key.properties`/env'den mi geliyor? `key.properties` ve `*.keystore`/`*.jks` **`.gitignore`'da mı**?

## D4 — 🟡 Play Integrity
- Hassas uygulamalarda Play Integrity API / App Check (Play Integrity provider) entegre mi? Emülatör/root/tamper tespiti backend'e sinyal veriyor mu?

---

# BÖLÜM E — iOS'A ÖZEL

`ios/` klasörü olan her projede. Dosyalar: `ios/Runner/Info.plist`, `ios/Runner/Runner.entitlements`, `ios/Podfile`.

## E1 — 🟠 App Transport Security (ATS)
- `NSAppTransportSecurity` → `NSAllowsArbitraryLoads = true` global ATS bypass → 🟠. İstisna gerekiyorsa domain bazlı `NSExceptionDomains` olmalı, global değil.

## E2 — 🟠 URL Şemaları & Universal Links
- `CFBundleURLTypes` (custom scheme) ile açılan akışlar doğrulanıyor mu? Custom scheme hijack'e açık; hassas akış için **Universal Links** tercih edilmeli.
- `Runner.entitlements` → `com.apple.developer.associated-domains` (`applinks:`) doğru mu? `apple-app-site-association` dosyası host'ta mı?

## E3 — 🟡 Keychain & Gizlilik
- Hassas veri Keychain'de mi (secure_storage bunu kullanır), yoksa `NSUserDefaults`/plist'te düz mü? (bkz. A3.)
- Keychain erişilebilirlik seviyesi: `kSecAttrAccessibleWhenUnlocked` (veya daha sıkı, örn. `...WhenPasscodeSetThisDeviceOnly`) mı, yoksa `...Always`/`AfterFirstUnlock` gibi gevşek mi? Gevşek seviye = jailbreak'li cihazda `objection > ios keychain dump` ile okunması kolaylaşır.
- **Biyometrik doğrulama tek savunma değil:** objection/Frida ile iOS biyometrik kontrol (LocalAuthentication) çalışma zamanında bypass edilebilir. Biyometrik "onay" kritik veriyi *kilitli tutmalı* (veri biyometrik olmadan çözülemeyecek şekilde `SecAccessControl` flag'leriyle şifrelenmeli — örn. `biometric_storage` paketi `SecAccessControlWithFlags` kullanır), sadece "if (authenticated) göster" mantığı yetmez.
- `Info.plist` izin açıklama string'leri (`NS...UsageDescription`) eksiksiz mi (çökme + mağaza reddi).
- **Privacy Manifest** (`PrivacyInfo.xcprivacy`) ve "required reason API" beyanları güncel mi? (Apple 2024'ten beri zorunlu; eksikse mağaza reddi → 🟡.)

## E4 — 🟡 Pod Bağımlılıkları
- `Podfile.lock` commit'li mi? Pod'lar pinli mi, bilinen CVE'li pod var mı?

---

# BÖLÜM F — KVKK / YASAL UYUM (YERLİ ZORUNLULUK)

Bunlar teknik "açık" değil ama **denetimde idari para cezası** getirir; mağaza reddi ve dava riski taşır. KVKK m.12: veri sorumlusu "uygun güvenlik düzeyini temin edecek her türlü teknik ve idari tedbiri almak zorundadır." Yukarıdaki teknik maddeler bu yükümlülüğün teknik ayağıdır. (GDPR gereken projelerde paralel yükümlülükler geçerli.)

**Claude Code proje içinde şunları arasın (varlık/yokluk kontrolü):**

## F1 — 🟠 Aydınlatma Metni + Açık Rıza Akışı
- Aydınlatma metni ekranı/dosyası var mı? KVKK m.10: veri sorumlusunun kimliği, hangi verinin hangi amaçla işlendiği, kimlere aktarıldığı, toplama yöntemi/hukuki sebebi, ilgili kişinin hakları.
- Kişisel veri işleme şartı yoksa veya aktarım varsa **açık rıza** ekranı var mı?
- Ticari ileti (FCM push, e-posta) için **ayrı izin** alınıyor mu? (İzinsiz ticari ileti ayrı ihlal.)
- Analitik/Crashlytics/reklam SDK'ları için rıza/opt-out var mı?

## F2 — 🟠 Veri Minimizasyonu (Konum vb.)
KVKK "amaçla bağlantılı, sınırlı, ölçülü" ilkesi:
- Harita/ilan sisteminde **tam GPS pin yerine şehir/yaklaşık konum** kullanılıyor mu? (Hem A7 güvenlik hem KVKK.)
- Telefon numarası/e-posta görünürlüğü **opt-in ve varsayılan kapalı** mı? (Kodda ilgili görünürlük alanının varsayılanının `false` olduğunu doğrula.)
- Gerçekten gerekmeyen alan toplanıyor mu?

## F3 — 🟡 UGC & Kullanıcı Hakları (İlan/Yorum/Mesaj Sistemi)
- Şikayet + engelleme + içerik bildirme mekanizması var mı? (Hem mağaza hem KVKK.)
- Kullanım şartları + gizlilik politikası kabul akışı var mı?
- **Hesap silme** (veri yok etme talebi) uygulama içinde var mı? KVKK m.7 + Apple Guideline 5.1.1(v) + Google Play politikası **zorunlu** kılar → yoksa 🟠 mağaza riski.
- Veri erişim/düzeltme/taşınabilirlik talebi karşılanabiliyor mu?

## F4 — 🔵 Veri Saklama + VERBİS
- Kişisel veri "gerekli süre kadar" mı tutuluyor, süresiz mi? Otomatik silme/anonimleştirme var mı?
- VERBİS kayıt yükümlülüğü geçerli mi? (İstisna kriterlerine göre bir kez teyit.)
- Yurt dışına veri aktarımı (Firebase/Supabase sunucu bölgesi) uyum açısından değerlendirildi mi?

**Not:** KVKK'nın "yasak olmayan serbest değil, izinli olan serbest" yaklaşımı, teknik tarafta C bölümündeki **deny-by-default** kural mantığıyla birebir örtüşür — doğru güvenlik kurgusu aynı zamanda uyumu sağlar.

---

# ÇIKTI FORMATI (Claude Code bunu üretsin → `GUVENLIK_BULGULARI.md`)

**1) En başa Proje Profili tablosu (Adım 0).**

**2) Özet tablo:**

| Seviye | Adet |
|--------|------|
| 🔴 Kritik | ? |
| 🟠 Yüksek | ? |
| 🟡 Orta | ? |
| 🔵 Düşük | ? |
| ⚪ Manuel Doğrulama | ? |
| ➖ Uygulanmaz | ? |

**3) Her bulgu için (kritikler en üstte):**

```
## [🔴/🟠/🟡/🔵] Kısa başlık
- **Kategori:** (A1 / B2 / C1 / D3 / F3 ...)
- **Dosya:satır:** lib/product/db/auth_storage.dart:42
- **Sorun:** Refresh token SharedPreferences'a düz metin yazılıyor.
- **Kanıt:** `prefs.setString('refreshToken', token);`
- **Etki:** Root'lu cihazda token okunur → oturum ele geçirme. (İstemci tarafı mı / backend zorlaması mı olduğunu belirt.)
- **Önerilen düzeltme:** flutter_secure_storage'a taşı. (Kod YAZMA, önce onay al.)
```

**4) Ayrı "MANUEL DOĞRULAMA" listesi** (Cloud Console / dashboard / mağaza ayarı gibi kod-dışı maddeler): A2 anahtar kısıtlaması, App Check açık mı, email enumeration protection, Firestore/RLS dashboard durumu, GraphQL introspection, GCP bütçe uyarısı, VERBİS vb.

**5) Ayrı "UYGULANMAZ" listesi** (projede o teknoloji yoksa: örn. Supabase yoksa C3, iOS klasörü yoksa Bölüm E, GraphQL yoksa A8-GraphQL, özel native kod yoksa A13).

**6) Öz-denetim (kapsama) listesi:** A1–A13, B1–B4, C1–C8, D1–D4, E1–E4, F1–F4 arasındaki **her maddeyi** tek tek "değerlendirildi / UYGULANMAZ / MANUEL DOĞRULAMA" olarak işaretle. Bu liste, hiçbir maddenin sessizce atlanmadığının kanıtıdır — eksik madde bırakma.

---

# ÖNCELİK SIRASI (önce şunları düzelt)

1. 🔴 `service_role` / gerçek secret / gömülü şifreleme anahtarı / legacy FCM server key istemcide mi? (A1, A11, C3, C6)
2. 🔴 Firestore/RLS `if true` veya RLS kapalı tablo / IDOR / sınırsız `list` query? (C1, C3, B4)
3. 🔴 Kullanıcı kendi `role`/`isPremium` alanını yazabiliyor mu — IAP makbuzu sunucuda doğrulanıyor mu? (C1, C3, C7)
4. 🔴 Token/şifre/PII düz metin depolama (Firestore offline cache dahil)? (A3)
5. 🔴 Cleartext HTTP / sertifika doğrulama kapalı / kırık kripto (MD5-SHA1-ECB)? (A4, A11)
6. 🔴/🟠 Konum/skor/zaman istemciden geldiği gibi mi kabul ediliyor, velocity/jump check yok mu? (C8)
7. 🟠 MethodChannel / native köprüde injection / path traversal (SQL, dosya yolu, Intent, exec)? (A13)
8. 🟠 Sadece `auth != null`, sahiplik kontrolü yok? (C1, B4)
9. 🟠 Logout'ta token revoke + `clearPersistence()` + hassas state temizliği yok / sosyal giriş sessiz hesap oluşturma? (B2, B3, A3, A12)
10. 🟠 App Check + API anahtarı kısıtlaması + email enumeration protection? (C1, A2, B1)
11. 🟠 FCM: payload'da hassas veri, tahmin edilebilir topic, korumasız token alanı? (C6)
12. 🟠 Deep link / url_launcher / WebView JS köprü doğrulaması, exported bileşenler, tapjacking/task hijacking? (A10, D1, D2)
13. 🟠 GraphQL introspection açık / query depth limiti yok? (A8)
14. 🟠 Sırlar düz env'de mi, Secret Manager/Vault'ta mı? (C5)
15. 🟠 KVKK aydınlatma/rıza/hesap silme akışı? (F1–F3)
16. 🟡 Ekran snapshot/pano/klavye/bellek sızıntıları, typosquatting, bütçe uyarısı yokluğu? (A12, A9, C7)

---

### Kaynaklar (2026 itibarıyla güncel)
- OWASP Mobile Top 10 (2024 Final Release) — M1 Improper Credential Usage … M10 Insufficient Binary Protections
- OWASP MASVS & MASTG (Mobile App Security Verification Standard / Testing Guide)
- OWASP Top 10:2025 (Web) — Broken Access Control #1 (yetkilendirme mantığı için); GraphQL introspection/depth için OWASP GraphQL Cheat Sheet
- Firebase resmî: Security Checklist, Firestore/Storage Security Rules (get vs list), App Check, "Understand API keys", Email Enumeration Protection dokümanları; Cloud Functions secrets (`defineSecret`) / Google Cloud Secret Manager
- FlutterFire: Auth error handling, `account-exists-with-different-credential`
- FCM: HTTP v1 API migrasyonu (legacy API kapatıldı), topic mesajlaşma güvenlik notları
- Android: Backup & Restore (`dataExtractionRules`, API 31+), tapjacking (`filterTouchesWhenObscured`), StrandHogg/task affinity araştırması
- Google Play Developer API / App Store Server API (sunucu tarafı satın alma doğrulaması)
- Flutter platform kanalları (MethodChannel/EventChannel) native güvenlik notları — girdi doğrulama, path canonicalization, explicit Intent
- **Saldırgan tarafı araç ve araştırma (savunma beklentisini kalibre etmek için, çok dilli — sürümleri denetim anında güncel sür ile teyit et):**
  - reFlutter (OWASP MASTG aracı) · Blutter — Flutter/Dart AOT tersine mühendislik
  - Frida · objection (SensePost) — pinning/keychain/biyometrik runtime bypass
  - Çok dilli "universal" root/SSL/emülatör bypass script depoları (Frida CodeShare, GitHub — İngilizce & Çince topluluklar) · HackTricks anti-instrumentation
  - CyberCX & Appknox — Flutter/iOS jailbreak-detection (IOSSecuritySuite) bypass çalışmaları
  - iOS pentest metodolojisi (Frida, objection keychain dump, MASVS L1/L2)
  - Yayınlanmış üretim Flutter tersine mühendislik vakası — gömülü RSA key çıkarımı + API yeniden çağırma
  - Talsec freeRASP — Flutter RASP (koruma tarafı, karşı-önlem referansı)
- Supabase resmî: Row Level Security & "Securing your API" dokümanları · Supabase Vault
- KVKK Kişisel Veri Güvenliği Rehberi (Teknik ve İdari Tedbirler) + Kurul Karar Özetleri
- Apple App Store Review Guidelines (4.8 / 5.1.1) · Google Play Data Safety & hesap silme politikaları