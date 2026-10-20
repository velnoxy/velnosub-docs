# velnosub Faz 2 — Cihazda duman testi (arm64-v8a)

## Amaç
Faz 2 çıktısı `libvelnosub-core.so` ve `velnosub-cli` gerçek bir Android cihazında
(arm64) koşar. Plan §Faz 2'nin "ASS aç → kaydet → libass kare render"
hattının ilk iki ayağı (aç/kaydet) cihazda doğrulanır.

## Taşınacaklar
1. `libvelnosub-core.so` — `build-android-arm64/libvelnosub-core.so`
2. `velnosub-cli` — `build-android-arm64/tools/cross/velnosub-cli`
3. `libc++_shared.so` — yalnız readelf `-d` NEEDED görünüyorsa (NDK sysroot/lib)
4. `src/libresrc/default_config.json` — `VELNOSUB_CONFIG_JSON` ile CLI'ye verilir
5. Küçük bir örnek `.ass` dosyası

## Termux/cihaz koşumu
```sh
# ağaca kopyalanan dosyalar /data/local/tmp altında
chmod +x velnosub-cli
LD_LIBRARY_PATH=/data/local/tmp \
VELNOSUB_CONFIG_JSON=/data/local/tmp/default_config.json \
./velnosub-cli in.ass out.ass "--edit-first test metni"
```
- `out.ass` oluşursa aç/kaydet hattı çekirdekte onaylı demektir.
- `velnosub-cli`, `--edit-first` metnini ilk Dialogue satırına yazar, `Commit`
  ile commitler, kaydeder ve gidiş-dönüş yapan (reload) satır sayısını basar.

## libass kare render (P2.3)
- İkinci ikili `velnosub-ass-render` taşındıktan sonra font dosyasını da pushla:
  örneğin cihazda `DejaVuSans.ttf`.
- Koşum (`lib/` klasörüne paylaşılan kütüphaneler konursa):

```sh
cd /data/local/tmp/velnosub
LD_LIBRARY_PATH=$PWD/lib ./velnosub-ass-render in.ass out.ppm \
  --font $PWD/DejaVuSans.ttf --defaultfont $PWD/DejaVuSans.ttf --t 1 --w 640 --h 360
```

- Beklenen marker sırası: `[AR0] libass kütüphanesi kuruldu`,
  `[AR1] ASS yüklendi (olaylar: N)`, `[AR2] kare render edildi (glyph pikseli: N>0)`,
  `[AR3] PPM yazıldı`. Çıktı `out.ppm` 640x360 boyutundadır; glyph pikselleri siyah
  arka planın üstünde açık renkte koşar.
- Font seçimi notu (libass 0.17 davranışı): `ass_add_font`'ın ad parametresi
  eşleşme anahtarı DEĞİLDİR; embed provider fontu dosyanın gerçek family'sinden
  indeksler (DejaVuSans.ttf = "DejaVu Sans"). ASS stili başka bir ad isterse
  (bizim test dosyası "Arial" ister) `--defaultfont` verilir; libass log satırı
  `fontselect: (Arial, 400, 0) -> DejaVuSans.ttf` seçimi gösterir. `--family`
  yalnızca ass_add_font adı olarak geçer; işlevsel eşleşme sağlamaz.

