# velnosub Faz 3 — Medya katmanı planı ve künyeleri

Master plan Faz 3 hedefleri: FFmpeg tabanlı kare-doğru video sağlayıcı, FFmpeg decode →
PCM önbellek (disk tabanlı) → Oboe oynatma, mevcut spektrum/dalga hesap kodunun yeniden
kullanımı, dahili keyframe üretimi (VapourSynth/Avisynth mobilde devre dışı).

Bu dosya planı ve künyeleri tutar; şu an uygulama başlanmadı (bekleme: x86_64 dep
massası build-deps.sh x86_64 sırasında).

## Dilim planı

- **P3.0** — FFmpeg aarch64 cross bootstrap: `tools/cross/build-ffmpeg.sh`
  (build-deps.sh örneğine benzer kabuk; statik headless libs: `--disable-programs
  --disable-doc --disable-network --disable-avdevice`, seçili demuxer/decoder seti).
- **P3.1** — `velnosub-video-probe` CLI: dosyayı aç, ilk N kareyi decode et,
  marker satırlarıyla (çözünürlük, kare sayısı, ilk kare PTS) duman.
- **P3.2** — video sağlayıcı TU mobil kolda `video_provider_ffmpeg` (kare-doğru seek:
  ts tabanlı arama + çerçeveli önbellek), host-check'te derlenir.
- **P3.3** — Ses decode → PCM disk önbellek CLI dilimi (`velnosub-audio-probe`),
  büyük dosya için parça-parça okuma kanıtlama.
- **P3.4** — Oboe oynatma (arm64 cihaz dumanı; x86_64 için AAudio yolu).
- **P3.5** — Spektrum/dalga hesap döngüsü UI katmanına çizim teslim arayüzü.

## Künyeler

Her dilim kapandığında buraya commit + koşum kanıtı satırı işlenir.
