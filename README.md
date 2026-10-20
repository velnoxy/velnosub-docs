# velnosub

**velnosub**, [Aegisub](https://github.com/arch1t3cht/Aegisub)'ın mobil (Android)
yeniden mühendislik portudur (iOS 2026-10-20'de kapsam dışı). UI yığını
Rust/**egui-eframe** olarak seçilmiştir (ilk seçenek Qt 6'nın bu konakta
ilerletilme zorluğu → karar kaydı master plan
§3'te). Kod tabanı:
[arch1t3cht/Aegisub](https://github.com/arch1t3cht/Aegisub) (`feature` dalı, `9bfd500`).

Port stratejisi "core-first" (çekirdek öncelikli): önce wx'siz ASS çekirdeği ayrıştırılır
ve kendi kendine yeten bir ikili hale gelir; mobil kollar bu çekirdeğin üstüne kurulur.
Lua/MoonScript otomasyonu (`aegisub.*`, DependencyControl, autoload) bölünmeden korunur:
LuaJIT'in aarch64 JIT/FFI mimarisi Faz 5'e kadar saklanır, çekirdekten hiçbir şey çıkmaz.

## Depo haritası

| Depo | İçerik |
|---|---|
| [velnoxy/velnosub](https://github.com/velnoxy/velnosub) | Kod (bağımsız depo, `feature` dalı); hedefler `libvelnosub-core.so` + `velnosub-cli` + `velnosub-ass-render` (medya Faz 3, UI Faz 4); Aegisub atfı korunur |
| **velnoxy/velnosub-docs** (bu depo) | Plan/analiz belgeleri; künyeler ve doğrulama kanıtları |

## Durum (2026-10-19)

| Faz / Paket | Durum | Kod commit | Not |
|---|---|---|---|
| Faz 0 (keşif) | ✔ | | 344 src dosyasının wx bağımlılık sınıflandırma haritası; masaüstü build+test baseline 345/351 |
| Faz 1 / P1.1 | ✔ | [`c2aef9bed`](https://github.com/velnoxy/velnosub/commit/c2aef9bed) | wx'siz `aegisub-core` statik lib (`-DAGI_CORE`) |
| Faz 1 / P1.2 | ✔ | [`c77d23cb1`](https://github.com/velnoxy/velnosub/commit/c77d23cb1) | `ass_style` çekirdeğe katıldı; `GetEncodings` UI'a taşındı |
| Faz 1 / P1.3 | ✔ | [`5f129ca99`](https://github.com/velnoxy/velnosub/commit/5f129ca99) | compat ABI (SingleChoice/Confirm/FileDialog/Clipboard) + 5 TU |
| Faz 1 / P1.4 | ✔ | [`9f90c87e5`](https://github.com/velnoxy/velnosub/commit/9f90c87e5) | runtime ABI (runner/main-thread/Timer) + CoordTranslator; mkv/ffms/libass yolları çekirdekte |
| Faz 1 / P1.5 | ✔ | [`4501d4d95`](https://github.com/velnoxy/velnosub/commit/4501d4d95) | font ölçüm ABI (headless; masaüstü wx korundu) |
| Faz 1 / P1.6 | ✔ | [`d718cf2a7`](https://github.com/velnoxy/velnosub/commit/d718cf2a7) | `velnosub core smoke`: ASS aç → düzenle → kaydet turu (wx'siz çekirdek ikilisi) |
| Faz 1 (denetim) | ✔ | [`b8a318177`](https://github.com/velnoxy/velnosub/commit/b8a318177) | audit-001 bulguları 1-3 uygulandı; 4-6 bilgi olarak kaldı |
| Faz 2 / P2.1 (bootstrap) | ✔ | [`29ae8cb2e`](https://github.com/velnoxy/velnosub/commit/29ae8cb2e) | `-Dvelnosub_mobile` + `libvelnosub-core.so` hedefi; arm64/x86_64 cross dosyaları; `tools/cross/build-deps.sh` (ICU 75 + Boost 1.84) |
| Faz 2 / P2.2 (CLI + cross zincir) | ✔ link + host smoke | [`48af0fb22`](https://github.com/velnoxy/velnosub/commit/48af0fb22)...[`6ccd0a5ef`](https://github.com/velnoxy/velnosub/commit/6ccd0a5ef) | `velnosub-cli` (ASS aç → düzenle → kaydet); mobil kolda GUI/wx/LuaJIT/GL/fontconfig kapısı kapalı; libass wrap fallback; PIC'li statik arşivler ve boost.pc ([`b4342588e`](https://github.com/velnoxy/velnosub/commit/b4342588e)); boost `cxxstd=17` + system + find_library zinciri ([`ea0eff8d9`](https://github.com/velnoxy/velnosub/commit/ea0eff8d9)...[`edb3e2d75`](https://github.com/velnoxy/velnosub/commit/edb3e2d75)); CLI singleton'ları ([`6ccd0a5ef`](https://github.com/velnoxy/velnosub/commit/6ccd0a5ef)) |
| Faz 2 / revizyon (adlandırma) | ✔ | [`997c5819a`](https://github.com/velnoxy/velnosub/commit/997c5819a) | `velsub → velnosub` revizyonu (`-Dvelnosub_mobile`, `libvelnosub-core.so`, `velnosub-cli`, `velnosub core smoke`); kod deposu bağımsız `velnoxy/velnosub` |
| Faz 2 / P2.3 (kare render) | ✔ kod + host koşum | [`f1deb8ada`](https://github.com/velnoxy/velnosub/commit/f1deb8ada) | `velnosub-ass-render`: libass 0.17 `ass_process_data` + `ass_add_font` + `ass_render_frame`, PPM; üç düzeltme: mobil `libresrc` kapısı ([`3b170e897`](https://github.com/velnoxy/velnosub/commit/3b170e897)), boş karede PPM ([`e01a6e7eb`](https://github.com/velnoxy/velnosub/commit/e01a6e7eb)), `--defaultfont` fallback ([`5da6fd29b`](https://github.com/velnoxy/velnosub/commit/5da6fd29b)) |
| Faz 2 / P2.2 yeniden kanıt (ek IV) | ✔ | [`5da6fd29b`](https://github.com/velnoxy/velnosub/commit/5da6fd29b) | arm64 sıfır-temiz tam koval: `ninja 484/484` (`A64BUILD=0`); üç artefakt readelf doğrulamalı; künye eki III-IV faz2 planında |
| Test çerçevesi | ✔ | [`d878c06af`](https://github.com/velnoxy/velnosub/commit/d878c06af) | gtest `protocol: 'gtest'` kaybı bulundu ve onarıldı; masaüstü baseline 345/351 birebir yeniden üretildi |
| Faz 4 / P4.0 (UI bootstrap) | ✔ | [`e4e69a0da`](https://github.com/velnoxy/velnosub/commit/e4e69a0da) | Rust/egui-eframe 0.37, 223 crate, aarch64-PIE artifact 7.431.800 B (NEEDED yalnız libdl/libandroid/libm/libc); proot koşumu: NEEDED çözülür, `libicu.so` apex yolu proot bind'ı olmadığından duran (APK bağlamı sıradaki) |
| Faz 4 / P4.1 (C ABI şeridi) | ✔ host + **cihaz** koşum | [`564d9739e`](https://github.com/velnoxy/velnosub/commit/564d9739e) | `libvelnosub-abi.so` + `velnosub-ui/abi-smoke`; şerit `[AB0..AB5]` `CAPI=0` — host VE proot'ta Rust-bionic ikiliyle birebir; künye faz4 planında |
| Faz 4 / P4.2 (grid + edit) | ✔ host + **cihaz** koşum | [`4c623cf2a`](https://github.com/velnoxy/velnosub/commit/4c623cf2a) | ABI 12 sembole büyüdü (`line_text/line_times/edit_line`, COMMIT_DIAG_TEXT hattı); Rust smoke şeridi `[AG0..AG5]` host+cihaz birebir (ikinci-tutamaç kalıcılığı dahil); egui GridApp (sanallaştırılmış liste, hücre düzenleme, Kaydet/Yeniden aç); künye faz4 planında |
| Faz 4 / P4.3 (video paneli omurgası) | ✔ host + **cihaz** koşum | [`04ede790e`](https://github.com/velnoxy/velnosub/commit/04ede790e) | ABI 17 sembol (`video_open/info/geom/frame/close`; P3.2 sağlayıcının cihazdaki İLK koşumu); `[AV0..AV4]` şeridi host+cihaz birebir — mpeg4 çözgü, 307.200 B BGRA kare, PTS geri-sarma byte-birebir; egui VideoSection doku paneli; künye faz4 planında |
| Faz 4 / P4.4 (ses paneli omurgası) | ✔ host + **cihaz** koşum | [`7fb1fc3d1`](https://github.com/velnoxy/velnosub/commit/7fb1fc3d1) | ABI 26 sembole (`audio_open/len/len64/info/peaks/play/state/stop/close`); **arka-iplikli Oboe kurulum** — play hemen döner, openStream proot binder bloke etmez; durum `-1/4/0/1/2`; `[AW0..AW5]` host+cihaz+paket-içi yeşil (cihazda `[AW3]` durum 2: proot kısıtı belgeli, gerçek çalma APK bağlamında); egui AudioSection dalga paneli (512-kova Painter waveform); künye faz4 planında |
| Faz 5 / P5.0 (LuaJIT zemini) | ✔ host + arm64 + x86_64 | [`46cbecd32`](https://github.com/velnoxy/velnosub/commit/46cbecd32) | host `[AU0..AU3]` koşumu + arm64/x86_64 LuaJIT cross kurulumu; `velnosub-automation` arm64 763.168 B, x86_64 801.264 B; reconfigure clearcache tuzağı çözüldü; künye faz5 planında |
| Faz 3 / P3.0–P3.4 + P3.1 destek (FFmpeg+probe+sağlayıcı+oboe+mkmpeg) | ✔ arm64 + x86_64 + **cihaz dumanı yeşil** | [`5fd29e77c`](https://github.com/velnoxy/velnosub/commit/5fd29e77c) | statik headless FFmpeg 6.1.2: arm64 (libavcodec 124.293.294 B ...) + x86_64 (121.003.582 B; `--disable-x86asm`); `velnosub-video-probe` arm64 71.902.704 B / x86_64 71.518.992 B; `velnosub-audio-probe` (s16 PCM) arm64 71.915.040 B / x86_64 71.534.224 B; P3.2 sağlayıcı çekirdeğe bağlı (core.so arm64 117.401.312 B / x86_64 146.968.936 B, `CreateFFmpegVideoProvider` ihracı; `-Wl,-Bsymbolic`; masa `DESK_RC=0`); P3.4 `velnosub-audio-play` (oboe içsel, AAudio→OpenSLES yedeği) arm64 2.584.368 B / x86_64 2.448.544 B; P3.1 destek `velnosub-mkmpeg` (sentetik mp4, `e7dd229bb`); **proot-içi cihaz şeritleri**: `[AB0..AB5]` `[AU0..AU3]` `[AP0..AP2]` `[MK0..MK3]→[VP0..VP3]` yeşil — künye faz3 planında + faz2-device-test tablosu |
| Faz 2 / x86_64 | ✔ tam koval | [`1584bf29a`](https://github.com/velnoxy/velnosub/commit/1584bf29a) | b2 boost ABI-kimlik düzeltmesi (AArch64 sızması giderildi) + masa7 476/476: core.so 72.495.784 B (X86-64 DYN), cli 585.976 B, ass-render 223.040 B, abi.so 692.704 B (9 sembol); künye eki V |

## Koşum kanıtları

- **Android arm64-v8a** (`build-android-arm64/`, 484/484 tam koval):
  `libvelnosub-core.so` (42.813.912 B, ELF64 AArch64 DYN), `tools/cross/velnosub-cli`
  (599.976 B, PIE) ve `tools/cross/velnosub-ass-render` (230.432 B, PIE). NEEDED
  listesi: `libvelnosub-core.so` (ikililer), `libc++_shared.so` (NDK), bionic
  (`libm/libdl/libz/libc`) ve wrap-zinciri
  (`libfreetype/libpng16/libfribidi/libharfbuzz/libuchardet`).
- **Android x86_64** (`build-android-x86_64/`, 476/476 tam koval):
  `libvelnosub-core.so` (72.495.784 B, ELF64 X86-64 DYN), `tools/cross/velnosub-cli`
  (585.976 B), `tools/cross/velnosub-ass-render` (223.040 B) ve
  `tools/cross/libvelnosub-abi.so` (692.704 B; 9 tanımlı `velnosub_abi` sembolü).
  Künye eki V.
- **Host CLI duman koşumu:** [`docs/specs/ass-format-tests.ass`](https://github.com/velnoxy/velnosub/blob/feature/docs/specs/ass-format-tests.ass)
  girişiyle 41 satır, 2 stil; aç → düzenle → kaydet → tur-okuma geçti.
- **Host kare-render koşumu (P2.3):** libass 0.17.4; `--defaultfont` fallback
  seçimi logda (`[ass] fontselect: (Arial, 400, 0) -> DejaVuSans.ttf`),
  `[AR2] glyph pikseli: 9203`, PPM `691.215 bayt` (640×360×3+15 başlık,
  `[AR3]`). Araştırmayla netleşen davranış: libass 0.17'de `ass_add_font` adı
  eşleşme anahtarı değildir; embed provider fontu dosyanın gerçek family'sinden
  indeksler. Künye eki III'te kayıtlıdır.
- **Masaüstü:** `--reconfigure` yeşil; incremental build yeşil; `velnosub core smoke` geçti;
  345/351 test baseline muhafaza.
- **Cihaz paketi:** `device-pkg.tar.gz` (22.560.092 B; iki ikili + 6 shared/lib
  `libc++_shared` dahil + test ASS + DejaVu font); reçete
  [`docs/mobile/faz2-device-test.md`](docs/mobile/faz2-device-test.md)'de.

Araç zinciri: SnowNF r29 aarch64-host NDK (API 29), ICU 75 arm64-v8a PIC'li statik
kurulum (`.pc` dahil), Boost 1.84 arm64 statik (7 modül: system/atomic/chrono/filesystem/
locale/regex/thread; GIL clang-22 yaması dahil). Aşılan kapıların künyeli tam listesi:
[`docs/mobile/faz2-android-cross-plan.md`](docs/mobile/faz2-android-cross-plan.md).

## Belge rehberi

- [`aegisub-mobile-port.md`](aegisub-mobile-port.md): master plan; kurallar ve fazlar (Faz 0'dan Faz 7'ye)
- [`velnosub-app/docs/mobile/wx-dependency-map.md`](velnosub-app/docs/mobile/wx-dependency-map.md): Faz 0 ürünü: 344 src dosyasının wx bağımlılık sınıflandırması
- [`velnosub-app/docs/mobile/faz1-core-split-plan.md`](velnosub-app/docs/mobile/faz1-core-split-plan.md): Faz 1 iş paketleri (P1.1-P1.6), künyeler, ABI kararları
- [`docs/mobile/faz2-android-cross-plan.md`](docs/mobile/faz2-android-cross-plan.md): Faz 2 cross-build planı, künyeler, aşılan 12 kapı
- [`docs/mobile/faz2-device-test.md`](docs/mobile/faz2-device-test.md): Faz 2 cihaz duman testi rehberi (paket listesi + koşum)
- [`docs/mobile/faz5-automation-plan.md`](docs/mobile/faz5-automation-plan.md): Faz 5 LuaJIT/automation dilimleri ve P5.0 künyesi
- [`anti-slop/audit-001-2026-10-18.md`](anti-slop/audit-001-2026-10-18.md): Faz 1 anti-slop denetimi (bulgular 1-3 uygulandı)
- [`aegisub-upstream-YERINDEN-TASINDI.md`](aegisub-upstream-YERINDEN-TASINDI.md): kaynak deposunun taşınma gerekçesi

## Yol haritası

Faz 2'nin kalanı: — tüm dumalar cihazda (proot) yeşil koşuldu. Faz 3 ✔
(P3.0–P3.4 + mkmpeg test-varlık üreticisi: arm64+x86_64; cihaz şeritleri
`[AP0..AP2]`, `[MK0..MK3]→[VP0..VP3]` dâhil; P3.4 akış açımı proot'ta
sınırlı — APK bağlamında ayrı doğrulanacak). Faz 4 mobil UI:
P4.0 Rust/egui bootstrap ✔, P4.1 C ABI ✔ (host+cihaz), **P4.2 grid+edit ✔
(host+cihaz `[AG0..AG5]`)**, **P4.3 video paneli ✔**, **P4.4 ses paneli ✔
(arka-iplikli Oboe, `[AW0..AW5]` host+cihaz+paket-içi; çalma APK
bağlamında)**; sırada P4.5 stil/dialog, P4.6 jestler/düzenler ve APK
bağlamı, Faz 5
automation ✔ (host+arm64+x86_64 + cihaz `[AU0..AU3]`; LuaJIT script
dumaları sıradaki), Faz 6 (iOS) 2026-10-20'de **iptal** — Faz 7 paketleme
ve test Android hedefiyle sürer.

## Atıf / lisans

Aegisub tabanlıdır; Aegisub GPLv3, libaegisub/libass/ffms2 BSD/MIT, atıflar velnosub'ta
korunur. `velnoxy/velnosub` bağımsız bir depodur (fork değil): Aegisub kaynak
deposuna yalnızca atıf yapılır.
