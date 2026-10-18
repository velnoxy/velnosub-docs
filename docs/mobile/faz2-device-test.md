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
- Koşum:

```sh
LD_LIBRARY_PATH=/data/local/tmp ./velnosub-ass-render in.ass out.ppm \
  --font /data/local/tmp/DejaVuSans.ttf --t 1 --w 640 --h 360
```

- Beklenen marker sırası: `[AR0] libass kütüphanesi kuruldu`,
  `[AR1] ASS yüklendi (olaylar: N)`, `[AR2] kare render edildi (glyph pikseli: N>0)`,
  `[AR3] PPM yazıldı`. Çıktı `out.ppm` 640x360 boyutundadır; glyph pikselleri siyah
  arka planın üstünde açık renkte koşar. `--family` varsayılan `Default`; ASS
  stilinde farklı bir font adı varsa blob'u o adla ass_add_font'a verin.

## Notlar
- CLI statik libc++ ile bağlanabilir; `readelf -d` çıktısında NEEDED listesine
  göre `libc++_shared.so` taşınması gerekip gerekmediği netleşir.
- `libvelnosub-core.so` içinde ICU statik gömülü (veri paketi static).
