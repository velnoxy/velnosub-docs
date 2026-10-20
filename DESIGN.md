# DESIGN.md — velnosub arayüz yönü

Bu belge 2026-10-20 denetiminde eksiği tespit edilen (bulgu 8) yön
kaynağıdır; mevcut ürün dili uygulanıyor, yazılı hâle getiriliyor.
Arayüz: `velnosub-ui/` (egui/eframe).

## Kimlik

- **Ne**: Türkçe arayüzlü mobil öncelikli altyazı düzenleyici. Aegisub'un
  otomasyon davranışını (auto4/taslak komutları) dokunmatik yere taşır.
- **Ürün sesi**: sakin, kısa, emirle değil fiille konuşan ("Kaydet",
  "Yeniden aç"). kullanıcıya "sen" dili, jargon yok (yüzeyde).
- **Kategori** (3 diyapozon): enerji 2 aracı / ritim 1 sessiz / hareket 1
  minimal (animasyon yok — dokunuş tepkisi egui'nin seçili/hover
  durumlarıyla yetinir).

## Görsel temeller

- **Tema**: koyu varsayılan (araç kategorisi kuralı); açık tema seçenekli
  (Ayarlar → Tema). İkisi de `stil_tamir` ile AA uyumuna zorlanır.
- **Renkler**: egui tema paletleri temel; özel renkler yalnız anlam
  götürür: zaman gri `from_gray(170)` (koyu) / `from_gray(90)` (açık),
  hata `rgb(220,120,90)`. Marka motifi yok — kimlik Türkçe metin ve
  düzen titizliğinden gelir (motif üretimi ayrı dilim asla
  otomatik değil).
- **Tipografi**: default_egui_fonts; ayar ekranlarında başlık 18pt,
  gövde default; mobil sekme etiketi uyarlanır (9–15.5pt, sekme
  genişliği ile ölçeklenir).
- **Boşluk**: şeritler arası 4–6pt; kartlar 96pt yükseklik; hedefler
  ≥44pt (R-03; dar pencerede sekme 44pt alt genişlik uyarlanır).

## Düzen sistemleri (iki resmî düzen)

1. **Mobil** (varsayılan öneri): alt sekme çubuğu (R-24: boş sekme yok —
   Altyazı+Ayarlar hep, Video/Ses/Stil/Form yalnız yüklüyse); her sekme
   tek bölme; grid dikey reflow (zaman üstte, metin tam genişlik).
2. **Masaüstü** (miras): şeritler üst üste (video→ses→stil→form), alt
   durum çubuğu, sanallaştırılmış yatay grid; "Ayarlar" durum
   çubuğunda pencere olarak.

İki düzen aynı gövdeleri paylaşır (`*_icerik` yöntemleri) — fark yalnız
çerçeve yerleşimi ve dokunma boyu. Tek kaynağa bağlılık R-34 kuralıdır.

## Doğrulama reçeteleri

- **Kontrast**: `cargo test -p velnosub-ui` → `kontrast_tamiri_aa_uyumlu`
  ölçüyor (koyu 7.41 / zaman-gri 7.41 / hata 5.64 / açık 6.49 ≥ 4.5).
- **Mantık habercileri**: `ayar_dongusu` (dosya yok→seçim, dönüşüm,
  bilinmeyen→tekrar sor).
- **Cihaz şeridi** (APK fazından sonra): `[UI0..]` şerit listesi
  faz4-ui-plan P4.7 koşum planında.

## Yapılacaklar listesi değil, akıllar listesi

Bu belge kural listesi değil; koda yazılmış tercihler için bahane
belgesidir. Bunu değiştiren her şey künye ve test ile kanıtlanmalıdır.
