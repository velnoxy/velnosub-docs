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
- 2026-10-20 eki: (1) ikili taze derleme 7.431.800 B (RC=0, artımlı 1 dk 22 sn). (2) termux proot'ta koşum girişimi: kendi NEEDED zinciri çözlür; ama `libharfbuzz_ng.so`'nun istediği `libicu.so` `/apex/com.android.i18n` altında — proot'ta apex bind yok → `CANNOT LINK` (APK/gerçek kabuk bağlamında normal). (3) Rust-bionic zincirinin cihaz kanıtı abi-smoke ile geldi: `[AB0..AB5]` RC=0 (P4.1 künyesi).

## P4.2 — Grid + edit ✔ (2026-10-20, host + cihaz koşumu)

| Kayıt | Değer |
|---|---|
| Kod commit | [`4c623cf2a`](https://github.com/velnoxy/velnosub/commit/4c623cf2a) (feature dalı) |
| ABI yüzeyi | 9 sembol → 12 sembol: `line_text(h,idx,buf,n)`/`line_times(...)`/`edit_line(h,idx,text)`; hepsi `Commit("velnosub abi", COMMIT_DIAG_TEXT)` hattında (P2.2 CLI kanalıyla aynı); okuma tampon-gelişimi: yetmezse tam uzunluk döner |
| Host şeridi | `[AG0] satır metinleri okundu: 41 (dolu 41)` `[AG1] 5. satır …/zamanları 0:00:07.50 / 0:00:09.00` `[AG2] düzenlendi + geri okundu` `[AG3] kayıt ikinci tutamaçta doğrulandı` `[AG4] kapandı` `[AG5] şerit tamam`; AB şeridinin üstüne ekli (birebir) |
| Cihaz şeridi | Rust-bionic ikili (cargo aarch64-linux-android) proot'ta DOĞRUDAN koştu; AG şeridi host ile birebir RC=0. Yol: `LD_LIBRARY_PATH=<arm64-cross tools/cross>:<pkg lib>` — taze ABI .so önce gereklidir |
| egui GridApp | `velnosub-ui/src/abi.rs`: `ScrollArea::show_rows` sanallaştırma (yalnız görünen satır çizilir), her hücre değişimi `edit_line`+Commit, Kaydet/Yeniden aç düğmeleri; `VELNOSUB_OPEN`/`VELNOSUB_SAVE` env ile dosyaya bağlanır; build.rs hedef-duyarlı bağ yolu (android→mobil kolu sarmalı) |
| egui 0.37 bulguları | `TopBottomPanel` tekilleşti → `Panel::bottom(id_salt)`; panel `show(&mut Ui, …)` alır; `CentralPanel::show_inside` deprecated (panel içi kullanım için yine de doğru) |
| Derleme kanıtı | host `-p velnosub-abi-smoke` 3,77 sn RC=0; android workspace 5,26 sn artımlı RC=0; `velnosub-ui` 7.431.800 B (AArch64 PIE) |
| Etki sınırları | (1) GridApp koşumu Activity ister → APK bağlamı ayrı dilim. (2) host tüm-workspace derlemesi winit'te takılıyor ("platform not supported" — host-GUI özellik seti ayrı künye konusu); host doğrulama `-p velnosub-abi-smoke` ile |

## Bekleyen paketler

- P4.3 video paneli ✔ omurgası aşağıda künyeli; ses paneli (Faz 3 P3.5
  dalga/spektrum teslimi bunun içinden bağlanır) sırada, P4.5 stil/dialog,
  P4.6 jestler/hotkey/düzenler: plan metni master plan Faz 4'te.

## P4.3 — Video paneli omurgası ✔ (2026-10-20, host + cihaz koşumu)

| Kayıt | Değer |
|---|---|
| Kod commit | [`04ede790e`](https://github.com/velnoxy/velnosub/commit/04ede790e) (feature dalı) |
| ABI yüzeyi | 12 → 17 sembol: `video_open` (VideoProviderFactory::GetProvider hattı — P3.2 sağlayıcının cihazdaki İLK koşumu), `video_info` ("WxH fps kare çözücü" dizgesi), `video_geom` (sayısal W/H/fps/kare çıkışları), `video_frame(h, kare, buf, n)` (BGRA, pitch=w·4, satır-satır kopya; tampon azsa gereken boyutu döner), `video_close` |
| Host şeridi | `[AV0] tutamaç 1` `[AV1] 320x240 fps:24.000 kare:47 FFmpeg (velnosub)` `[AV2] kare 0 (307.200 B) ve kare 1 farklı: BGRA akışı canlı` `[AV3] geri sarma kanıtlandı (kare 0 birebir)` `[AV4] kapandı` — host-check çekirdeği de FFmpeg sağlayıcılı derlendi (swscale uyarısı normal: hızlandırılmış yuv420p→bgra sıralaması yok) |
| Cihaz şeridi | Rust-bionic ikili proot'ta birebir RC=0; sağlayıcı-fabrikası mpeg4 çözer, swscale BGRA, PTS-geri-sarma byte-birebir — P3.2 sağlayıcısı artık cihaz kanıtlı |
| egui VideoSection | `Panel::top("vpanel")`: doku + `◀ geri/ileri ▶` + `kare i/N` etiketi; BGRA→`ColorImage` R/B takası (alfa doygun); `ctx.load_texture` adlı önbellek; `VELNOSUB_VIDEO` env ile açılır |
| Sıradaki | ses paneli (P3.5 dalga/spektrum ABI teslimi + oynatma Oboe), libass önizleme katmanı (P2.3 render yolunun sayfa-dışı hattı) |

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
