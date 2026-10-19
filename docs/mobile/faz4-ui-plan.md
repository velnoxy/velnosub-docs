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

- P4.1 C ABI köprüsü, P4.2 grid+edit, P4.3 video paneli, P4.4 ses paneli, P4.5 stil/dialog,
P4.6 jestler/hotkey/düzenler: plan metni master plan Faz 4'te.
