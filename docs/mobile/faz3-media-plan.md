# velnosub Faz 3 — Medya katmanı planı ve künyeleri

Master plan Faz 3 hedefleri: FFmpeg tabanlı kare-doğru video sağlayıcı, FFmpeg decode →
PCM önbellek (disk tabanlı) → Oboe oynatma, mevcut spektrum/dalga hesap kodunun yeniden
kullanımı, dahili keyframe üretimi (VapourSynth/Avisynth mobilde devre dışı).

Bu dosya planı ve künyeleri tutar.

## Dilim planı

- **P3.0** — FFmpeg aarch64 cross bootstrap ✔: `tools/cross/build-ffmpeg.sh`
  (build-deps.sh örneğine benzer kabuk; statik headless libs: `--disable-programs
  --disable-doc --disable-network --disable-avdevice`, seçili demuxer/decoder seti).
- **P3.1** — `velnosub-video-probe` CLI ✔ (arm64 ikili çıktı): dosyayı aç, ilk N kareyi decode et,
  marker satırlarıyla (çözünürlük, kare sayısı, ilk kare PTS) duman.
- **P3.2** — video sağlayıcı TU mobil kolda `video_provider_ffmpeg` (kare-doğru seek:
  ts tabanlı arama + çerçeveli önbellek), host-check'te derlenir.
- **P3.3** — Ses decode → PCM disk önbellek CLI dilimi ✔ (`velnosub-audio-probe`),
  büyük dosya için parça-parça okuma kanıtlama (arm64 + x86_64 ikili çıktı).
- **P3.4** — Oboe oynatma (arm64 cihaz dumanı; x86_64 için AAudio yolu).
- **P3.5** — Spektrum/dalga hesap döngüsü UI katmanına çizim teslim arayüzü.

## P3.0 + P3.1 künyesi — FFmpeg aarch64 kurulum + video-probe ikilisi (2026-10-19, 02:18/02:23)

- `tools/cross/build-ffmpeg.sh arm64` (JOBS=2; FFmpeg 6.1.2, statik headless): kurulum
  `ndk-deps/arm64-v8a/lib` → `libavcodec.a` 124.293.294 B, `libavformat.a` 38.434.070 B,
  `libavutil.a` 4.115.922 B, `libswscale.a` 4.707.174 B, `libswresample.a` 565.430 B
  (+ `libavcodec/libavformat/libavutil.pc`). LLVM readelf: `libavcodec.a` REL/AArch64.
- Seçilmiş yıldız kümesi: demuxer `mov,matroska,avi,ogg,flv,wav,ass,srt`; decoder
  `h264,hevc,vp8,vp9,aac,mp3,flac,opus,vorbis,pcm_s16le,pcm_f32le,srt`; parser
  `h264,hevc,aac,vorbis,opus,flac`; protocol yalnız `file`.
- P3.1 probe: arm64 build dir'e `meson configure --clearcache` (bağımlılık önbelleği
  tazelenir) + reconfigure + `ninja -j2` → `build-android-arm64/tools/cross/
  velnosub-video-probe` 71.902.704 B (ELF64, `Type: DYN`, `Machine: AArch64`; `-g`
  statik bağlı FFmpeg nedeniyle büyük — cihaz paketine strip'li gider).
- P3.0 x86_64 ✔ (02:41, `FFMX64_RC=0`): kurulum `ndk-deps/x86_64/lib` →
  `libavcodec.a` 121.003.582 B, `libavformat.a` 38.135.734 B, `libavutil.a`
  3.870.036 B, `libswscale.a` 4.942.186 B, `libswresample.a` 584.050 B (+ pc'ler);
  LLVM readelf `libavcodec.a` REL/X86-64.
- P3.1 x86_64 ✔ (02:43): `build-android-x86_64/tools/cross/velnosub-video-probe`
  71.518.992 B (`Type: DYN`, `Machine: Advanced Micro Devices X86-64`).
- P3.3 ✔ (02:50): `velnosub-audio-probe` her iki ABI'de — arm64 71.915.040 B
  (`DYN`/AArch64), x86_64 71.534.224 B. swresample ile s16 dönüşümü +
  parça-parça disk yazımı (`--seconds N`, varsayılan 5 sn; çıktı `<girdi>.pcm`).
  Commit'ler: TU + meson kapısı
  [`3f85e7897`](https://github.com/velnoxy/velnosub/commit/3f85e7897), imza
  düzeltmesi [`105a635f2`](https://github.com/velnoxy/velnosub/commit/105a635f2).
- Derinleşen bulgular:
  1. `ffbuild/version.sh` `... | tail -1` boru hattı proot kesintlerinde/mesaj
     yönlendirmelerinde asılı: make tek hedefte 1:33:08 bekledi (0 ek .o).
     `tail`'i öldürmek yetti; kök-düzeltme: `tools/cross/ffversion-fast.sh`
     (sabit sürüm yazan yardımcı; script artık make öncesi version.sh'i bununla
     değiştirir — [`212131afc`](https://github.com/velnoxy/velnosub/commit/212131afc)).
  2. `build-ffmpeg.sh` +x biti eksikken zincir `Permission denied` (126) ile öldü;
     commit [`77dac02da`](https://github.com/velnoxy/velnosub/commit/77dac02da).
  3. Koşan zincirin yürüttüğü betik dosyası o anda elle düzenlenirse bash'in
     tembel ayrıştırması bozulan yerleşimde takıldı (`line 62: syntax error near
     unexpected token ')'`): zincir kaydı DEAD gösterdi ama make tamamlanmıştı
     (kurulum 02:18'de birebir). Ders: koşum başladıktan sonra yürüyen betiği
     düzenleme; düzeltmeyi ayrı commit ile SONRAKİ koşuma bırak.
  4. x86_64 FFmpeg configure nasm ister: `nasm/yasm not found or too old` →
     x86_64 moduna `--disable-x86asm` eklendi
     ([`9571e3039`](https://github.com/velnoxy/velnosub/commit/9571e3039));
     assembly'siz headless decode için yeterli (arm64'ün 1,5 saatinin tersine
     x86_64 kurulumu 8 dk'da bitti — asm TÜ'ler tamamen atlandı).
  5. `av_samples_alloc_array_and_samples` ilk argüman `uint8_t ***`
     ister: `uint8_t *dst[8]` yerine `uint8_t **dst` bildirimi gerekli
     (derleyici uyarısıyla yakalandı).

## Künyeler

Her dilim kapandığında buraya commit + koşum kanıtı satırı işlenir.
