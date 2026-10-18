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

- **Faz 0** ✔ tamam: repo klonu, tam masaüstü derleme (11.1 MB arm64), test 345/351 (6 adet `lagi_acs` uid-0 artefaktı), sınıflandırma haritası
- **Faz 1** ▶ sürüyor: **P1.1–P1.4 ✔** (çekirdek statik lib + string/geo + compat ABI'leri + runtime/koordinat ABI'leri; 27+ TU wx'siz) — sırada P1.5 (FontMeasure), P1.6 (CLI duman testi)

## Atıf / lisans

Aegisub tabanlıdır; Aegisub GPLv3, libaegisup/libass/ffms2 BSD/MIT — atıflar Velsub'ta korunur. Şeffaflık için fork ilişkisi açık £ `velnoxy/Velsub` GitHub fork rozetiyle görünür.
