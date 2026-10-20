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

## Hazır cihaz paketi (2026-10-20 sabah, device-pkg)
- `device-pkg.tar.gz` (host'ta `/root/velsub/device-pkg.tar.gz`, 76.325.745 B):
  `bin/velnosub-cli`, `bin/velnosub-ass-render`, **`bin/velnosub-automation`** (P5.0),
  **`bin/velnosub-abi-smoke`** (P4.1, aarch64-PIE), **`bin/velnosub-video-probe`**
  (P3.1, strip'li 13.841.040 B), **`bin/velnosub-audio-probe`** (P3.3, strip'li
  13.842.600 B), **`bin/velnosub-audio-play`** (P3.4, strip'li 188.152 B;
  Oboe statik içsel), `lib/libvelnosub-core.so` (P3.2 sağlayıcılı
  117.401.312 B),
  **`lib/libvelnosub-abi.so`** (P4.1), `lib/libfreetype.so`, `lib/libpng16.so`,
  `lib/libfribidi.so`, `lib/libharfbuzz.so`, `lib/libuchardet.so`,
  `lib/libc++_shared.so`, `ass-format-tests.ass`, **`default_config.json`**,
  `DejaVuSans.ttf`. (Kök seviyede eski kopya `velnosub-cli/velnosub-ass-render`
  de duruyor; `bin/` kullanın.) Eski `device-pkg4.tar.gz` (23.479.573 B, 19-10)
  yedektir.
- Aktarım:

```sh
tar -xzf device-pkg.tar.gz
adb push device-pkg /data/local/tmp/velnosub
adb shell
# içinde: yukarıdaki CLI ve ass-render koşumları (LD_LIBRARY_PATH=$PWD/lib)
```

## P3.1 cihaz dumanı (velnosub-video-probe)
- Pakette örnek video yok; küçük bir kare örneği elle pushlayın
  (`adb push kisa.mp4 /data/local/tmp/velnosub/`). Paketlenmiş kümesi:
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
- P3.3 çıktısı (`kisa.wav.pcm`) mét verisiz ham PCM'dir: rate/kanal bilgisi
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
  içinden `aegisub.log("...")` çağrıları `[AUL] ` önekiyle stdout'a akar.

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
