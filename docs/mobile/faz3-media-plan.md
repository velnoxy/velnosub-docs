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
- **P3.2** — video sağlayıcı TU mobil kolda `video_provider_ffmpeg` ✔ (kare-doğru
  GetFrame: ts tabanlı geri sarma + çerçeve önbelleği), çekirdek `.so`'ya bağlı
  (`FFmpeg` fabrika kaydı; masa koluna etkisiz).
- **P3.3** — Ses decode → PCM disk önbellek CLI dilimi ✔ (`velnosub-audio-probe`),
  büyük dosya için parça-parça okuma kanıtlama (arm64 + x86_64 ikili çıktı).
- **P3.4** — Oboe oynatma ✔ ikili (`velnosub-audio-play` iki ABI; oboe
  1.9.0 statik altyapısı da kurulu; cihaz dumanı reçete faz2-device-test'te;
  x86 ABI'sinde de bağlanır).
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
- P3.2 ✔ (03:48/03:50): `src/video_provider_ffmpeg.cpp` çekirdek kola katıldı —
  `VideoProvider` arayüzü; kare-doğru GetFrame (PTS tablosu birikimi ile ts
  geri sarma + tek kare önbellek + swscale BGRA), manager'a `FFmpeg` fabrika
  kaydı. Kapı yalnız `-Dvelnosub_mobile` + libav pc bulunduğunda açılır
  (`WITH_VELNOSUB_FFMPEG`); masa baseline'ı etkilenmez (`DESK_RC=0` iki kez).
  Kanıt: arm64 `libvelnosub-core.so` ~72 MB → **117.401.312 B**, x86_64 →
  **146.968.936 B**; `llvm-nm -D`: `CreateFFmpegVideoProvider` +
  `avformat_open_input` dışa açık. Commit
  [`5fd29e77c`](https://github.com/velnoxy/velnosub/commit/5fd29e77c).
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
  6. Çekirdekte PCH yok (`agi_pre.h` sadece wx kolu): conf tanımları çekirdek
     TU'suna akmaz ve `#ifdef`'lu TU BOŞ derlenir (784 B .o, sembolsüz).
     Çözüm: tanımı mesondan `cpp_args`'a açıkça beslemek
     (`-DWITH_VELNOSUB_FFMPEG=1`).
  7. FFmpeg `.a` bağının DSO kuralı: farklı TU'lardaki genel sembollere (fft
     tabloları) çapraz-nesne `adrp` atıfları `R_AARCH64_ADR_PREL_PG_HI21
     cannot be used against symbol` hatası verir — `-Wl,-Bsymbolic` bağ
     argümanıyla semboller .so içi bağlanır (interpoz hedeflenmiyor).
  8. FFmpeg 6.1'de `AVStream::display_aspect_ratio` yok —
     `sample_aspect_ratio` × çözünürlük oranından türet; `agi::BackgroundRunner`
     ileri bildirimi TU içinde yapılmalı.
  9. Ayrık zincir betiğinde PATH'i atlamak meson regen'i düşürür: cross clang
     PATH'ten aranır (`FileNotFoundError: 'aarch64-linux-android29-clang'`);
     zincir betiklerinde PATH ihracı bilinçli satır olmalı.
  10. Oboe CMake 4.x'te `cmake_minimum_required` < 3.5 kuralıyla düşer —
      `-DCMAKE_POLICY_VERSION_MINIMUM=3.5` geçiş çare
      (deprecation uyarısı kalır ama derlama birebir tamamlanır).

### P3.4 altyapı künyesi — Oboe 1.9.0 statik kurulum (2026-10-20, 04:03)

- Kaynak: google/oboe 1.9.0 tarball; CMake cross
  (`android.toolchain.cmake`, `ANDROID_PLATFORM=android-29`,
  `BUILD_SHARED_LIBS=OFF`); kurulum `ndk-deps/<abi>/` başlıklarla.
- Kanıt: arm64 `liboboe.a` 6.952.108 B (`Machine: AArch64`), x86_64
  6.601.164 B (`Machine: Advanced Micro Devices X86-64`); `include/oboe/`
  her iki prefix'te. cmake install ABI alt-dizinine koyar (`lib/<abi>/`) —
  prefix düzenine taşındı (`lib/liboboe.a`).

### P3.4 ikili künyesi — velnosub-audio-play (2026-10-20, 04:14/04:15)

- `tools/cross/velnosub-audio-play.cpp`: Oboe `AudioStreamDataCallback` +
  `AudioStreamErrorCallback` kalıtımıyla s16 PCM çalar (LowLatency önergesi;
  Shared paylaşım, cihaz Exclusive'e düşebilir). Meson kapısı `oboe.pc`
  üzerinden (oboe install çıktısına elle yazılan pc; pkg_config_libdir
  hattı); masa kolunda sistem oboe yok → hedef bilinçli atlanır.
- Kanıt: arm64 2.584.368 B (`DYN`/AArch64), x86_64 2.448.544 B; NEEDED yalnız
  `liblog/libc++_shared/libm/libdl/libc` (oboe statik). Commit'ler: TU +
  meson kapısı ve hata-arayüzü düzeltmesi — son hâli
  `git log --oneline -3` (feature dalında `velnosub: P3.4 velnosub-audio-play`
  başlıklı iki kayıt).
- Cihaz dumanı: `[OP0..OP3]` şeridi faz2-device-test.md'de; ham PCM
  metaverisiz olduğu için rate/kanal P3.3 `[AP1]` çıktısıyla eşleşir.

## Künyeler

Her dilim kapandığında buraya commit + koşum kanıtı satırı işlenir.
