# Velsub Faz 2 — Android Cross-Build (plan + künye)

Faz 2 hedefi (master plan §4): NDK r26+ ile arm64-v8a + x86_64 (emülatör)
çapraz derleme; çıktı `libaegisub-core.so` + basit CLI test
(ASS aç → kaydet → libass ile kare render).

## P2.1 — Cross bootstrap (**tamam**, `29ae8cb2e`)

| Karar | Not |
|---|---|
| Mobil hedef seçeneği | `-Dvelsub_mobile` (meson_options.txt); açılınca GUI hedefleri ve CLI only |
| Paylaşılan kitaplık | `shared_library('velsub-core', link_whole: [libaegisub, aegisub_core])` — root meson |
| Cross dosyaları | `tools/cross/android-arm64.txt`, `tools/cross/android-x86_64.txt` (system=android, API 29, `pkg_config_libdir` prefix bazlı) |
| Bağımlılık betiği | `tools/cross/build-deps.sh` — ICU 75 (host build + cross; statik veri) + Boost 1.84 (b2 bootstrap, `--with filesystem/regex/locale`) per-ABI |
| NDK | r29, **aarch64-host** tampasi (SnowNF o.0.2 sürümü — resmi linux NDK yalnız x86_64 host sağlar; aarch64 binary'lerle klasör adı `linux-x86_64` korunmuş, binary ELF AArch64) |

Kurulum dizini: `/root/velsub/ndk-deps/<abi>/{include,lib,pkgconfig}`.

## P2.2 — CLI + cross zincir (✔ tam link, `edb3e2d75`)

Çıktılar (`build-android-arm64/`):
- `libvelsub-core.so` — 42.8 MB, ELF64 **AArch64** DYN; NEEDED: wrap-zinciri
  `.so`'lar (freetype/fribidi/harfbuzz/uchardet) + bionic (m/dl/z/c) +
  `libc++_shared.so`
- `tools/cross/velsub-cli` — 555 KB, AArch64 PIE; NEEDED: `libvelsub-core.so`,
  `libc++_shared.so`, libdl, libc

| İş | Commit | Not |
|---|---|---|
| `tools/cross/velsub-cli.cpp` | [`48af0fb22`](https://github.com/velnoxy/velnosubs/commit/48af0fb22) | gtest'siz: GetReader→ReadFile→Commit(std)→WriteFile→tur dokunuşu; cihazda `VELSUB_CONFIG_JSON` ile gerçek default_config.json ister; `-DAGI_CORE` ile derlenir |
| Cross gating | [`0b1fe29f5`](https://github.com/velnoxy/velnosubs/commit/0b1fe29f5) | root meson: wx/vendor-subproject ve LuaJIT+luabins GUI-only; fontconfig GUI-only |
| OpenGL gating | [`86ade834c`](https://github.com/velnoxy/velnosubs/commit/86ade834c) | GUI-only (`dep_gl`); masaüstünde değişiklik yok |
| Çekirdek libass wrap fallback | [`6884ba5ae`](https://github.com/velnoxy/velnosubs/commit/6884ba5ae) | `dependency('libass', fallback: ['libass','libass_dep'])` — cross'ta wrap alt-proje (freetype+hb+fribidi zinciri zaten provide'lı) |
| Lua TU'ları libaegisub'da gate | [`a29eabf32`](https://github.com/velnoxy/velnosubs/commit/a29eabf32) | 7 lua birimi yalnız GUI kol; Faz 5'te geri döner |
| PIC + boost.pc | [`b4342588e`](https://github.com/velnoxy/velnosubs/commit/b4342588e) | ICU/Boost statik arşivleri `-fPIC`; b2 .pc üretmediği için boost.pc betikten üretilir |
| Boost modülleri + cxxstd | [`ea0eff8d9`](https://github.com/velnoxy/velnosubs/commit/ea0eff8d9)…[`5bb97d636`](https://github.com/velnoxy/velnosubs/commit/5bb97d636) | b2'ye `--with-system` ve **`cxxstd=17`** (C++03 modunda static constexpr üyeler dış sembole düşüyor — core ile dil modu uyumsuzdu); boost.pc `Libs` hem system'i hem bütün modülleri listeler; cross-file `boost_includedir/boost_librarydir` ile sistem-boost gölgesi kesildi |
| Boost link düzeni | [`edb3e2d75`](https://github.com/velnoxy/velnosubs/commit/edb3e2d75) | cross'ta boost `find_library` zinciriyle (system öncelikli, 7 arşiv) explicit link; host kolunda `boost_dep` |
| Mobil çekirdek TU seti | [`edb3e2d75`](https://github.com/velnoxy/velnosubs/commit/edb3e2d75) | core'un referans verdiği wx'siz tanım TU'ları mobil kolda core'a katıldı: `selection_controller.cpp`, `text_selection_controller.cpp` (AGI_CORE kapıları), `video_provider_cache/dummy/yuv4mpeg`, `core/options_globals.cpp` (config::opt/path) |
| utils.h AGI_CORE ayrımı | [`edb3e2d75`](https://github.com/velnoxy/velnosubs/commit/edb3e2d75) | wx yüzeyi (PrettySize / seçim / IO / IME) korumada; CleanCache + SmallestPowerOf2 mobil çekirdeğe açık |
| CLI host saniteyesi | [`6ccd0a5ef`](https://github.com/velnoxy/velnosubs/commit/6ccd0a5ef) | `agi::dispatch::Init` (inline thunk) + `agi::log::LogSink` CLI'da kurulur; config::opt/path tek tanım `.so`'ya taşındı; `file_mapping.h` `<memory>` |

**Host CLI duman koşumu (dokümana kanıt):** `ninja -C build-host-check tools/cross/velsub-cli` →
`VELSUB_CONFIG_JSON=src/libresrc/default_config.json` ile `docs/specs/ass-format-tests.ass`
→ *41 satır, 2 stil; aç → düzenle → kaydet → tur tamamlama* → 345/351 test baseline'ıyla
aynı ABI; CLI çıktısında sahne işaretçileri (`[0]…[3]`) cihaz testinde de aynı bekleniyor.

## Fondör (takılıp aşılan kapılar)

1. **PATH glob expand**: `PATH=...*/bin:$PATH` atamada glob açılmaz, `$(echo ...)` gerekir
2. **fontconfig root şartı** — GUI listeyicisi için; core kullanmaz → gate
3. **libass sistem font provider** — Android'de yok → `require-system-font-provider=false` (`ass_add_font` yolu)
4. **vendor LuaJIT meson'u Android'i 'Unsupported'** sayar → mobil kol çizgisel atlar (Faz 5'te tekrar)
5. **meson boost özel-yordam sytem-boost görürse** `-I/usr/include` zehirler → boost.pc koşuluyla pkg-config baskın; yine de meson libleri düşürüyor → cross-file `boost_includedir/librarydir` + sonunda **find_library explicit zinciri**
6. **Boost 1.84 GIL başlığı** clang 22 kurallarında kırılıyor → `::template apply(` → `::apply(` minimal yama (yalnız cross paket kurulumunda)
7. **proot fork limiti** → uzun derlemelerin -j1 / düşük paralelide koşması
8. **Boost arşivleri C++03 modunda derlenmişti** — `system_cat_holder::instance` static constexpr üyeleri dil moduna göre dış sembole düşüyor, core C++17 bekliyordu → b2 `cxxstd=17`
9. **libboost_system kurulmamıştı** — cat_holder tanımları system arşivinde; `--with-system` + `.pc`/`find_library` listesinde başa
10. **core TU'ları GUI sembolleri referans veriyor** — SelectionController/TextSelectionController/CreateCacheVideoProvider/config::opt path → wx'siz tanım TU'ları (selection/text_selection/video cache/dummy/yuv) mobil core'a katıldı; `config::opt|path` için yeni `core/options_globals.cpp`
11. **utils.h wx yüzeyi** — video_provider_yuv4mpeg gibi TU'lar CleanCache'e ihtiyaç duyuyor; wx yüzeyi AGI_CORE ile kapanıp wx'siz yardımcılar açık
12. **CLI'da GUI singleton'ları yok** — `agi::dispatch::Init` (`::service` null → SerialQueue SEGV) ve `agi::log::LogSink` (`log::Log` null yazımı) GUI main.cpp'te kuruluyor; CLI ikisini kendi başlatır (mobil app Faz 4'te aynı kalıbı uygular)

## Cross kurulum komutları (arm64 örnek)

```sh
export PATH=$(echo /root/velsub/ndk/r29/toolchains/llvm/prebuilt/*/bin):$PATH
NDK_ROOT=/root/velsub/ndk/r29 tools/cross/build-deps.sh arm64
meson setup build-android-arm64 \
  -Dvelsub_mobile=true \
  -Dalsa=disabled -Dopenal=disabled -Dlibpulse=disabled -Dportaudio=disabled \
  -Dffms2=disabled -Dbestsource=disabled -Dvapoursynth=disabled -Davisynth=disabled \
  -Dfftw3=disabled -Dhunspell=disabled \
  -Dlibass:require-system-font-provider=false -Dlibass:fontconfig=disabled \
  --cross-file tools/cross/android-arm64.txt
ninja -C build-android-arm64 libvelsub-core.so tools/cross/velsub-cli
```

x86_64 için `build-android-x86_64` + `android-x86_64.txt` (emülatör hedefi).

## Yapılacaklar (Faz 2 kapanışı öncesi)

- [x] PIC'li ICU/Boost arşivleriyle `libvelsub-core.so` linki biter → ELF aarch64 Android doğrulaması (llvm-readelf) ✔
- [x] `velsub-cli` HOST duman koşumu (ass-format-tests.ass 41 satır) ✔
- [ ] `velsub-cli` cihaz duman testi (bkz. [`docs/mobile/faz2-device-test.md`](faz2-device-test.md)) — paket: `.so` + CLI + `libc++_shared.so` (NDK'dan) + wrap-zinciri `.so`'lar (build-android-arm64/subprojects/**)
- [ ] libass kare render künyesi (P2.3): `velsub-ass-render` (ass_add_font + ass_render_frame)
- [ ] x86_64 ABI derlemesi (emülatör çıktısı)
- [ ] masaüstü doğrulaması: utils.h/selection kapıları diffuse edilmişken incremental build + core smoke testi

## Künye eki (2026-10-18)

- Adlandırma revizyonu: `velsub → velnosub`, commit [`997c5819a`](https://github.com/velnoxy/velnosub/commit/997c5819a). Kod deposu artık bağımsız repo [`velnoxy/velnosub`](https://github.com/velnoxy/velnosub); eski fork `Velsub-fork` adıyla arşivde durur. Faz 2 künye ve koşum tabloları o tarihli snapshot olarak `velsub-*` adlarını muhafaza eder: bu satırlar o commit'lerin kaydıdır, sonradan bozunmamıştır.
- P2.3: `velnosub-ass-render` kodu commit [`f1deb8ada`](https://github.com/velnoxy/velnosub/commit/f1deb8ada). Kapsam: libass `ass_add_font` + `ass_render_frame`, PPM (P6) çıktı, `ASS_FONTPROVIDER_NONE` (fontconfig'siz; cihazla aynı yol). Host koşum kanıtı ayrı satırda işlenecektir.
- Arayüz yığını kararı: Qt 6 yerine Rust (egui/eframe). Gerekçe ve Faz 4 paketleri master plan §3/Faz 4'te.
- Push durumu: GitHub OAuth token'ına `workflow` scope onayı bekler; `velnosub` repona push bir onay ile tek turda açılacaktır. Komiteler yerelde mantıklı dilimlerle yazıldı.
