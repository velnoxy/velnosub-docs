# Faz 4.8 — Asıl uygulama APK'si: egui UI cdylib + NativeActivity

Tarih: 2026-10-21 · Orkestratör: **AI** (ruling akışı; kullanıcı yalnızca APK kurulumu/yayın adımlarında onay verir) · Kapsam: **faz4 kataloğu geniş** (grid + stiller + form + önizleme + ses/video iskeletleri, mevcut düzen sistemiyle)

## Doğrulama bulgusu (planı değiştirdi)

Eski planın 1. dilimi "panel kodu kayboldu, yeniden kur" idi. Ağaç denetimi yanlışlığını gösterdi:

- `velnosub-ui/src/abi.rs` **1863 satır**, içersinde mevcut faz4 arayüzseti: `VideoSection` (217), `Oniz` (231), `GridApp` (507), `Sekme` (539), `video_icerik` (972), `ses_icerik` (1046), `form_icerik` (1224), stiller/form/durum gövdeleri.
- `velnosub-ui/src/main.rs` 48 satır: bootstrap + tema uygulama (P4.7 künyesiyle uyumlu: "main.rs açılışta kayıtlı temayı uygular").
- `git log --follow velnosub-ui/src/main.rs` sadece P4.0/P4.2/P4.7 göstergesti — çünkü panel kodu P4.2'den beri abi.rs'de.
- aarch64 bin (eski kuyu) 8.07 MB, PIE, `ANativeActivity_onCreate` YOK (bin formatındaki beklenti).

**Revize plan:** dilim 1 "kurtarma" değil **doğrulama**; asıl yeni iş cdylib + NativeActivity APK.

## Mimari kararlar (ruling)

1. **Canonical lane:** `/root/velsub/aegisub-upstream/velnosub-ui` (feature dalı, ağaç temiz, 0 değişiklik). `/root/velsub/velnosub-ui` eski bina kuyusu (gerçek repo değil → atıl; üzerine yazılmayacak).
2. **cdylib + bin birlikte:** `[lib] name="velnosub_ui", crate-type=["cdylib"]`, bin yolunda main.rs kalır (host/karar dev). lib.rs `pub mod abi;` taşır; main.rs lib'i çağırır.
3. **Android girişi:** native-activity glue (android-activity) `ANativeActivity_onCreate`'ü cdylib'den export eder; uygulama `#[no_mangle] fn android_main(app: AndroidApp)` sağlar; NativeOptions → `event_loop_builder` içinden `with_android_app(app)`. API adları **locked eframe 0.37 kaynağından** doğrulanır (~/.cargo/registry; uydurma alandan değil).
4. **APK kimlik:** paket `com.velnoxy.velnosubui` (automation alpha'yla yanyana kurulabilir), versionCode 4, label "velnosub UI", minSdk 24 / targetSdk 28, `extractNativeLibs="true"` — mevcut pipeline sırası değişmez (aapt → zip lib → zipalign 4 → apksigner).
5. **Alpha dosya kapsamı:** SAF/JNI dosya seçici kuyrukta (ayrı faz); alpha = AssetManager'dan paketlenmiş örnek `.ass`+font → filesDir → aç, grid düzenle, filesDir'e Kaydet/Yeniden aç. Künye dürüst yazar.
6. **Test disiplini:** host `cargo test` şeridi RC=0; aarch64 `cargo build --release` + `llvm-nm -D` `ANativeActivity_onCreate` araması; APK imzala → `/mnt/sdcard/Download/` kopyala → kullanıcı telefonda kurar.

## Görevler

### T1 — Doğrulama (host)
- `cd /root/velsub/aegisub-upstream/velnosub-ui && cargo check && cargo test`
- Kanıt: test şeridi adları + RC. Bitince T2'yi besler.

### T2 — cdylib dönüşümü + android_main
- Cargo.toml: `[lib]` bölümü; lib.rs oluştur (`pub mod abi;` + android girişi); main.rs tek satıra düşür (`velnosub_ui::calistir()`).
- eframe 0.37 Locked API kontrolü: `find ~/.cargo/registry/src -path "*eframe-0.37*" -name "*.rs"` → NativeOptions alan adları (event_loop_builder / cached_size_to_point vs.) gerçek kaynaktan.
- `cargo check` (host) + `cargo build --target aarch64-linux-android --release` (cdylib .so).
- Kanıt: `llvm-nm -D release/libvelnosub_ui.so | grep ANativeActivity_onCreate` → T satırı; `cargo test` RC=0 hâlâ.
- Commit + push (feature).

### T3 — NativeActivity APK
- Yeni: `tools/cross/apk-ui/` (manifest + build-apk.sh, mevcut alpha script'in yol/kimlik uyarlaması).
- jniLibs: `libvelnosub_ui.so` + NEEDED zinciri (libvelnosub-abi.so, core, log, c++_shared, dl, c). assets: DejaVuSans.ttf + örnek .ass + (gerekiyorsa) style örnekleri; aapt 0.2 yol düzeni.
- Kurulum kanıtı: install dmdrug yok — cihazda kullanıcı kurar; script sonunda `apksigner verify` + `zipalign -c` ekrana.

### T4 — Künye + push (her iki repo)
- `docs/mobile/faz4-ui-plan.md` "Bekleyen paketler" → yapıldı kaydı; bu belge revizyonla bağlanır.
- Commit + push (docs main).

### T5 — Son denetim (superlite final-reviewer)
- T1..T4 paket üzerinde yeni taze göz; YALNIZCA okuyan, rapor veren ajan.

## Bitiş ölçütü
- `libvelnosub_ui.so` aarch64, `ANativeActivity_onCreate` export'u kanıtlandımış.
- `velnosub-ui-alpha.apk` (`com.velnoxy.velnosubui`, v4) imzalı, `/mnt/sdcard/Download/` içinde.
- Kullanıcı telefonda kurar; ekran görüntüsü şeridine göre sonraki alfa (SAF dosya seçici vb.) planlanır.