## Hazır cihaz paketi (2026-10-20, device-pkg — P4.7 Δ yenilemeli)
- `device-pkg.tar.gz` (host'ta `/root/velsub/device-pkg.tar.gz`, **86.948.448 B**;
  37 dosya):
  `bin/velnosub-cli`, `bin/velnosub-ass-render`, **`bin/velnosub-automation`** (P5.0+P5.1/P5.2 manager taramalı),
  **`bin/velnosub-abi-smoke`** (P4.1–P4.5 III şeritleri + `[AB7]`+`[AB8]`+`[AB9]` libass önizleme, strip'li),
  **`bin/velnosub-video-probe`**
  (P3.1, strip'li 13.841.040 B), **`bin/velnosub-audio-probe`** (P3.3, strip'li
  13.842.600 B), **`bin/velnosub-audio-play`** (P3.4, strip'li 188.600 B;
  Oboe statik içsel), **`bin/velnosub-mkmpeg`** (P3.1 destek, strip'li
  13.841.416 B; sentetik mp4 üreticisi), **`bin/velnosub-ui`** (P4.0 egui
  eframe ikilisi, strip'li 5.647.032 B; stil+dialog panelleri + P4.7(+Δ)
  ilk-giriş düzen seçimi, mobil sekme çerçevesi, açık tema ve Ayarlar
  dahil; pencere APK bağlamı ister), `lib/libvelnosub-core.so` (P3.2
  sağlayıcılı 117.401.312 B),
  **`lib/libvelnosub-abi.so`** (P4.1–P4.5 III + P5.3 line_meta + P5.2 devam insert/delete + **P4.6 dilim 1 libass render** — libass STATİK bağlandı, **45 sembol**; Oboe-bağlı), `lib/libfreetype.so`, `lib/libpng16.so`,
  `lib/libfribidi.so`, `lib/libharfbuzz.so`, `lib/libuchardet.so`,
  `lib/libc++_shared.so`, `ass-format-tests.ass`, **`default_config.json`**,
  `DejaVuSans.ttf`, **`media/`** (`kisa.wav` 192.044 B, `kisa.wav.pcm`
  192.000 B, `kisa.mp4` 96.065 B, `dialog-ornak.txt` 288 B), **`share/lua/`**
  (`velnosub-aegisub.lua` yüzeyi + `velnosub-demo-macro.lua` + `lua-noop.lua`
  + **`autos/`** upstream birebir kopyalar: `cleantags-autoload.lua`,
  `strip-tags.lua` + **`inc/cleantags.lua`** — P5.1/P5.2/P5.3 makro koşumları,
  aşağıda reçete) — paket kendi test varlıklarıyla dolu.
  (Kök seviyede eski kopya `velnosub-cli/velnosub-ass-render`
  de duruyor; `bin/` kullanın.) Eski `device-pkg4.tar.gz` (23.479.573 B, 19-10)
  yedektir.
- Aktarım:

```sh
tar -xzf device-pkg.tar.gz
adb push device-pkg /data/local/tmp/velnosub
adb shell
# içinde: yukarıdaki CLI ve ass-render koşumları (LD_LIBRARY_PATH=$PWD/lib)
```

- **proot-içi paket-içi koşum reçetesi** (adb'siz, Termux içinden —
  2026-10-20'de kanıtlı): paket dizinine girip
  `LD_LIBRARY_PATH=$PWD/lib VELNOSUB_CONFIG_JSON=$PWD/default_config.json
  VELNOSUB_AUDIO=$PWD/media/kisa.wav.pcm ./bin/velnosub-abi-smoke
  $PWD/ass-format-tests.ass /tmp/out.ass` — `[AB0..AB5]`, `[AG0..AG5]`,
  `[AS0..AS5]`, `[AW0..AW5]`, `[AD0..AD6]` (dialog şeridi gömülü spec ile); video için
  `VELNOSUB_VIDEO=$PWD/media/kisa.mp4`
  → `[AV0..AV4]`. Paket kendi medyasıyla dolu, bağımsız koşar. `bin/velnosub-ui`
  eframe ikilisi pencereyi APK bağlamında açar (proot'ta beklenmedik durur);
  UI'da `VELNOSUB_DIALOG=$PWD/media/dialog-ornak.txt` ile dpanel makro formu
  açılır.
- **Makro koşum reçetesi (P5.1/P5.2)**: paket dizininden
  `VELNOSUB_AEGISHIM=$PWD/share/lua/velnosub-aegisub.lua LD_LIBRARY_PATH=$PWD/lib
  VELNOSUB_CONFIG_JSON=$PWD/default_config.json ./bin/velnosub-automation
  share/lua/velnosub-demo-macro.lua --ass $PWD/ass-format-tests.ass
  --macro "Form dene" --val metin=duman-formu --val sayi=7 --btn 1
  --save /tmp/f4-dev.ass` → `[AUS1] satır 41` `[AUL] form döndü: düğme Uygula,
  metin duman-formu, sayı 7, buyuk true` `[AUL] form hattı: DUMAN-FORMU @7`
  `[AUS2] makro koştu` `[AUS3] yazıldı` `[AUS4] kalıcılık: yeniden açılış
  satır 1: DUMAN-FORMU @7` `[AUS5] pump tamam` `[AUP0] pompacı koştu` —
  host + cihaz birebir; `--macro "Ana harf" --sel 2 --sel 3` seçim-upper şeridi.
- **Manager + upstream reçetesi (P5.3)**: paket dizininden
  `VELNOSUB_AEGISHIM=$PWD/share/lua/velnosub-aegisub.lua
  VELNOSUB_AEGIINC=$PWD/share/lua/inc LD_LIBRARY_PATH=$PWD/lib
  VELNOSUB_CONFIG_JSON=$PWD/default_config.json ./bin/velnosub-automation
  share/lua/lua-noop.lua --dir share/lua/autos --ass $PWD/ass-format-tests.ass
  --macro "Clean Tags" --sel 10 --save /tmp/f7-dev-clean.ass` →
  `[AUZ0] tarama: share/lua/autos (2 script)` `[AUZ1] yüklendi
  cleantags-autoload.lua / strip-tags.lua` `[AUS1..AUS5]` (40 satır Clean
  Tags + `[AUS6]` idx2/idx4 içerik birebir) — cihaz RC=0; `--macro
  "Strip tags" --sel 4 --sel 6` etiket sıyırma şeridi (bkz. faz5 plan
  P5.3 künyesi).
- **Ekle sil reçetesi (P5.2 devam)**: demo şeridine `--macro "Ekle sil
  dene" --sel 4 --save` → seçili satırın öne `"önce: "` önekli kopyası
  `subs:insert_line` ile eklenip aslı `subs:delete_line` ile siliniyor
  (net önekleme; adet değişmez); `[AUL] ekle sil: 4. satır öncesi/
  sonrası` + `[AUS2]` + `[AUS6]` kalıcılık `"önce: Now going to
  test..."` + `0:00:07.00,0:00:09.00` **birebir** — host + cihaz
  RC=0 (bkz. faz5 plan P5.2 devam künyesi).

## 2026-10-20 — proot-içi cihaz dumanı: tüm şeritler yeşil (KEŞİF)

Bulgu: paketteki bionic ikilileri adb'siz, Termux proot içinden DOĞRUDAN
koşmak mümkün — telefonun kendisi aarch64 olduğu için PT_INTERP üzerinden
sistem linker64 devreye girer; `LD_LIBRARY_PATH=$PWD/lib` ile lib/
çözümleniyor. Aşağıdaki şeritlerin tümü cihaz çipsetinde koşulmuş sayılır
(proot; `adb shell` içindeki koşumla birebir aynı kullanıcı-alanı):

| Duman | Şerit | Sonuç |
| --- | --- | --- |
| P4.1 `abi-smoke` | `[AB0] init` `[AB1] açıldı` `[AB2] satır: 41 stil: 2` `[AB3] düzenlendi` `[AB4] kaydedildi` `[AB5] tur: 41 satır` | ✔ `CAPI=0` |
| P5.0 `automation` | `[AU0]` `[AU1]` `[AUL]×2` `[AU2] script koştu` `[AU3] tamam` | ✔ |
| Faz 2 `cli` | `[0] config okuma` `[0b] 15030 bayt` `[1] config hazır` `[2] okuyucu hazır` `[3] 41 satır` + düzenleme + tur | ✔ |
| Faz 2 `ass-render` | `[AR0]` `[AR1] oluyanlar: 40` `[AR2] kare` `[AR3] PPM` | ✔ (aşağıda font notu) |
| P3.3 `audio-probe` | `[AP0] WAV` `[AP1] pcm_s16le 2ch 48000` `[AP2] parça…` | ✔ `.pcm` 192.000 B |
| P3.1 `mkmpeg`→`video-probe` | `[MK0..MK3]` (48 paket) → `[VP0] QuickTime/MOV 1.95833 sn` `[VP1] mpeg4 320x240 24fps` `[VP2]×3` `[VP3] decode 3` | ✔ |
| P3.4 `audio-play` | `[OP1] tampon 96000 sample` → `[OPE] ErrorInternal` (AAudio + OpenSLES yemleri) | ⚠ proot kısıtı (aşağıda) |
| P4.4 `abi-smoke` ses şeridi | `[AW0] ses açıldı (tutamaç: 1, rate 48000, ch 2)` `[AW1] s16 48000Hz 2k 48000 örnek` `[AW2] peak kovaları: 64 (boş olmayan 64, min<=max true)` `[AW3] oynatma durumu: 2 — arka-iplik openStream proot binder'da hızlı hata veriyor` `[AW4] oynatma kesildi` `[AW5] ses kol kapandı` | ✔ RC=0; **play artık bloke etmiyor** (senkron sürümün süresiz gömülmesi giderildi); gerçek çalma APK bağlamına kalır |
| P4.5 III `abi-smoke` dialog şeridi | `[AD0] dialog açıldı (alan 7, düğme 2)` `[AD1] ctl 0..6` `[AD2] 5 alan yazıldı` `[AD3] teyit: beş alan ikinci okumada birebir` `[AD4] düğme 1 seçildi (İptal, kayıt tutarlı)` `[AD5] tip reddi (-1,-1)` `[AD6] dialog kol kapandı` | ✔ RC=0 (host/cihaz/paket-içi birebir); 39 sembol defteri |
| P5.1/P5.2 `automation` makro şeridi | `[AUS0] yüzey yüklendi` `[AUS1] dosya (satır 41)` `[AUL] form döndü: düğme Uygula, metin duman-formu, sayı 7, buyuk true` `[AUL] form hattı: DUMAN-FORMU @7` `[AUS2] makro koştu` `[AUS3] yazıldı` `[AUS4] kalıcılık @7` `[AUS5] pump tamam` `[AUP0]` | ✔ RC=0 (host + cihaz birebir); `--macro "Ana harf"` seçim-upper da ✔ |
| P5.3 `[AB7]` tekrarlı düzenleme | `3 tur x 4 satır birebir` (aynı tutamaçta çoklu atama; flyweight çapraz-DSO regresyonu) | ✔ RC=0 (host + cihaz) |
| P5.3 manager + Clean Tags | `[AUZ0] tarama (2 script)` `[AUZ1] cleantags-autoload/strip-tags` `[AUL] progress: 40 lines cleaned` `[AUS3/4/6]` kayıt+kalıcılık | ✔ RC=0 (host + cihaz; upstream scriptler birebir) |
| P5.2 devam `[AB8]` ekleme/silme | `41→42 satır` (`ab8 ekleme domatesi` + `0:00:01.00 / 0:00:02.00` birebir) → `42→41 satır — şerit tamam` | ✔ RC=0 (host + cihaz) |
| P5.2 devam `Ekle sil dene` makrosu | `[AUL] ekle sil: 4. satır öncesi/sonrası` `[AUS2] makro koştu` `[AUS6] önce: Now going to test...` + `0:00:07.00,0:00:09.00` birebir | ✔ RC=0 (host + cihaz; `subs:insert_line`/`delete_line`/`n_lines`) |
| P4.6-1 `[AB9]` libass önizleme | `render açıldı: 240x180` `erken kare render: en çok glyph etki 9937` `uzak gele empty kare 0` `deposit boyu reddedildi` `şartı kapandı` | ✔ RC=0 (host + cihaz **birebir 9937**); ass-render lehine kanıt: `--t 8 --defaultfont DejaVuSans.ttf` → `glyph pikseli 11569`, PPM 691.215 B |
| P5.2 devam `[AB8]` ekleme/silme | `41→42 satır` (metin+zaman birebir) → `42→41 satır — şerit tamam` | ✔ RC=0 (host + cihaz) |
| P5.2 devam `Ekle sil dene` makrosu | `[AUL] ekle sil: 4. satır öncesi/sonrası` `[AUS2] makro koştu` `[AUS6] önce: Now going to test... +0:00:07.00/0:00:09.00 birebir` | ✔ RC=0 (host + cihaz; `subs:insert_line`/`delete_line`/`n_lines`) |

Notlar:
- **Proot boru tuzağı**: `… \| head/tail` ile koşan ikili ortada takılabiliyor
  (FFmpeg `version.sh \| tail -1` tuzağıyla aynı sınıf); çıktıyı `> dosya`
  yönlendirip sonra okumak güvenli.
- **Font eşleşmesi**: stil `Arial` isteyince glyph 0 (piksel yok); stil
  `DejaVu Sans` + `--font DejaVuSans.ttf --family "DejaVu Sans"` iken
  `[AR2] kare render edildi (glyph pikseli: 14345)` — libass fontselect
  gerçek family adıyla eşleşir; `ass_add_font` adı anahtar değildir.
- **P3.4 proot kısıtı**: `[OP1]` tampon yüklemesi çalışır, akış açımı
  AAudio ve OpenSLES yemlerinde de `ErrorInternal` (proot'ta dlopen/binder
  medya-sunucu yolu). Kod tarafına AAudio→OpenSLES yedek yolu eklendi
  (kısıtlı ortam dayanıklılığı); gerçek ses çıkışı APK bağlamında Faz 4
  koşucusunda ayrı doğrulanacak.
- **Medya kendin üret**: `bin/velnosub-mkmpeg kisa.mp4 --saniye 2` kendi
  statik FFmpeg'inin mpeg4 kodlayıcısıyla 96.065 B'lik test mp4'ü verir —
  P3.1 reçetesi kullanıcı medyasına muhtaç değil artık.

## P3.1 cihaz dumanı (velnosub-video-probe)
- Pakette örnek video YOK; medyayı kendi üret:

```sh
cd /data/local/tmp/velnosub
LD_LIBRARY_PATH=$PWD/lib ./bin/velnosub-mkmpeg kisa.mp4 --saniye 2
LD_LIBRARY_PATH=$PWD/lib ./bin/velnosub-video-probe kisa.mp4 --frames 5
```

  Paketlenmiş kümesi:
  demuxer `mov/matroska/avi/ogg/flv/wav`; decoder
  `h264/hevc/vp8/vp9/aac/mp3/flac/opus/vorbis/pcm_s16le/pcm_f32le/srt`.
- Probe statik FFmpeg bağlı; NEEDED yalnız `libm/libc++_shared/libdl/libc`:

```sh
cd /data/local/tmp/velnosub
LD_LIBRARY_PATH=$PWD/lib ./bin/velnosub-video-probe kisa.mp4 --frames 5
```
- Beklenen şerit: `[VP0] açıldı: <kapsayıcı adı> süre: X sn`, `[VP1] akış: <codec>
  WxH fps: F (time_base a/b)`, `[VP2]` PTS satırları; hata satırları `[VPE]`.

## P3.3 cihaz dumanı (velnosub-audio-probe)
- Örnek ses dosyası elle pushlanır (`adb push kisa.wav /data/local/tmp/velnosub/`;
  pcm_s16le wav doğrudan destekli; aac/mp3/flac/opus/vorbis decoderları da açık):
```sh
cd /data/local/tmp/velnosub
LD_LIBRARY_PATH=$PWD/lib ./bin/velnosub-audio-probe kisa.wav --seconds 5
```
- Beklenen şerit: `[AP0] açıldı: <kapsayıcı> süre: X sn`, `[AP1] akış: <codec>
  kanal: N samplerate: R (girdi <fmt>)`, `[AP2] parça K: S sample (toplam T)`
  satırları, `[AP3] toplam: T sample / B bayt / K parça → kisa.wav.pcm`,
  `[AP4] tamam`; hata satırları `[APE]`.
- PCM s16'ya swresample'la çevrilip parça-parça diske yazılır (bellek sınırlı;
  P3.4 Oboe oynatma zemini kanıtı). NEEDED yalnız
  `libm/libc++_shared/libdl/libc` (FFmpeg statik).

## P3.4 cihaz dumanı (velnosub-audio-play)
- P3.3 çıktısı (`kisa.wav.pcm`) meta verisiz ham PCM'dir: rate/kanal bilgisi
  P3.3 koşumunun `[AP1] samplerate/kanal` satırıyla eşleştirilip argümanla
  verilir:
```sh
cd /data/local/tmp/velnosub
LD_LIBRARY_PATH=$PWD/lib ./bin/velnosub-audio-play kisa.wav.pcm --rate 48000 --ch 2
```
- Beklenen şerit: `[OP1] tampon yüklü: N sample / S sn (rate R, ch C)`,
  `[OP0] akış açıldı: rate R, ch C, buffer B frames`, `[OP2] koşum: ... sample
  oynatıldı / süre S sn`, `[OP3] tamam`; hata satırları `[OPE]`.
- Oboe statik bağlı (NEEDED yalnız `liblog/libc++_shared/libm/libdl/libc`):
  hoparlörden gerçek çıkış duyulur (duyulmasa bile `[OP2]` satırı oynatma
  sayacını verir).

## P5.0 cihaz dumanı (velnosub-automation)
```sh
cd /data/local/tmp/velnosub
LD_LIBRARY_PATH=$PWD/lib ./bin/velnosub-automation
```
- Beklenen şerit: `[AU0] LuaJIT kuruldu: LuaJIT 2.1.1791988261`,
  `[AU1] aegisub tablo kayıtlı`, `[AUL] P5.0 zemin dumanı başladı`,
  `[AUL] arg: 0, toplam: 1499999500000`, `[AU2] script koştu`, `[AU3] tamam`.
- İsteğe bağlı script koşumu: `./bin/velnosub-automation buyuk_sc.lua` — script
  içinden `aegisub.log("...")` çağrıları `[AUL]` önekiyle stdout'a akar.
- Proot notu: çıktıyı `> dosya` yönlendir; `| head/tail` boru altında ikili
  ortada takılabiliyor (yukarıdaki boru tuzağı notası).

## P4.1 cihaz dumanı (velnosub-abi-smoke)
```sh
cd /data/local/tmp/velnosub
LD_LIBRARY_PATH=$PWD/lib \
VELNOSUB_CONFIG_JSON=$PWD/default_config.json \
./bin/velnosub-abi-smoke in.ass out_abi.ass "--edit-first cihaz abi dumanı"
```
- Beklenen şerit: `[AB0] init` → `[AB1] açıldı (tutamaç: 1)` → `[AB2] satır: 41 stil: 2`
  → `[AB3] düzenlendi` → `[AB4] kaydedildi` → `[AB5] tur: 41 satır`; çıkış kodu 0.
- `out_abi.ass` içinde ilk Dialogue satırı "cihaz abi dumanı" olmalı.
- Hata durumunda `[ABE] ...` satırı `velnosub_abi_last_error` metnini taşır.

## Notlar
- CLI statik libc++ ile bağlanabilir; `readelf -d` çıktısında NEEDED listesine
  göre `libc++_shared.so` taşınması gerekip gerekmediği netleşir.
- `libvelnosub-core.so` içinde ICU statik gömülü (veri paketi static).
