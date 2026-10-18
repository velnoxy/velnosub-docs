# velnosub

**velnosub**, [Aegisub](https://github.com/arch1t3cht/Aegisub)'ın mobil (Android/iOS) ve
ilerleyen fazlarda Qt 6 ön yüzlü yeniden mühendislik portudur. Kod tabanı:
[arch1t3cht/Aegisub](https://github.com/arch1t3cht/Aegisub) (`feature` dalı, `9bfd500`).

Port stratejisi "core-first" (çekirdek öncelikli): önce wx'siz ASS çekirdeği ayrıştırılır
ve kendi kendine yeten bir ikili hale gelir; mobil kollar bu çekirdeğin üstüne kurulur.
Lua/MoonScript otomasyonu (`aegisub.*`, DependencyControl, autoload) bölünmeden korunur:
LuaJIT'in aarch64 JIT/FFI mimarisi Faz 5'e kadar saklanır, çekirdekten hiçbir şey çıkmaz.

## Depo haritası

| Depo | İçerik |
|---|---|
| [velnoxy/velnosub](https://github.com/velnoxy/velnosub) | Kod (bağımsız depo, `feature` dalı); hedefler `libvelnosub-core.so` + `velnosub-cli` (GUI Faz 3'te); Aegisub atfı korunur |
| **velnoxy/velnosub-docs** (bu depo) | Plan/analiz belgeleri; künyeler ve doğrulama kanıtları |

## Durum (2026-10-18)

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
| Faz 2 / P2.3 (kare render) | ✔ kod (koşum aşağıda) | [`f1deb8ada`](https://github.com/velnoxy/velnosub/commit/f1deb8ada) | `velnosub-ass-render`: libass `ass_add_font` + `ass_render_frame`, PPM çıktı, `ASS_FONTPROVIDER_NONE` (fontconfig'siz) |

## Koşum kanıtları

- **Android arm64-v8a** (`build-android-arm64/`): `libvelnosub-core.so` (42.8 MB, ELF64
  AArch64 DYN) ve `tools/cross/velnosub-cli` (555 KB, AArch64 PIE). NEEDED listesi:
  `libc++_shared.so` (NDK), bionic (`libm/libdl/libz/libc`) ve wrap-zinciri
  (`libfreetype/libfribidi/libharfbuzz/libuchardet`).
- **Host CLI duman koşumu:** [`docs/specs/ass-format-tests.ass`](https://github.com/velnoxy/velnosub/blob/feature/docs/specs/ass-format-tests.ass)
  girişiyle 41 satır, 2 stil; aç → düzenle → kaydet → tur-okuma geçti.
- **Masaüstü:** `--reconfigure` yeşil; incremental build yeşil; `velnosub core smoke` geçti;
  345/351 test baseline muhafaza.

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
- [`anti-slop/audit-001-2026-10-18.md`](anti-slop/audit-001-2026-10-18.md): Faz 1 anti-slop denetimi (bulgular 1-3 uygulandı)
- [`aegisub-upstream-YERINDEN-TASINDI.md`](aegisub-upstream-YERINDEN-TASINDI.md): kaynak deposunun taşınma gerekçesi

## Yol haritası

Faz 2'nin kalanı: cihaz duman testleri (arm64-v8a cihazda `velnosub-cli` ve
`velnosub-ass-render`), x86_64 ABI çıktısı. **P2.3** kodu atıldı
(`velnosub-ass-render`), host koşumu sıraya alındı. Sonra: Faz 3 medya katmanı
(FFmpeg video/ses sağlayıcı), Faz 4 mobil UI (Rust/egui kararı künyeli),
Faz 5 automation (LuaJIT aarch64), Faz 6 iOS, Faz 7 paketleme ve test.

## Atıf / lisans

Aegisub tabanlıdır; Aegisub GPLv3, libaegisub/libass/ffms2 BSD/MIT, atıflar velnosub'ta
korunur. `velnoxy/velnosub` bağımsız bir depodur (fork değil): Aegisub kaynak
deposuna yalnızca atıf yapılır.
