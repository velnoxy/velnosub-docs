# Velsub — Planlama Belgeleri

**Velsub**, Aegisub'ın mobil (Android/iOS) ve Qt 6 tabanlı yeniden mühendislik portudur.
Kod tabani: [arch1t3cht/Aegisub](https://github.com/arch1t3cht/Aegisub) (`feature` dalı, `9bfd500`).
Bu depo yalnızca plan/analiz belgelerini tutar.

## Depo haritası

| Depo | İçerik |
|---|---|
| [velnoxy/Velsub](https://github.com/velnoxy/Velsub) | Kod — Aegisub fork'u; Faz 1 commit'leri burada (`feature` dalı) |
| **velnoxy/Velsub-docs** (bu depo) | Master plan, bağımlılık haritası, faz çalışma planı |

## Belge rehberi

- [`aegisub-mobile-port.md`](aegisub-mobile-port.md) — master plan: kurallar, fazlar (Faz 0–7)
- [`velsub-app/docs/mobile/wx-dependency-map.md`](velsub-app/docs/mobile/wx-dependency-map.md) — Faz 0 ürünü: 344 src dosyasının wx bağımlılık sınıflandırması, derleme/test raporu
- [`velsub-app/docs/mobile/faz1-core-split-plan.md`](velsub-app/docs/mobile/faz1-core-split-plan.md) — Faz 1 iş paketleri (P1.1–P1.6), P1.1 künyesi, ABI kararları
- [`docs/mobile/faz2-device-test.md`](docs/mobile/faz2-device-test.md) — Faz 2 cihaz duman testi rehberi (paket listesi + koşum)
- [`aegisub-upstream-YERINDEN-TASINDI.md`](aegisub-upstream-YERINDEN-TASINDI.md) — kaynak deposunun sdcard'dan iç dizine taşınma gerekçesi

## Faz durumu (2026-10-18)

| Faz / Paket | Durum | Kod commit | Not |
|---|---|---|---|
| Faz 0 (keşif) | ✔ | — | 344 dosya sınıflandırma haritası; masaüstü build+test baseline (345/351) |
| Faz 1 / P1.1 ✓ | ✔ | [`c2aef9bed`](https://github.com/velnoxy/Velsub/commit/c2aef9bed) | `aegisub-core` statik lib (`-DAGI_CORE`) |
| Faz 1 / P1.2 ✓ | ✔ | [`c77d23cb1`](https://github.com/velnoxy/Velsub/commit/c77d23cb1) | `ass_style` çekirdeğe katıldı; `GetEncodings` UI'a taşındı |
| Faz 1 / P1.3 ✓ | ✔ | [`5f129ca99`](https://github.com/velnoxy/Velsub/commit/5f129ca99) | compat ABI (SingleChoice/Confirm/FileDialog/Clipboard) + 5 TU |
| Faz 1 / P1.4 ✓ | ✔ | [`9f90c87e`](https://github.com/velnoxy/Velsub/commit/9f90c87e) | runtime ABI (runner/main-thread/Timer) + CoordTranslator; mkv/ffms/libass yolları çekirdekte |
| Faz 1 / P1.5 ✓ | ✔ | [`4501d4d9`](https://github.com/velnoxy/Velsub/commit/4501d4d9) | font ölçüm ABI (headless; masaüstü wx korundu) |
| Faz 1 / P1.6 ✓ | ✔ | [`d718cf2a7`](https://github.com/velnoxy/Velsub/commit/d718cf2a7) | `velsub core smoke`: ASS aç → düzenle → kaydet turu (wx'siz çekirdek ikilisi) |
| **Faz 1** | ✔ tamam + denetim | — | audit-001 bulguları 1–3 uygulandı ([`b8a318177`](https://github.com/velnoxy/Velsub/commit/b8a318177)); 4–6 bilgi olarak kaldı |
| Faz 2 / P2.1 (bootstrap) | ✔ | [`29ae8cb2e`](https://github.com/velnoxy/Velsub/commit/29ae8cb2e) | `-Dvelsub_mobile` + `libvelsub-core.so` hedefi; arm64/x86_64 cross dosyaları; `tools/cross/build-deps.sh` (ICU 75 + Boost 1.84) |
| Faz 2 / P2.2 (CLI + cross zincir) | 🚧 derleme | [`48af0fb22`](https://github.com/velnoxy/Velsub/commit/48af0fb22)...[`b4342588e`](https://github.com/velnoxy/Velsub/commit/b4342588e) | `velsub-cli` (gtest'siz ASS aç→düzenle→kaydet); GUI/wx/LuaJIT/GL/fontconfig kapıları mobil kolda kapalı ([`0b1fe29f5`](https://github.com/velnoxy/Velsub/commit/0b1fe29f5), [`86ade834c`](https://github.com/velnoxy/Velsub/commit/86ade834c)); libass wrap fallback ([`6884ba5ae`](https://github.com/velnoxy/Velsub/commit/6884ba5ae)); statik arşiv PIC + boost.pc ([`b4342588e`](https://github.com/velnoxy/Velsub/commit/b4342588e)) |

**Faz 2 araç zinciri durumu:** SnowNF'r29 aarch64-host NDK (resmi linux NDK yalnız x86_64 host sağlar) + ICU 75 cross (arm64-v8a, statik veri, `.pc`'ler) + Boost 1.84 arm64 statik kurulu; Android arm64 meson kurulumu geçti (ASET=0). Cross `.so` linki PIC yeniden derlemesiyle sürüyor; devre takvimi: kasımdan önce cihazda duman testi (bkz. [`docs/mobile/faz2-device-test.md`](docs/mobile/faz2-device-test.md)).

## Atıf / lisans

Aegisub tabanlıdır; Aegisub GPLv3, libaegisub/libass/ffms2 BSD/MIT — atıflar Velsub'ta korunur. Şeffaflık için fork ilişkisi açıktır: `velnoxy/Velsub` GitHub fork rozetiyle görünür.
