# Velsub Faz 2 — Cihazda duman testi (arm64-v8a)

## Amaç
Faz 2 çıktısı `libvelsub-core.so` ve `velsub-cli` gerçek bir Android cihazında
(arm64) koşar. Plan §Faz 2'nin "ASS aç → kaydet → libass kare render"
hattının ilk iki ayağı (aç/kaydet) cihazda doğrulanır.

## Taşınacaklar
1. `libvelsub-core.so` — `build-android-arm64/libvelsub-core.so`
2. `velsub-cli` — `build-android-arm64/tools/cross/velsub-cli`
3. `libc++_shared.so` — yalnız readelf `-d` NEEDED görünüyorsa (NDK sysroot/lib)
4. `src/libresrc/default_config.json` — `VELSUB_CONFIG_JSON` ile CLI'ye verilir
5. Küçük bir örnek `.ass` dosyası

## Termux/cihaz koşumu
```sh
# ağaca kopyalanan dosyalar /data/local/tmp altında
chmod +x velsub-cli
LD_LIBRARY_PATH=/data/local/tmp \
VELSUB_CONFIG_JSON=/data/local/tmp/default_config.json \
./velsub-cli in.ass out.ass "--edit-first test metni"
```
- `out.ass` oluşursa aç/kaydet hattı çekirdekte onaylı demektir.
- `velsub-cli`, `--edit-first` metnini ilk Dialogue satırına yazar, `Commit`
  ile commitler, kaydeder ve gidiş-dönüş yapan (reload) satır sayısını basar.

## libass kare render (ayrı künye — P2.3)
- `velsub-ass-render` ikinci TU: `ass_library_create`, `ass_add_font`
  (font bloblarını ASS ekindeki fontlardan ya da diskten kaydeder), sonra
  `ass_renderer_create` + `ass_render_frame`.
- Sistem font sağlayıcısı yok (libass `require-system-font-provider=false`),
  fontlar yalnız kayıt yoluyla girer.

## Notlar
- CLI statik libc++ ile bağlanabilir; `readelf -d` çıktısında NEEDED listesine
  göre `libc++_shared.so` taşınması gerekip gerekmediği netleşir.
- `libvelsub-core.so` içinde ICU statik gömülü (veri paketi static).
