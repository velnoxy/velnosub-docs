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

## P2.2 — CLI + cross zincir (🚧, `48af0fb22`…`b4342588e`)

| İş | Commit | Not |
|---|---|---|
| `tools/cross/velsub-cli.cpp` | [`48af0fb22`](https://github.com/velnoxy/Velsub/commit/48af0fb22) | gtest'siz: GetReader→ReadFile→Commit(std)→WriteFile→tur dokunuşu; cihazda `VELSUB_CONFIG_JSON` ile gerçek default_config.json ister; `-DAGI_CORE` ile derlenir |
| Cross gating | [`0b1fe29f5`](https://github.com/velnoxy/Velsub/commit/0b1fe29f5) | root meson: wx/vendor-subproject ve LuaJIT+luabins GUI-only; fontconfig GUI-only |
| OpenGL gating | [`86ade834c`](https://github.com/velnoxy/Velsub/commit/86ade834c) | GUI-only (`dep_gl`); masaüstünde değişiklik yok |
| Çekirdek libass wrap fallback | [`6884ba5ae`](https://github.com/velnoxy/Velsub/commit/6884ba5ae) | `dependency('libass', fallback: ['libass','libass_dep'])` — cross'ta wrap alt-proje (freetype+hb+fribidi zinciri zaten provide'lı) |
| Lua TU'ları libaegisub'da gate | [`a29eabf32`](https://github.com/velnoxy/Velsub/commit/a29eabf32) | 7 lua birimi yalnız GUI kol; Faz 5'te geri döner |
| PIC + boost.pc | [`b4342588e`](https://github.com/velnoxy/Velsub/commit/b4342588e) | ICU/Boost statik arşivleri `-fPIC`; b2 .pc üretmediği için boost.pc betikten üretilir |

## Fondör (bir zamanlar takılıp aşılan beş kapı)

1. **PATH glob expand**: `PATH=...*/bin:$PATH` atamada glob açılmaz, `$(echo ...)` gerekir
2. **fontconfig root şartı** — GUI listeyicisi için; core kullanmaz → gate
3. **libass sistem font provider** — Android'de yok → `require-system-font-provider=false` (`ass_add_font` yolu)
4. **vendor LuaJIT meson'u Android'i 'Unsupported'\u200b** sayar → mobil kol çizgisel atlar (Faz 5'te tekrar)
5. **meson boost özel-yordam sytem-boost görürse** `-I/usr/include` zehirler → boost.pc koşuluyla pkg-config baskın
6. **Boost 1.84 GIL başlığı** clang 22 kurallarında kırılıyor → `::template apply(` → `::apply(` minimal yama (yalnız cross paket kurulumunda)
7. **proot fork limiti** → uzun derlemelerin -j1 / düşük paralelide koşması

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

- [ ] PIC'li ICU/Boost arşivleriyle `libvelsub-core.so` linki biter → ELF aarch64 Android tehdidi doğrulaması (`file`/`readelf`)
- [ ] `velsub-cli` cihaz duman testi (bkz. [`docs/mobile/faz2-device-test.md`](faz2-device-test.md))
- [ ] libass kare render künyesi (P2.3): `velsub-ass-render` (ass_add_font + ass_render_frame)
- [ ] x86_64 ABI derlemesi (emülatör çıktısı)
