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
| **Faz 1** | ✔ tamam | — | anti-slop denetimi beklemede: `anti-slop/audit-001-2026-10-18.md`; sırada Faz 2 |

## Atıf / lisans

Aegisub tabanlıdır; Aegisub GPLv3, libaegisup/libass/ffms2 BSD/MIT — atıflar Velsub'ta korunur. Şeffaflık için fork ilişkisi açık £ `velnoxy/Velsub` GitHub fork rozetiyle görünür.
