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

- P4.3 video paneli ✔, P4.4 ses paneli ✔, P4.5 stil omurgası **+ tam
  editör** ✔, P4.6 düzenler/önizleme ✔, P4.7 denetim düzeltmeleri ✔
  aşağıda künyeli; **P4.8 asıl uygulama APK'sı ✔ (aşağıda künyeli —
  cdylib + NativeActivity, telefona kurulabilir ilk gerçek UI)**. Sırada:
  SAF dosya seçici (kullanıcı dosyası açma), suağırım ay'ları (ses/video
  gerçek kanıtları), testmedia-ile panel şeritlerinin cihaz ekran yaşamı.

## P4.8 — Asıl uygulama APK'sı (egui UI cdylib + NativeActivity) ✔ (2026-10-21)

| Kayıt | Değer |
|---|---|
| Kod commit | [`baac28bde`](https://github.com/velnoxy/velnosub/commit/baac28bde) (dilim 2: cdylib + android_main) → [`a5feda404`](https://github.com/velnoxy/velnosub/commit/a5feda404) (dilim 3: apk-ui pakacı), `feature` |
| Plan/bulgu | `docs/mobile/faz4-8-ui-apk-plan.md` (`1a9235b`): eski planın "panel kodu kayıp, yeniden kur" varsayımı **yanlıştı** — panel kodu `abi.rs`'de mevcuttu (GridApp/Sekme/video/ses/form gövdeleri, 1863 satır); plan "kurtarma" → "doğrulama"ya revize edildi |
| Yapı | `[lib] velnosub_ui` crate-type `["cdylib","rlib"]` (bin host dev koşumunu sürer); `main.rs` tek satır (`calistir_env`); ortak `kos()` (tema `ayar_yukle`→`stil_tamir` + GridApp) host ile AYNI kapı |
| Android girişi | `android_main`: `internal_data_path` + `asset_manager`→filesDir (karaskel-dene.ass + DejaVuSans.ttf kopyası) → `calistir(open,save)`; `options.event_loop_builder → with_android_app(app)` (winit `platform/android.rs:128`); `ANativeActivity_onCreate`'in android-activity-0.6.1 `native_activity/glue.rs:876`'dan geldiği kaynağa doğrulandı |
| Sembol kanıtı | `llvm-nm -D libvelnosub_ui.so` → `T ANativeActivity_onCreate` + `T android_main`; boyut 8.359.432 B; NEEDED: `libvelnosub-abi.so` + platform küpü (dl/android/log/m/c) |
| Test | host `cargo test` **2 passed** (`kontrast_tamiri_aa_uyumlu`, `ayar_dongusu`) RC=0 — dönüşüm öncesiyle birebir; ABI defteri **47 sabit**; aarch64 cdylib RC=0 (26.67 s) |
| APK | `velnosub-ui-0.1.0-alpha.apk` **13.583.682 B**, `com.velnoxy.velnosubui` v4 (0.1.0-ui-alpha) — NativeActivity, `hasCode="false"` (dex yok), `extractNativeLibs`; kapanış küpü: ui/abi/core/freetype/fribidi/harfbuzz/uchardet/c++_shared (core 117 MB→18.4 MB strip); aapt (Debian arm64) → zip → zipalign 4 → apksigner **verify OK** (alpha keystore kimlik ailesiyle aynı); `/mnt/sdcard/Download/`'a kopyalandı |
| Süreç dürüstüğü | Dilim 2 alt-ajanı koşum ortasında sessizleşti (süreç/flock/yazma yok 10+ dk; iki durum-sorgusu boş döndü) — **orkestratör mekanik kalanı kendisi aldı**, alt-ajanın yazdığı yapıtlar (lib.rs + Cargo.toml E0433 `["cdylib","rlib"]` düzeltmesi) desteklendi; ilk apk denemesi 570 sn timeout yedi (160 MB kopya+strip+zip+imza — telefon IO), arka planda yeniden koşum sürmedi |
| Alpha sınırları | Dürüst: dosya seçici (SAF/JNI) **kuyrukta** — açılış yalnız paketli örnek .ass; Kayıt filesDir `kayit.ass`; depolama izni yok; libass önizleme fontu asset kopyasından |

## P4.5 — Stiller omurgası ✔ (2026-10-20, host + cihaz + paket-içi)

| Kayıt | Değer |
|---|---|
| Kod commit | [`5c62ee6bb`](https://github.com/velnoxy/velnosub/commit/5c62ee6bb) (feature dalı); devam: [`c6fd3fcd2`](https://github.com/velnoxy/velnosub/commit/c6fd3fcd2) |
| ABI yüzeyi | 26 → **30 sembol**: `style_of(h, idx)` → 13-hanelik '\|' ayrıklı alan dizgesi (ad\|font\|boyut\|4 renk ASS biçimi\|biçim-baytları\|hiza\|kenarlar\|kodlama\|ölçek-haraba-açı\|kenarstili-kontur-gölge), `style_edit_size`, `style_edit_name` (ikisi de `UpdateData` + `AssFile::COMMIT_STYLES`); **devam `style_write`** (13-hücreli dizge geri döngüsü: `ass_renk_ayikla` (`&HAABBGGRR` elle bayt'lı), `bi_coz`, tüm sayısal hücreler çözülmeden hiçbir alan yazılmaz — yarım durum yok; UpdateData + COMMIT_STYLES) |
| Rust smoke | `[AS0] stiller: 2` → `[AS1] stil 0: Default\|Arial\|20.000000…` `[AS1] stil 1: Alt\|Times New Roman\|40.000000…` → `[AS2] boyutu 24 yazıldı; teyit: 24.000000` → `[AS3] adı yazıldı (VelnoSub)` → **`[AS4] tam-yazma: font DejaVu Sans + ana renk &H0000FFFF teyitli`** → **`[AS5] kalıcılık teyidi: ad+boyut+font+renk ikinci tutamaçta birebir`** → `[AS6] şerit tamam` — host RC=0, cihaz (proot) RC=0, **paket-içi** RC=0 |
| egui Stiller paneli | `Panel::top("spanel")`: `StyleRow` ayıklayıcı (13 alan doğrudan parse), canlı `DragValue` boyut; **devam: font kutusu** (odak kaybında yazma), **ana renk renk düğmesi** (`ass_to_color32`/`color32_to_ass`, alfa tersi), **hiza ComboBox `\an1..\an9`** — üçü de `stil_yaz` detenciona açık: dizgeyi okur, seçili hücreleri değiştirir, ABI yazar, `load_styles()` tazelemesini açar |
| Paket | `refresh-device-pkg.sh` artık Rust ikililerini de kopyalar (`abi-smoke` 342.168 B + `velnosub-ui` eframe ikilisi) — eski bayat ikili tuzağı (ilk cihaz koşumunda yakalandı) kapandı; tar **86.780.115 B** (`libvelnosub-abi.so` Oboe-bağlı 3.831.416 B, 30 sembol) |
| Sıradaki | Faz 5 dialog formları (aynı widget dilimi), P4.6 jestler/hotkey/düzenler, APK bağlamı |

## P4.5 III — Dialog form omurgası ✔ (2026-10-20, host + cihaz + paket-içi)

| Kayıt | Değer |
|---|---|
| Kod commit | [`03cbf985f`](https://github.com/velnoxy/velnosub/commit/03cbf985f) (feature dalı) |
| ABI yüzeyi | 30 → **39 sembol**: `dialog_open(spec)` (mini DSL: `ctl\|sınıf\|ad\|başlik\|değer[\|ek]` + `buton\|metin` satırları, `#` yorum; sınıf kapısı: label/edit/intedit/floatedit/checkbox/dropdown/textbox/color/coloralpha; tip doğrulaması — intedit tam sayı, floatedit sayı, checkbox 0/1, dropdown değer öğeler arasında, label hariç boş değer reddi; varsayılan düğme tek halinde "Uygula"), `dialog_count`, `dialog_btncount`, `dialog_ctl` (okuma: sinif\|ad\|başlik\|değer[\|sınırlar\|öğeler] — intedit/floatedit `min,max` sınır hücresi yansır), `dialog_button`, `dialog_set` (tipli yazım), `dialog_show` (düğme seçimi kaydı), `dialog_showid`, `dialog_close` |
| Rust smoke | `[AD0] dialog açıldı (alan 7, düğme 2)` → `[AD1] ctl 0..6` (label 4-hücreli değersiz satır dahil) → `[AD2] 5 alan yazıldı (sayı/sürü/anahtar)` → `[AD3] teyit: beş alan ikinci okumada birebir` → `[AD4] düğme 1 seçildi (İptal, kayıt tutarlı)` → `[AD5] tip reddi: harf sayıya, öğe dışı anahtar reddedildi (-1,-1)` → `[AD6] dialog kol kapandı — şerit tamam` — host RC=0, cihaz (proot) RC=0, **paket-içi** RC=0 |
| Zarif-teardown kök düzeltmesi | abi-smoke'un tanınmış `RC=134` quirk'ü (hata-erken-çıkışlar `velnosub_abi_close`'u atlar → statik yıkıcıda çöküş) 44 çıkış noktası `cik(h)` kapısından geçtirilerek kökten kapatıldı — her yol kolunu kapatır, temiz RC |
| egui dinamik form paneli | `Panel::top("dpanel")`: sınıf↔widget eşleme — label→etiket, edit→singleline TextEdit, textbox→multiline, intedit/floatedit→DragValue (sınırlar `alt..=ust`), checkbox→`ui.checkbox` (0/1), dropdown→ComboBox (`selectable_value`, boru tazeleme), color/coloralpha→`color_edit_button_srgba` (ASS dönüşümü); düğme satırı: gönderme akışı (tüm alan değerleri `dialog_set` ile ABI'ye yazılır, sonra `dialog_show` seçim kaydı) + not satırı; `VELNOSUB_DIALOG` env (spec dosyası yolu) |
| Örnek spec | `testmedia/dialog-ornak.txt` (paket `media/` içine de aynen; ORNEK_SPEC ile birebir); `refresh-device-pkg.sh` medya bloğu eklendi — media/ artık elle kopyalanmıyor |
| Sıradaki | gerçek `aegisub.dialog.display` pompası (Lua makro ↔ form; P5.2'yle bağlanır), P4.6 jestler/hotkey/düzenler, APK bağlamı |

## P4.4 — Ses paneli omurgası ✔ (2026-10-20, host + cihaz koşumu)

| Kayıt | Değer |
|---|---|
| Kod commit | [`7fb1fc3d1`](https://github.com/velnoxy/velnosub/commit/7fb1fc3d1) (feature dalı; öncül WIP `23c81033b`) |
| ABI yüzeyi | 17 → 26 sembol: `audio_open(yol, rate, ch)` (s16 PCM açım + tek-geçiş 256-kare kova peak taraması), `audio_len/len64`, `audio_info` ("s16 {rate}Hz {ch}k {N} örnek" dizgesi), `audio_peaks(h, d0, adet, k, minv, maxv)`, `audio_play(h, d0, adet)`, `audio_state`, `audio_stop`, `audio_close` |
| Oynatma mimarisi | **Arka-iplikli kurulum**: `play` yalnız oynatıcıyı kurar ve `std::thread(ses_kurulum_ipligi).detach()` fırlatır, hemen 0 döner; iplik fopen→fseek→openStream→start yapar. Gerekçe: proot/edarlisiz ortamlarda `openStream` binder'da süresiz bloke edebilir (P3.4'te senkron sürüm cihaz koşumunu gömmüştü) — çağıran iplik ASLA beklemez |
| Durum kodları | `-1` oynatma yok, `4` hazırlanıyor (kurulma benekli), `0` oynatılıyor, `1` boşalma, `2` hata; `AudioPlayer` akışı sahipler (`akis` alanı; yıkıcı dosya+akışı kapatır), `ses_kes_kilitli` kilitli iç kesici — play-içi stop yeniden kilitleyip ölü kilide düşme hatası düzeltildi |
| Dallama | `meson.build`: `oboe.pc` bulunursa `abi_oboe_dep` + `-DVELNOSUB_ABI_OBOE` → meson yoksa zarif yed (`[AVE] "oboe bu derlemede yok (masa kolu)"`); cross'ta gerçek play, host-check'ta yed |
| Host şeridi | `[AW0] ses açıldı (tutamaç: 1, rate 48000, ch 2)` `[AW1] s16 48000Hz 2k 48000 örnek (1.00 sn)` `[AW2] peak kovaları: 64 (boş olmayan 64, min<=max true)` `[AVE]` `[AW5] kol kapandı` — RC=0 |
| Cihaz şeridi | proot birebir RC=0 + **paket içi bağımsız koşum** (`media/kisa.wav.pcm`): `[AW0..AW2]` yeşil, `[AW3] oynatma durumu: 2` (arka-iplik binder kısıtını hızlı hata olarak raporlar — senkron sürümün süresiz gömülmesi bitti), `[AW4] kesildi`, `[AW5]`; P3.4 proot konum ARTIK belgeli: APK bağlamı gerçek çalmayı açar |
| P3.5 kapanışı | dalga/spektrum "hesaplamayı yeniden kullan, çizimi UI'ya bırak" maddesi: peak defteri çekirdekte (ABI), çizim egui'de |
| egui AudioSection | `Panel::top("wpanel")`: `▶ oynat` (ilk ≤5 sn)/`⏹ kes`/durum adları, **512-kova Painter waveform** (`Painter::line_segment` çubukları; `allocate_exact_size` çizim dikdörtgeni), `VELNOSUB_AUDIO` env |
| Sıradaki | P4.5 stil yöneticisi + dialog (Faz 5.3 widget dilimi), APK bağlamı (NativeActivity paketleme yolu) |

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

## P4.6 dilim 1 — libass önizleme ABI'sı ✔ (2026-10-20, host + cihaz)

| Kayıt | Değer |
|---|---|
| Kod commit | [`3dcae0830`](https://github.com/velnoxy/velnosub/commit/3dcae0830) (feature dalı) |
| Yeni ABI | `velnosub_abi_render_open(h, gen, yuk, font_yolu, font_ailesi)` + `velnosub_abi_render_frame(h, ms, RGBA sekizlisi deposit, boy)` + `velnosub_abi_render_close(h)` — nm defteri 42→**45** sembol |
| Meson | `tools/cross/meson.build` abi_deps'a `velnosub_libass_dep` eklendi — libass alt projeden **statik** bağlanır (cihaz paketine ek .so gerekmez; DT_NEEDED AYNI KALIR: core/m/log/c++_shared/dl/c) |
| Şart mimarisi | tutamaç başına `RCtx{ASS_Library*, ASS_Renderer*, ASS_Track*}` harita; yeniden açılışta önceki şart kapatılır; bellek dizge serileştirme: `AssStyle::GetEntryData()` + `AssDialogue::GetEntryData()` (okuma-kolu, fabrika dokunmuyor) → `[Script Info]+[V4+ Styles]+[Events]` → `ass_process_data` — **düzenlenmiş DURUM birebir taşınır** (edit_line/insert_line sonrası önizleme doğru) |
| Font düzeni | fontconfig cihazda YOK: `ass_add_font` blob (font yolu parametreli, aile varsayılan "Default") + `ass_set_fonts(..., ASS_FONTPROVIDER_NONE, ...; nokta-of-example: default font dosyası)` — **Fontname eşleşmese bile ass_set_fonts'in default font dosyası glyphleri çözüyor** (ass-render tool'da --defaultfont verilmezse `[AR2]` 0 piksel — bu notun kökeni) |
| Kare kompozisyonu | P2.3 pipeline: `ass_render_frame(ms)` → ASS_Image listesi → kapsama*etki kompozit (uint32 0xFFRRGGBB iç kare) → RGBA (r,g,b,255) sekizli deposit yazımı; deposit boyu katı denetimi (`n == gen*yuk*4` — yoksa hata), dönüş glyph etki pikseli (0 = boş kare, hata değil) |
| Regresyon | abi-smoke `[AB9]` şeridi: 240x180 açık → ms {0,8k,16k,24k,32k} taraması en yüksek glyph etki **9937 (host+cihaz BİREBİR)** → uzak gele (60 dk) kare 0 → deposit boy reddi → kapanış; ayrıca `Ekle sil dene` + `Clean Tags` device regresyonları yerinde (RC=0) |
| Tool lehine kanıt | cihaz ass-render `--t 8 --defaultfont DejaVuSans.ttf` → `[AR2] glyph pikseli 11569`, PPM 691.215 B — libass cihazda gerçek kare basıyor |
| UI bağlanma | ✔ **P4.6 dilim 2 ile yapıldı** (aşağıda: egui ColorImage overlay) |
| Paket | 86.904.159 B, 37 dosya; ABI 45 sembol |

## P4.6 dilim 2 — egui ASS önizleme (ColorImage overlay) ✔ (2026-10-20, C kompozit + Rust kol)

| Kayıt | Değer |
|---|---|
| Commit'ler | C: [`e78cf9ca9`](https://github.com/velnoxy/velnosub/commit/e78cf9ca9) (dizgi-alfa kompozit + close sızıntı kapanışı), Rust: [`9cc02a7ed`](https://github.com/velnoxy/velnosub/commit/9cc02a7ed) (Oniz kolu + video paneli overlay) |
| C kolu | `velnosub_abi_render_frame` iç kare artık **dizgi alfa** (0xAARRGGBB, zemin alfa 0; over kompozit: `out_a = sa + da*(255-sa)/255`, `out_rgb = (src*sa + dst*w1)/oa`) → egui saydam dokusu video üstüne biner; eski çıkışta bölge arka plan siyah opaktı (video gizlenirdi); glyph sayacı `etki > 0` koşuluna bağlı olduğundan metrik sabit |
| [AB9] birebir teyit | alfa kompoziti sonrası host **9937** + cihaz (taze .so ile pkg'da koşum) **9937** — BİREBİR; uzak-gele kare 0 ve deposit-boy reddi kanca satırları şeridin tamamını doğruladı |
| Sızıntı kapanışı | `velnosub_abi_close` artık tutamaç kapanınca `g_r` şartını da yumlatır — reload'da RCtx sızması kapandı |
| Rust kol | `VideoSection` +% `fps: f64`; `Oniz{h/gen/yuk/acik/denendi/son_ms/etki/tampon/dok/hata}`; `oniz_cek`: kol değişiminde durum sıfırlanır (C zaten kapattığından ayrı kapatma çağrısı yok), ölçü değişiminde eski şartı kapatır, açılış tek denemede (`VELNOSUB_FONT` + aile "Default" — [AB9] konvensiyonu), ms `kare*1000/fps`, ms değişmedikçe yeniden render yok; hata şeritte weak satır olarak gösterilir |
| Overlay çizimi | `video_icerik`: video dokusu rect'i `ui.image` response'tan yakalanır; `ui.put(rect, Image)` ile `etki > 0` iken işlem — etkinin 0 olduğu boş karelerde işleme yok (boş doku geçişi yok); masaüstü + mobil Video sekmesi AYNI yol |
| Kanıt | dört kanlı: host x86_64/aarch64/x2 (üç meson kanlı RC=0; abi-smoke host cihaz 9937 birebir), egui host RC=0 + aarch64 RC=0 (8.175.232 B) + test 2 passed; egui penceresi ekransız ortamda koşamaz — görsel doğrulama `[UI8]` adımına kaldı (aşağıda tabloda) |
| Paket | 86.955.311 B / 37 dosya (velnosub-ui 5.651.784 B taze) |

## P4.7 — Arayüz düzeni seçimi (ilk giriş + Ayarlar + mobil sekme çerçevesi) ✔ (2026-10-20, host derleme; etkileşimli koşum ekransız)

Kullanıcı kararı: "ilk girişte mobile uygun bir UI seçimi; mobil kullanıma
daha uygun bir UI olsun; ayarlardan değiştirilebilir." antislop **AFTER**
modu işledi (önce kurulum, ardından numaralı denetim — aşağıda).

| Kayıt | Değer |
|---|---|
| Kod commit'leri | [`1a433efdd`](https://github.com/velnoxy/velnosub/commit/1a433efdd) (özellik), [`e639946f8`](https://github.com/velnoxy/velnosub/commit/e639946f8) (Cargo.lock), [`f485ba75d`](https://github.com/velnoxy/velnosub/commit/f485ba75d) (imla temizliği) |
| Ayar deposu | `velnosub-ui.json` (çalışma dizini; `VELNOSUB_UI_JSON` ile yol geçilebilir) — `{"ui": "mobil|masaustu"}`; dosya yok → ilk giriş seçim ekranı; bilinmeyen içerik → yeniden sorar; yazma hatası oturumu bozmadan dürüst gösterilir |
| İlk giriş ekranı | iki 96pt yükseklikli seçim kartı (Mobil UI / Masaüstü UI) + gerçek işlevi anlatan açıklamalar + ayar dosyası yolu satırı; seçim dosyaya kalıcı yazılır ve oturum anında düzene geçer |
| Mobil düzen | alt sekme çubuğu `Panel::bottom("vtabs")` — 44pt+ sekme yüksekliği, kalan genişliğe oranlı sekme genişlikleri (dar pencere taşıma yok), seçili vurgusu; sekme gizliliği gerçek içeriğe göre (R-24: Video/Ses/Stil/Form yalnız yüklüyse; Altyazı+Ayarlar her zaman); geçersiz kalan aktif sekme Altyazı'ya düşer |
| Mobil grid | dikey reflow: zaman üstte, metin tam genişlik (satır içi yatay taşma yok), satır yüksekliği 3.2x gövde (≥44pt dokunma alanı); masaüstünde mevcut yatay satır AYNI |
| Mobil şeritler | Kaydet/Yeniden aç dolu genişlik 44pt düğme; video ◀/▶, ses ▶/⏹, form düğmeleri 44pt yükseklik; masaüstü boyutları değişmedi |
| Ayarlar bölümü | canlı düzen seçici (seçili vurgulu iki 44pt düğme, kalıcı yazım, hatada uyarı), "İlk seçim ekranını tekrar göster" gerçek eylem, ayar dosyası yolu satırı |
| Tek kaynak | şerit gövdeleri `video/ses/stil/form/durum/grid_icerik` yöntemlerine çıkarıldı; masaüstü çerçevesi (`masaustu_cerceve`) ve mobil çerçevesi (`mobil_cerceve`) AYNI gövdeyi paylaşır — fark yalnız `mobil` bayrağı (hedef boyu + satır düzeni); masaüstü yerleşimi görsel olarak değişmedi |
| Cargo.toml | android dışı hedeflerde eframe `x11` kabiliyeti — winit `aarch64-linux-gnu` host derlemesindeki "platform not supported" düzeltildi; android AArch64 yolu etkilenmez (android-native-activity kalır) |
| Kanıt | host `cargo build -p velnosub-ui --release` RC=0; `cargo test` → 1 passed (`ayar_yoksa_secim_kaydi_birebir`: dosya yok→seçim, yazım/okuma birebir, bilinmeyen→tekrar sor); `--target aarch64-linux-android --release` RC=0 (8.067.368 B) |
| Koşum dürüst notu (R-35) | etkileşimli GUI pencere koşumu bu ortamda mümkün değil (DISPLAY yok, Xvfb yok); gerçek dokunma doğrulaması APK bağlamı fazından sonra cihazda koşulacak |
| antislop tayini | AFTER modu teslimatı: [`anti-slop/audit-001-2026-10-20.md`](../../anti-slop/audit-001-2026-10-20.md) — numaralı bulgu listesi (1-3 YÜKSEK: dar sekme kırpılması, kontrast ölçümsüzlüğü, etkileşimli koşum eksiği); düzeltmeler yalnız kullanıcı onayladığı numaralara uygulanır |
| Paket | 86.918.580 B / 37 dosya (velnosub-ui yenilendi); ABI 45 sembol değişmedi |

### P4.7 Δ — denetim düzeltmeleri ✔ (2026-10-20, kullanıcı "tümünü düzelt" onayıyla)

| Kayıt | Değer |
|---|---|
| Kod commit'leri | [`abacb542a`](https://github.com/velnoxy/velnosub/commit/abacb542a) (düzeltmeler) + [`9d3476892`](https://github.com/velnoxy/velnosub/commit/9d3476892) (yorum imlası) |
| #1 sekme kırpılması | uyarlanır metin boyu `gen*0.22, 9–15.5pt` + `button_padding 4px` + alt genişlik 48→44pt: 300px'te 6 sekme kırpılmasız (R-03 dokunma alanı korunur) |
| #2 kontrast | `parlaklik`/`kontrast` (WCAG 2.1 göreli parlaklık) + `stil_tamir`: egui varsayılan soluk metin koyu zeminde **2.27:1 (ihlal tespit)** → tamir sonrası koyu **7.41**, zaman-gri `from_gray(170)` **7.41** (iki kol da değişti), hata rengi **5.64**, açık tema **6.49** — hepsi ≥4.5 (test kanıt: `kontrast_tamiri_aa_uyumlu`) |
| #3 + #6 koşum/ölçüm | şimdi koşulamaz (DISPLAY yok); APK bağlamı için `[UI0..]` cihaz şeridi planı aşağıda kayıtlı — APK fazının ilk işi |
| #4 klavye/erişim | masaüstünde de Ayarlar erişilebilir: durum çubuğuna "Ayarlar" düğmesi + `egui::Window` modal (mobil Ayarlar gövdesiyle AYNI `ayarlar_icerik`); egui'nin yerleşik odak çerçevesi devrede — cihaz şeridine görsel doğrulama kayıtlı |
| #5 satır metin | tespit: egui 0.37 `TextEdit::singleline` `clip_text: true` — Denetimim varsayıldığı gibi iç yatay kaydırma çubuğu zaten YOK; metin kırpılır, imleç takip eder. Düzeltme değil kayıt: davranış egui ürünü olarak doğru |
| #7 açık tema | `Tema{Koyu,Acik}` + ayar dosyası `{"ui","tema"}` ikili alan; `tema_sec` anında `all_styles_mut` + `Visuals::dark()/light()` + `stil_tamir` + kalıcı yazım; `main.rs` açılışta kayıtlı temayı uygular; Ayarlar'da Koyu/Açık 44pt seçim düğmeleri |
| #8 yön belgesi | [`DESIGN.md`](../../DESIGN.md) yazıldı: kimlik, renk/tipografi temelleri, iki resmî düzen, doğrulama reçeteleri |
| #9 kayıt | kullanıcı yüzü metinlerinde jargon yok — ihlal değil; kod yorumlarındaki derin jargon (44pt referansları) DESIGN.md ile bağlandı |
| Test | `cargo test` → **2 passed** (`ayar_dongusu` + `kontrast_tamiri_aa_uyumlu`), RC=0; host + aarch64 release RC=0 (aarch64 ikili 8.167.536 B) |
| Paket | 86.948.448 B / 37 dosya; ABI 45 sembol değişmedi |

### P4.7 [UI0..] — cihaz koşum şeridi planı (APK bağlamı fazının ilk işi)

| Adım | Beklenen |
|---|---|
| `[UI0]` ilk açılış (ayar dosyası silinmiş) | seçim ekranı çıkar: iki kart (Mobil UI / Masaüstü UI) + ayar yolu satırı; hiçbir şerit çizilmez |
| `[UI1]` Mobil seçilir | alt sekme çubuğu: yalnız Altyazı+Ayarlar; Altyazı bölmesinde durum satırı + "dosya açılmadı"; `velnosub-ui.json` yazılmış |
| `[UI2]` dosya açılır (`VELNOSUB_OPEN`) | sekmeler büyür: Altyazı/Stil (+Video/Ses yüklenmişse); grid dikey satırlar ≥44pt; uzun satırda iç yatay kaydırma YOK (clip) |
| `[UI3]` Ayarlar sekmesi | düzen seçici seçili "Mobil UI" vurgulu; tema seçici uygulanır ve kalıcı |
| `[UI4]` Masaüstü'ne geç | şeritler üst üste + durum çubuğu + "Ayarlar" düğmesi; grid yatay iki sütun; pencere açıkken düzen değiştirilebilir |
| `[UI5]` yeniden açılış | seçim ekranı ÇIKMAZ (kayıtlı düzen+tema uygulanır); açılış teması: ekran görüntüsüyle kontrast ölçü teyidi |
| `[UI6]` açılış süresi ölçümü (bulgu 6) | gerçek boyutlu .ass dosyasında `UI2` adım süresi ölçülür; makul üzerinde ise yükleme durumu ekranı dilimi açılır |
| `[UI7]` regresyon | satır düzenle → Commit (`[AG0..AG5]` şeridiyle aynı dosyada) — düzen seçimi veriyi bozmaz |
| `[UI8]` P4.6 dilim 2 önizleme | video aç → Video sekmesi: seçili ms'de altyazı metni video üstünde saydam binner; ◀/▶ kare değişiminde doku tazelenir, boş ms'de binme yok; `VELNOSUB_FONT` pkg'da DejaVuSans.ttf |
