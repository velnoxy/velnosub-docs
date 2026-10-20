# velnosub Faz 4 — Mobil UI (Rust/egui) künyeleri

Master plan §3 ve Faz 4 revizyonu gereği arayüz yığını Rust (egui/eframe) seçildi. Bu
dosya Faz 4 paketlerinin künye ve doğrulama kaydını tutar; kod crates'i kod deposunda
[`velnosub-ui/`](https://github.com/velnoxy/velnosub/tree/feature/velnosub-ui) altındadır.

## P4.0 — Rust araç zinciri + Android bootstrap ✔ (2026-10-19)

| Kayıt | Değer |
|---|---|
| Kod commit | [`e4e69a0da`](https://github.com/velnoxy/velnosub/commit/e4e69a0da) (feature dalı, `velnosub-ui/`) |
| Araç zinciri | rustup 1.29.1, cargo/rustc 1.99.1 (host aarch64-unknown-linux-gnu) |
| Hedef | `aarch64-linux-android` (rustup target add) |
| Bağımlılık | eframe 0.37.0; özellikler: `glow`, `default_fonts`, `android-native-activity`; Cargo.lock 223 paket sabit |
| Derleme | `cargo build --target aarch64-linux-android --release`, ilk geçişte 1 dk 19 sn (128 crate), marker `CB=0`; artımlı yeniden derleme 6.33 sn |
| Linker | NDK r29 clang sarmalayıcı (`aarch64-linux-android29-clang`), `.cargo/config.toml` içinden makine-yerel yol |
| Artefakt | `target/aarch64-linux-android/release/velnosub-ui` — 7.431.952 bayt, sha256 `9ff3c45e0…e95f95` |
| ELF denetimi | AArch64, DYN (PIE, FLAGS_1 NOW PIE BIND_NOW); NEEDED yalnız bionic: `libdl.so`, `libandroid.so`, `libm.so`, `libc.so` |
| Loglar | `/tmp/opencode/rustup-install.log`, `/tmp/opencode/ui-cargobuild.log`, `/tmp/opencode/ui-cargobuild-r2.log` |

Notlar:

- eframe 0.37 ile `App::update` kaldırıldı; ilgili yöntem `fn ui(&mut self, ui: &mut egui::Ui, frame: &mut eframe::Frame)`. Kod buna göre yazıldı.
- Açık kısıt (bloker değil): ikili yalnız derlendi, koşulmadı; çalıştırma Android cihazı gerektirir (API ≥ 29). Pencere/açılış davranışı P4.0 cihaz dumanıyla doğrulanacak.
- xbuild/APK paketleme bu dilimde gerekmedi; gerektiğinde ayrı künye gelir.

## Bekleyen paketler

- P4.2 grid+edit, P4.3 video paneli, P4.4 ses paneli, P4.5 stil/dialog,
P4.6 jestler/hotkey/düzenler: plan metni master plan Faz 4'te.

## P4.1 — C ABI şeridi ✔ (2026-10-19, host koşum)

| Kayıt | Değer |
|---|---|
| Kod commit'leri | [`564d9739e`](https://github.com/velnoxy/velnosub/commit/564d9739e) (`libvelnosub-abi` + abi-smoke crate), [`ebe19435d`](https://github.com/velnoxy/velnosub/commit/ebe19435d) (TU `fstream` katılımı), [`0074297d6`](https://github.com/velnoxy/velnosub/commit/0074297d6) (boş-config koruması) |
| Yüzey | `tools/cross/velnosub-abi.cpp`: `init/open/event_count/style_count/edit_first/save/reload_count/close/last_error` (9 dış sembol; `libvelnosub-abi.so`, `libvelnosub-core.so`'ya bağlanır) |
| Rust tarafı | `velnosub-ui/abi-smoke` (eframe'siz küçük crate; build.rs `-L`/`-lvelnosub-abi`; release derleme 9.3 sn) |
| Host koşumu | `[AB0] init` → `[AB1] açıldı (tutamaç: 1)` → `[AB2] satır: 41 stil: 2` → `[AB3] düzenlendi` → `[AB4] kaydedildi` → `[AB5] tur: 41 satır`; `CAPI=0`; `NINJA=0` (`[11/11] Linking target tools/cross/libvelnosub-abi.so`) |
| Derinleşen bulgular | (1) init'te boş config sessizce geçiliyordu; şimdi `config boş` hatası döner. (2) cargo ikilisi koşumunda `LD_LIBRARY_PATH` ister. (3) ilk derlemede eksik `<fstream>` geldiği gündeme dair düzeltme commiti [`ebe19435d`](https://github.com/velnoxy/velnosub/commit/ebe19435d) |
| Cihaz yolu | abi-smoke + `libvelnosub-abi.so` + `libvelnosub-core.so` pakette; aynı şerit `/data/local/tmp` reçetesiyle koşur |
| Cihaz koşumu (2026-10-20) | ✔.Termux proot'ta DOĞRUDAN koştu (adb'siz): tam şerit `[AB0..AB5]` + `[AB3] düzenlendi` (`--edit-first` ile), `CAPI=0`; reçete faz2-device-test.md'de |
