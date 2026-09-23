# Zincir — Capacitor + SQLite sürümü

Bu sürüm veriyi tarayıcı depolamasında değil, telefonun kendi uygulama
alanındaki gerçek bir SQLite veritabanında tutar. Chrome/Brave'in site
verisini temizlemesi bu uygulamayı etkilemez.

## Yol A — Bilgisayarda Android Studio (önerilen, çok daha kolay)

1. Android Studio'yu kur (ücretsiz, Google'ın resmi aracı).
2. Bu klasörü bir terminalde aç:
   ```bash
   npm install
   npx cap add android
   npx cap sync android
   npx cap open android
   ```
3. Android Studio otomatik açılır. Üstteki yeşil "Run" (▶) tuşuna basarak
   telefonuna (USB ile bağlı, geliştirici modu + USB hata ayıklama açık)
   doğrudan kurabilir, ya da **Build → Generate Signed Bundle / APK**
   menüsünden imzalı bir `.apk` üretip elle kurabilirsin.

Android Studio; JDK, Gradle ve Android SDK'yı kendi içinde otomatik
yönetir, Termux'ta elle uğraştığın adımların hiçbiri gerekmez.

## Yol B — Telefonda Termux (proot Ubuntu), daha zahmetli

Önceden kurduğun JDK 17 ve Android SDK command-line tools yeterli, ek
olarak gerekiyor:

```bash
npm install -g @capacitor/cli   # zaten package.json'da devDependency var
cd zincir-capacitor
npm install
npx cap add android
npx cap sync android
```

Bu noktada projede bir `android/` klasörü oluşur, içinde Gradle wrapper
(`gradlew`) bulunur. Build almak için:

```bash
cd android
./gradlew assembleDebug
```

Çıktı `android/app/build/outputs/apk/debug/app-debug.apk` yolunda olur.
Bu, Bubblewrap'ten daha ağır bir adımdır: Gradle ilk çalıştığında birkaç
yüz MB bağımlılık indirir ve derleme telefon donanımına göre uzun (10-30+
dakika) sürebilir. `aapt2` burada da x86_64 nativedir; Gradle bunu Android
SDK'nın kendi build-tools'undan kullanır ve genelde sorunsuz çalışır çünkü
Gradle, ilgili platformun doğru `aapt2` ikili dosyasını `build-tools`
paketinden otomatik seçer (ARM64 Linux ana makine sorununu genelde
Gradle'ın kendi JVM tabanlı görevleri aşar, ama garanti değildir —
takılırsan hatayı paylaş, birlikte bakarız).

## İmzalama

İlk build'de Gradle imzasız bir debug APK üretir, bu telefonuna kurulabilir
ama Play Store'a yüklenemez ve güncellemede tutarlı bir imza anahtarı
garanti etmez. Kalıcı bir imza anahtarı oluşturmak için:

```bash
keytool -genkey -v -keystore zincir.keystore -alias zincir -keyalg RSA -keysize 2048 -validity 10000
```

Oluşan `zincir.keystore` dosyasını ve şifreni sakla, ileride her
güncellemede aynısını kullanman gerekiyor.

## Yapı

- `www/index.html` — tüm arayüz ve mantık, tek dosya.
- `capacitor.config.json` — uygulama kimliği (`com.frohdbek.zincir`) ve
  SQLite eklenti ayarları.
- Depolama katmanı native ortamda otomatik SQLite'a geçer; sıradan bir
  tarayıcıda açarsan (ör. `npx cap serve` veya `python3 -m http.server`)
  Capacitor bulunamadığı için localStorage'a düşer — böylece geliştirme
  sırasında tarayıcıdan da test edebilirsin, ama telefona kurulan gerçek
  APK her zaman SQLite kullanır.
