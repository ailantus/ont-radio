# ONT RADIO

Telegram'a gönderilen müzikleri GitHub çalma listesi üzerinden okuyan ve ses dosyalarını
doğrudan Telegram'dan çalan Android uygulaması. Uygulama web sitesine bağlanmaz; `sunucu/`
ve `web/` klasörleri eski kurulumun arşividir. Ayrı bir web oynatıcı için `web-site/`
klasörü kullanılır (aşağıya bakın).

## Nasıl çalışır?

1. `@radyoalse_bot` sohbetine mp3/audio/opus dosyası gönderilir.
2. `.github/workflows/telegram-sync.yml` mesajı okur, `radio/playlist.json` dosyasına
   gerçek Telegram `file_id` değerini ekler (beş dakikada bir).
3. Uygulama listeyi önbelleksiz CDN'den okur, `getFile` ile dosya yolunu çözer ve sesi
   Telegram'dan oynatır. Şarkılar Telegram'da kalır; GitHub'da yalnızca liste tutulur.
4. Çalma karışık ve sürekli döngüdür; ekolayzır yoktur.

Canlı liste: `https://cdn.jsdelivr.net/gh/ailantus/ont-radio@main/radio/playlist.json`

## Müzik gönderme

Telegram'da `@radyoalse_bot` sohbetini aç ve bir ses dosyası gönder. Workflow yeni parçayı
 otomatik ekler. YouTube bağlantısı gönderilirse bot videoyu indirip Telegram'a yükler ve
listeye ekler. Liste 100 parçayla sınırlıdır (en yeniler korunur).

Kanal veya grup kullanmak için botu kanala/gruba ekle ve gerekirse workflow'daki `chat_id`
filtresini kaldır. Botun grup mesajlarını görebilmesi için BotFather `/setprivacy` ayarı
**Disable** olmalıdır.

## Android uygulaması

Kotlin + Jetpack Compose + Media3/ExoPlayer. Sunucu adresi girişi yoktur; açılışta GitHub
listesine bağlanır. Ayarlar penceresinde `ONT RADIO · sürüm {versionName} ({versionCode})`
gösterilir.

Derleme ve kurulum:

```bash
cd TelegramRadyo
./gradlew :app:assembleDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## Otomatik güncelleme

Uygulama açılışta ve altı saatte bir
`https://api.github.com/repos/ailantus/ont-radio/releases/latest` adresini kontrol eder.
Etiket numarası (`v9` → `9`) telefondaki `versionCode`'dan **yeni**se “Güncelleme var”
penceresi çıkar, release'deki APK indirilir ve Android yükleme ekranı açılır. Sürüm aynıysa
veya daha yeniyse hiçbir şey yapılmaz. İndirme kesilirse kaldığı yerden devam eder.

Yeni sürüm yayınlama:

1. `app/build.gradle.kts` içinde `versionCode` değerini artır, `versionName` güncelle.
2. `./gradlew :app:assembleDebug` ile APK derle.
3. `https://github.com/ailantus/ont-radio/releases/new` adresinde etiketi `v<versionCode>`
   olarak oluştur, APK dosyasını varlık olarak ekle ve yayınla.

Şu anki sürüm: `v9` / `ONT RADIO 1.8` / `ont-radio-v9.apk`.

## Web oynatıcı

Adres: `https://ont-radio.rf.gd/`

`web-site/` klasörü InfinityFree üzerinde çalışan küçük bir PHP oynatıcıdır:

- `playlist.php` GitHub'daki `radio/playlist.json` dosyasını CDN'den okur (60 sn önbellek).
- `stream.php` Telegram `file_id` değerini PHP tarafında çözer; bot token tarayıcıya gönderilmez.
  InfinityFree sunucusu `api.telegram.org` DNS'ini çözemediği için `config.php` içindeki
  `TELEGRAM_API_IP` sabiti kullanılır.
- Arayüz karışık çalar, süre sayacı ve ileri/geri/ses denetimi vardır.

Yeniden yükleme (FTP bilgileri ortam değişkenlerinden okunur):

```bash
cd TelegramRadyo/web-site
FTP_USER=if0_xxxxx FTP_PASS=... python3 deploy.py
```

## Notlar

- En sorunsuz çalışma MP3 dosyalarıyladır; opus/ogg desteklenir ancak format geçişlerinde
  kısa kopmalar olabilir.
- Çalma sırası her açılışta karıştırılır; bir tur bitince aynı parçalar yeniden karışır.
- Eski `radyo.rf.gd` sitesi kapalıdır; yeni web adresi `ont-radio.rf.gd`'dir.
- Uygulamanın kapanma nedeni genellikle `ForegroundServiceDidNotStartInTimeException` olur;
  `PlaybackService.onCreate()` içinde `startForeground` çağrısı bunu önler.
