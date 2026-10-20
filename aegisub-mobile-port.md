# velnosub — Aegisub Mobil Port Planı (Android; iOS 2026-10-20'de iptal)

> **2026-10-18 durum notu:** Kod deposu bağımsız repo
> [`velnoxy/velnosub`](https://github.com/velnoxy/velnosub) (fork ilişkisi bırakıldı; eski fork `Velsub-fork` arşiv olarak durur), belge deposu [`velnoxy/velnosub-docs`](https://github.com/velnoxy/velnosub-docs). Adlandırma revizyonu `velsub → velnosub` commit [`997c5819a`](https://github.com/velnoxy/velnosub/commit/997c5819a). Arayüz yığını kararı Qt 6'dan Rust (egui/eframe)'a döndü; gerekçe ve yeni Faz 4 paketleri §3 ve Faz 4'te.

> **2026-10-19 durum notu:** Faz 2 arm64 kolu sıfır-temiz kovayla kapandı
> (`ninja 484/484`; künye eki IV). P2.3 `velnosub-ass-render` host koşumu yeşil:
> glyph 9203, PPM 691.215 bayt, `--defaultfont` fallback; künye eki III. Faz 4
> P4.0 Rust/egui bootstrap kapandı ([`e4e69a0da`](https://github.com/velnoxy/velnosub/commit/e4e69a0da)).
> Gece ekleri: P4.1 C ABI şeridi host koşumu yeşil (`libvelnosub-abi.so` +
> Rust `abi-smoke`, `[AB0..AB5]` `CAPI=0`; faz4 planındaki künye) ve P5.0
> LuaJIT zemini host koşumu yeşil (`velnosub-automation`, LuaJIT
> 2.1.1791988261, `[AU0..AU3]`; faz5 planındaki künye; arm64+x86_64 LuaJIT
> cross kurulumları + automation ikilileri, reconfigure clearcache tuzağı
> çözüldü). Faz 2 x86_64: ABI merdiveni (ICU+Boost) ve masa 476/476 ✔
> (boost b2 ABI-kimlik düzeltmesi başlıca; künye eki V). Faz 3 arm64+x86_64
> ✔ (statik headless FFmpeg 6.1.2; x86'da `--disable-x86asm`); P3.1/P3.3
> probe ikilileri iki ABI'de; P3.2 `video_provider_ffmpeg` çekirdek .so'ya
> bağlı (`CreateFFmpegVideoProvider` ihracı, core.so arm64 117.401.312 B;
> `-Wl,-Bsymbolic` bağ düzeltmesi; masa DESK_RC=0); P3.4 oboe oynatma
> ikilisi iki ABI'de (oboe 1.9.0 statik altyapılı; AAudio→OpenSLES yedeği)
> ve P3.1 destek `velnosub-mkmpeg` (sentetik mp4 üreticisi — kullanıcı
> medyasına muhtaçlık kalktı); version.sh boru-tuzağı düzeltmesi. **KEŞİF
> (20-10):** bionic ikililer Termux proot'ta adb'siz DOĞRUDAN koşuyor —
> cihaz duman şeritleri `[AB0..AB5]`, `[AU0..AU3]`, cli/ass-render,
> `[AP0..AP2]`, `[MK0..MK3]→[VP0..VP3]` yeşil (P3.4 sadece akış açımı
> proot-kısıtlı; künye faz3 planı). Cihaz paketi (83.459.415 B) ve
> reçete: docs/mobile/faz2-device-test.md.
>
> **20-10 gündüz ekleri:** P4.2 grid+edit ✔ (ABI 12 sembole büyüdü,
> `[AG0..AG5]` şeridi host+cihaz birebir — ikinci-tutamaç kalıcılığı dahil;
> egui GridApp: sanallaştırılmış liste + hücre düzenleme; commit
> [`4c623cf2a`](https://github.com/velnoxy/velnosub/commit/4c623cf2a)).
> P4.3 video paneli omurgası ✔ (ABI 17 sembole; `[AV0..AV4]` şeridi
> host+cihaz birebir — P3.2 sağlayıcının cihazdaki İLK koşumu: mpeg4 çözgü,
> BGRA kare, PTS geri-sarma byte-birebir; egui VideoSection doku paneli;
> commit [`04ede790e`](https://github.com/velnoxy/velnosub/commit/04ede790e)).
> P4.4 ses paneli ✔ (peak ABI + arka-iplikli Oboe oynatma — play hemen
> döner, durum -1/4/0/1/2; proot'ta kurulum ipliği binder kısıtına
> `[AW3]` durum 2 verir, gerçek çalma APK bağlamına kalır; egui
> AudioSection dalga paneli; commit
> [`7fb1fc3d1`](https://github.com/velnoxy/velnosub/commit/7fb1fc3d1)).
> P4.5 stiller omurgası + tam editör ✔ (style_of 13-hanelik alan dizgesi
> + ad/boyut/font/renk/hiza yazma ABI'sı, COMMIT_STYLES; `[AS0..AS6]`
> host+cihaz+paket-içi birebir — kalıcılık ikinci tutamaçta teyitli;
> egui Stiller paneli DragValue/font kutusu/ana renk/hiza; commits
> [`5c62ee6bb`](https://github.com/velnoxy/velnosub/commit/5c62ee6bb) +
> [`c6fd3fcd2`](https://github.com/velnoxy/velnosub/commit/c6fd3fcd2)).
> P4.5 III dialog form omurgası ✔ (39 sembol: `dialog_open/…/close` mini
> DSL — `ctl|sınıf|ad|başlik|değer[|ek]` + `buton|metin` satırları, tipli
> doğrulama; `[AD0..AD6]` host+cihaz+paket-içi birebir; abi-smoke'un
> RC=134 quirk'ü 44 çıkış noktası `cik(h)` kapısıyla kökten kapandı;
> egui dinamik form paneli dpanel; commit
> [`03cbf985f`](https://github.com/velnoxy/velnosub/commit/03cbf985f)).
> P5.0 bireysel script koşumu cihazda kanıtlandı (`[AUL]` akışı).
> P5.1/P5.2 aegisub.* gerçek yüzeyi ✔ — shim Lua FFI ile ABI kolunda
> makro çalıştırıyor: dosya aç/düzenle/kaydet + dialog pompası; "Form
> dene" makrosu `DUMAN-FORMU @7` üretip kalıcılığı ikinci tutamacta
> teyit etti ([AUS1..AUS5] host+cihaz birebir; commit
> [`0328d339d`](https://github.com/velnoxy/velnosub/commit/0328d339d)).
> P5.3 script manager + gerçek upstream makroları ✔ — birebir kopya
> Strip tags/Clean Tags koştu ([AUS]/[AUZ] akışı; commit
> [`c362063a5`](https://github.com/velnoxy/velnosub/commit/c362063a5));
> aynı dilimde boost::flyweight çapraz-DSO yığın bozulması kökten
> giderildi (mutasyonlar core.so'da tek kolda; ABI 40 sembol line_meta).
> P5.2 devam ✔ — satır ekleme/silme hatta: `velnosub_abi_line_insert/
> delete` (ABI 42 sembol), `subs:insert_line/delete_line/n_lines`
> yöntemleri ve "Ekle sil dene" demo makrosu; `[AB8]` 41→42→41 şeridi
> host+cihaz birebir (commit
> [`8d4eb608a`](https://github.com/velnoxy/velnosub/commit/8d4eb608a)).
> P4.6 dilim 1 libass önizleme ABI ✔ — `velnosub_abi_render_open/frame/
> close` (ABI 45 sembol; libass statik); `[AB9]` şeridi host+cihaz
> birebir 9937 glyph etki, cihaz ass-render `--t 8 --defaultfont` kanıtı
> 11569 glyph / PPM 691.215 B (commit
> [`3dcae0830`](https://github.com/velnoxy/velnosub/commit/3dcae0830)).
> Rust-bionic zinciri cihaz kanıtı: abi-smoke proot'ta DOĞRUDAN koştu.
> eframe ikilisi proot'ta libicu/apex bind eksikliğinden durur (APK
> bağlamına kalır). P3.4 audio-play AAudio→OpenSLES yedeği eklendi;
> P3.1 reçetesi mkmpeg ile kullanıcı medyasından bağımsız. Paket
> 86.918.580 B (P4.7 yenilemeli: media/ test varlıkları + dialog-ornak
> spec + Rust ikilileri dahil).
> P4.7 Arayüz düzeni seçimi ✔ — kullanıcı kararıyla: ilk girişte (ayar
> dosyası yokken Mobil/Masaüstü iki kartla) düzen seçimi, Ayarlar'dan
> sonradan değiştirilebilir; mobil desen alt sekme çubuğu (44pt hedefler,
> R-24 boş sekme yok), grid dikey reflow; masaüstü yerleşimi birebir
> korundu, şerit gövdeleri tek kaynağa çıkarıldı. Kanıt: host RC=0 + test
> 1 passed + aarch64 RC=0; etkileşimli koşum ekran yok (APK fazına
> kaldı) — antislop AFTER modunda numaralı denetim
> `anti-slop/audit-001-2026-10-20.md` (commit
> [`1a433efdd`](https://github.com/velnoxy/velnosub/commit/1a433efdd)).
> P4.7 Δ düzeltmelerim ✔ — kullanıcı "tümünü düzelt" onayıyla: kontrast
> ölçüm kolu + tamir (egui varsayılan soluk metin koyu zeminde 2.27:1
> İHLAL tespit → 7.41; hata 5.64; açık 6.49), uyarlanır sekme metin
> boyu (300px/6 sekme kırpılmasız), AÇIK TEMA (ayar dosyasında "tema"
> alanı + Ayarlar'da seçim düğmeleri + açılış uygulaması), masaüstüne
> Ayarlar penceresi; [UI0..UI7] cihaz koşum şeridi planı APK fazına
> kaydedildi; DESIGN.md yön belgesi yazıldı (test 2 passed; commit
> [`abacb542a`](https://github.com/velnoxy/velnosub/commit/abacb542a)).
> P4.6 dilim 2 ✔ — egui video paneline ASS önizleme: render_frame dizgi-alfa
> kompozitine çevrildi (zemin alfa 0, over), [AB9] host+cihaz BİREBİR 9937
> korundu; abi_close artık render şartını da yumlatır (sızıntı kapandı);
> Rust tarafında Oniz kolu (fps ile kare→ms, ms değişmedikçe yeniden
> render yok, hata dürüst şeritte), kare ui.put ile videonun üstüne
> biner (commit [`e78cf9ca9`](https://github.com/velnoxy/velnosub/commit/e78cf9ca9) +
> [`9cc02a7ed`](https://github.com/velnoxy/velnosub/commit/9cc02a7ed)).

## 0. Proje bilgisi

- **Uygulama adı:** velnosub (Aegisub'un mobil portu / türevi)
- **Çalışma dizini:** Bu dosyanın bulunduğu klasör
  - Tüm çalışma (klon, kaynak, build çıktıları, dokümanlar) bu klasör altında yapılır
  - Aegisub deposu bu klasörün içine klonlanır (ör. `aegisub-upstream`), velnosub'a özgü kod ayrı dizinde tutulur (ör. `velnosub-app`)
  - Not: Android'in paylaşılan depolama alanında (ör. Download) scoped storage kısıtları ve çalıştırma/symlink izni olmaması derlemeyi etkileyebilir; derleme Termux/proot gibi bir ortamdan yapılıyorsa kaynak ve build dizinini uygulamanın kendi dahili dizinine kopyalamak gerekebilir
- Kullanıcıya görünen ad, paket adı, ikon, ayar dizinleri, log ve hata mesajlarında "Aegisub" yerine **velnosub** kullanılır; lisans bildirimleri ve orijinal yazar atıfları (GPL/BSD gereği) korunur ve "Aegisub tabanlıdır" notu Hakkında ekranında yer alır

Kaynak: https://github.com/arch1t3cht/Aegisub (branch: `feature`)
Taban: TypesettingTools/Aegisub → orijinal Aegisub
Dil dağılımı: C/C++ (~94%), Lua (~3%), MoonScript, Meson

## 1. Hedef

Aegisub'un **tam** işlevselliğini Android'de çalıştırmak (iOS 2026-10-20'de kapsam dışı):

- ASS/SSA/SRT vb. altyazı düzenleme (grid, edit box, stil yöneticisi, find/replace, timing araçları)
- Video önizleme + libass render (mevcut libass entegrasyonu)
- Ses dalga formu/spektrum, ses oynatma, ses ile timing
- Görsel typesetting araçları (konum, ölçek, döndürme, vektör klip, drag araçları)
- **Automation: Lua ve MoonScript scriptleri, makrolar, export filtreleri, karaoke templater, DependencyControl uyumu**
- Hotkey sistemi, ayarlar, çok dilli arayüz (`po/`)

Kabul kriteri: mevcut masaüstü Aegisub'da çalışan tipik bir typesetting scripti (ör. kara-templater, ILL/Yutils tabanlı scriptler) mobilde değişiklik olmadan çalışmalı.

## 2. Depo yapısı (okunacak yerler)

| Dizin | İçerik | Port notu |
|---|---|---|
| `libaegisub/` | Çekirdek kütüphane: ASS ayrıştırma, ayarlar, hotkey, zaman, charset, JSON, log | wx'e bağımlı olmamalı → doğrudan taşınır |
| `src/` | Ana uygulama, wxWidgets GUI, video/ses sağlayıcıları, komutlar, `auto4_lua*` | **Asıl iş burada.** GUI ve wx bağımlılıkları ayrıştırılacak |
| `automation/` | Dahili scriptler (`autoload/`), `include/` (Lua kütüphaneleri, moonscript.lua, DependencyControl vb.) | Neredeyse olduğu gibi kullanılır |
| `vendor/luabins/` | Lua veri serileştirme | NDK ile derlenir |
| `subprojects/` | Meson wrap bağımlılıkları | NDK için cross-build gerekecek |
| `packages/` | Paketleme betikleri | Android için yeni paket hedefi eklenecek |
| `po/` | Çeviriler | Qt/Android tarafında yeniden kullan (gettext → Qt Linguist veya gettext) |
| `tests/` | Testler | Çekirdek için CI'da çalıştır |

## 3. Mimari karar

wxWidgets mobilde resmi olarak desteklenmiyor. Bu yüzden:

**Katmanlı yaklaşım:**

1. **`aegisub-core` (C++, platformdan bağımsız)**
   - `libaegisub` + `src/` içinden GUI'ye bağlı olmayan her şey (subtitle model, undo, komut sistemi, video/ses sağlayıcı soyutlamaları, auto4 çekirdeği)
   - wx tiplerini (`wxString`, `wxRect`, `wxColour`, event sistemi vb.) soyutlama katmanıyla değiştir
2. **`velnosub-ui` (Rust, egui/eframe)** *(2026-10-18 revizyonu: Qt yerine Rust)*
   - C++ çekirdek (`libvelnosub-core.so`) üstünde ince bir `extern "C"` ABI katmanı; Rust (egui/eframe) bu ABI'ye bağlanır, `cargo-ndk` + `cargo-apk`/`xbuild` ile gradle'sız APK
   - Tek kod tabanı ile Android (+ masaüstü test); iOS 2026-10-20'de kapsam dışı
   - Karar gerekçesi: hedef konak (Termux+PRoot, aarch64) Qt 6 Android kaynak derlemesini praktikte taşımıyor ve gradle/aapt2 zinciri kapalı; egui/eframe bu konakta çalışan en kısa bootstrap yoludur. Eski Qt 6 kararı git geçmişinde durur
   - Alternatif (seçilmedi): Kotlin + Jetpack Compose + JNI (JNI köprüsü şart)
3. **Platform adaptörleri**
   - Ses çıkışı: Oboe (AAudio/OpenSL ES)
   - Video: FFmpeg (ffms2 veya doğrudan libav*) + isteğe bağlı MediaCodec hızlandırma
   - Dosya erişimi: Storage Access Framework (SAF), `content://` URI'leri için dosya tanımlayıcı katmanı
   - Font: Android sistem fontları + kullanıcı font klasörü → libass/fontconfig yapılandırması

Önerilen strateji: **önce çekirdeği headless derle ve test et, sonra UI.**

## 4. Aşamalar

### Faz 0 — Keşif (1–2 gün eşdeğeri)
- Depoyu klonla, masaüstünde Meson ile derle (`meson setup build --buildtype=release`)
- `src/` içinde wx bağımlılık haritası çıkar: hangi dosyalar `#include <wx/...>` kullanıyor, hangileri GUI'den bağımsız
- Çıktı: `docs/mobile/wx-dependency-map.md`

### Faz 1 — Çekirdeği ayır
- wx'siz çekirdek hedefi (`aegisub-core`) oluştur
- wx tiplerini ince bir soyutlama ile değiştir (string, rect, color, timer, event, config)
- Mevcut masaüstü uygulaması bu çekirdek üzerinde hâlâ derlenmeli (regresyon yok)
- Birim testleri çekirdek üzerinde geçmeli

### Faz 2 — Android cross-build
- NDK (r26+), arm64-v8a ve x86_64 (emülatör)
- Meson cross-file veya CMake wrapper; bağımlılıklar: boost, ICU, libass (freetype, harfbuzz, fribidi), FFmpeg, fftw3 (veya kissfft ile değiştirme seçeneği), hunspell, zlib, LuaJIT (Android) / Lua 5.1
- Çıktı: `libaegisub-core.so` + basit bir CLI test (ASS aç → kaydet → libass ile kare render)

### Faz 3 — Medya katmanı
- Video sağlayıcı: FFmpeg tabanlı, kare-doğru seek (BestSource mantığı veya ffms2 indeksleme)
- Ses sağlayıcı: FFmpeg decode → PCM önbellek (disk tabanlı, büyük dosyalar için) → Oboe oynatma
- Spektrum/dalga formu: mevcut hesaplama kodunu yeniden kullan, çizimi UI katmanına bırak
- Keyframe üretimi: dahili algoritma (VapourSynth/Avisynth masaüstü-özel, mobilde devre dışı)

### Faz 4 — Mobil UI (Rust/egui; 2026-10-18 revizyonu)
Paketler (her biri kendi commit + koşum kanıtıyla kapanır):

- **P4.0** — Rust araç zinciri + Android bootstrap: rustup + cargo-ndk, minimal egui/eframe uygulaması aarch64'te derlenir; `cargo-apk`/`xbuild` paketleme cihazda açılır
- **P4.1** — C ABI köprüsü: `aegisub-core` yüzeyine `extern "C"` şeriti (aç/düzenle/kaydet/render çağrıları) + Rust crate bağlanması
- **P4.2** — Altyazı grid'i + edit box (sanallaştırılmış liste, hücre düzenleme, çekirdek undo bağlantısı)
- **P4.3** — Video paneli: Faz 3 video sağlayıcısından gelen kare egui texture'ı; libass önizleme katmanı (P2.3 render yolunu çoğaltmadan)
- **P4.4** — Ses paneli: dalga formu/spektrum canvas çerçevesi; oynatma Oboe (arm64) / AAudio (emülatör)
- **P4.5** — Stil yöneticisi + dialog'lar; Faz 5 dinamik form eşlemesiyle aynı widget dilimi (dialog formları omurgası ✔ — `dialog_open/…/close` mini DSL + egui dpanel; Lua pompası Faz 5'in P5.2'siyle bağlanır)
- **P4.6** — Dokunma jestleri (pinch-zoom/pan, sürükleme araçları), harici klavye hotkey şeridi, tablet/telefon düzenleri, komut paleti

Eski Qt 6 metni git geçmişinde durur; Faz 4 çıktısı cihazda duman edilmeden kapanmaz.

### Faz 5 — Automation (scriptler dahil, kritik)
Mevcut yapı: `src/auto4_lua*.cpp` Lua motorunu barındırır, `automation/include` ve `automation/autoload` içinde kütüphane ve scriptler bulunur.

İlerleme (2026-10-20): **P5.0 LuaJIT zemini ✔** (host+arm64+x86_64, cihaz proot koşumu kanıtlı), **P5.1/P5.2 ilk dilim ✔** — `aegisub.*` gerçek yüzeyi LuaJIT FFI ile `libvelnosub-abi.so` kolu üzerinden: dosya aç/düzenle/kaydet (`subs[i]` newproxy-userdata, `#subs` işler), `aegisub.dialog.display` → P4.5 III dialog ABI pompası (tipli cevap dönüşü), demo makrolar host+cihaz birebir (`DUMAN-FORMU @7` + kalıcılık teyidi; künye faz5 planında), **P5.3 ilk dilim ✔** — manager dizin taraması (`--dir/--script/--include`, `[AUZ]` izleri), aegisub 2.x `include()` globali, `decode_path`, satır `class/comment` (`velnosub_abi_line_meta`, ABI 40 sembol) ve **gerçek upstream scriptler koşutu**: birebir kopya `strip-tags.lua` + `cleantags-autoload.lua` [+ `cleantags.lua`] manager akışında yüklenip Clean Tags 40 satırı temizledi (host+cihaz RC=0). Aynı dilimde **boost::flyweight çapraz-DSO yığın bozulması kökten giderildi**: Text/Style/Actor/Effect alanları refcounted flyweight — fabrika tekili DSO'ya göre bölünüyor (core.so F1 yükler, abi.so/cli F2/F3'ten yazınca yabancı düğüm erası → `free(): invalid size`/`double free`/SEGFAULT); tüm flyweight yazmaları artık core.so'da `velnosub_core_edit_*` vekilinde tek kolda (+ abi-smoke `[AB7]` tekrarlı-düzenleme regresyon şeridi). **P5.2 devam ✔** — satır ekleme/
silme hatta: `velnosub_core_insert_line/delete_line` vekilleri (flyweight
yazımı yine core.so'da), `velnosub_abi_line_insert/line_delete` (ABI 42
sembol), auto4 yöntemleri `subs:insert_line(konum, satır)`/
`subs:delete_line(konum)`/`subs.n_lines` ve "Ekle sil dene" demo makrosu
— `[AB8]` 41→42→41 + makro kalıcılığı host+cihaz birebir (commit
[`8d4eb608a`](https://github.com/velnoxy/velnosub/commit/8d4eb608a)).
**P4.6 dilim 1 ✔** — libass önizleme ABI'sı: `velnosub_abi_render_open/
frame/close` (ABI 45 sembol; libass alt projeden STATİK bağlandı),
başına bellek-dizge serileştirme (`GetEntryData` okuma-kolu) ile
düzenlenmiş durum önizlemeye taşınıyor; `[AB9]` şeridi 9937 glyph etki
host+cihaz **birebir**, ass-render cihaz kanıtı `--t 8 --defaultfont` →
11569 glyph / PPM 691.215 B (commit
[`3dcae0830`](https://github.com/velnoxy/velnosub/commit/3dcae0830)).
**P4.7 ✔ (kullanıcı dilimi)** — arayüz düzeni seçimi: ilk girişte (ayar
dosyası `velnosub-ui.json` yokken) Mobil/Masaüstü iki 96pt kartla seçim,
Ayarlar bölümünden sonradan değiştirilebilir ve "İlk seçim ekranını tekrar
göster" gerçek eylemi; mobil desen alt sekme çubuğu (44pt+ hedefler,
oranlı sekme genişlikleri, R-24 gereği boş sekme yok), mobil grid dikey
reflow (zaman üstte + tam genişlik metin, ≥44pt satır), Kaydet/Yeniden aç
ve şerit düğmeleri mobilde 44pt; masaüstü yerleşimi birebir korundu.
Şerit gövdeleri `video/ses/stil/form/durum/grid_icerik` yöntemlerine
çıkarılıp iki çerçevede tek kaynağa bağlandı. Cargo.toml'a android dışı
hedeflerde eframe `x11` kabiliyeti (winit host derlemesi düzeltildi).
Kanıt: host release RC=0, `cargo test` 1 passed (ayar yazım/okuma
habercisi), aarch64-linux-android release RC=0 (8.067.368 B); etkileşimli
GUI koşumu bu ortamda ekran olmadığından APK fazına kaldı. antislop
AFTER modu teslimatı `anti-slop/audit-001-2026-10-20.md` (commit
[`1a433efdd`](https://github.com/velnoxy/velnosub/commit/1a433efdd) +
[`e639946f8`](https://github.com/velnoxy/velnosub/commit/e639946f8) +
[`f485ba75d`](https://github.com/velnoxy/velnosub/commit/f485ba75d)).
**P4.7 Δ ✔ (denetim düzeltmeleri, kullanıcının "tümünü düzelt" onayı)** —
kontrast ölçüm kolu (`parlaklik`/`kontrast` WCAG 2.1) + tamir: egui
varsayılan soluk metin koyu zeminde **2.27:1 İHLAL** tespit edildi →
tamirle 7.41 (soluk), grid zaman gri `from_gray(170)` iki kol 7.41, hata
rengi 5.64, açık tema 6.49 (test `kontrast_tamiri_aa_uyumlu` kanıtı);
sekme kırpılması: uyarlanır metin boyu (9–15.5pt) + padding 4px + alt
genişlik 44pt; AÇIK TEMA: `Tema{Koyu,Acik}` + ayar dosyasında `"tema"`
alanı + Ayarlar'da 44pt seçim düğmeleri + açılış uygulaması
(`main.rs`); masaüstüne Ayarlar penceresi (AYNI `ayarlar_icerik`
gövdesiyle); [UI0..UI7] cihaz koşum şeridi planı APK fazının ilk işi
olarak faz4 künyesine yazıldı; [`DESIGN.md`](DESIGN.md) yön belgesi yazıldı (commit
[`abacb542a`](https://github.com/velnoxy/velnosub/commit/abacb542a) +
[`9d3476892`](https://github.com/velnoxy/velnosub/commit/9d3476892)).
**P4.6 dilim 2 ✔ — egui video paneline ASS önizleme** — C kol: render
kompoziti dizgi-alfaya çevrildi (zemin alfa 0, over kuralı; eski
çıkışta bölge siyah-opaktı), `[AB9]` glyph metriği etki koşuluna bağlı
olduğundan değişmedi: host+cihaz **BİREBİR 9937** (taze .so pkg'da koşuldu);
`velnosub_abi_close` artık render şartını da yumlatır (reload sızıntısı
kapandı). Rust kol: `Oniz` durumu altyazı kolunu izler, açılış tek
denemede (`VELNOSUB_FONT` + "Default"), ms = kare*1000/fps (VideoSection
artık fps tutar), ms değişmedikçe yeniden render yok; kare `ui.put`
ile video rect üstünde `etki > 0` iken işlenir, etkinin 0 olduğu
boş karelerde doku işlenmez; hata şeritte dürüst weak satır. Kanıt: üç meson kan
RC=0 (.so), host RC=0 + test 2 passed + aarch64 RC=0 (8.175.232 B) —
egui penceresi görsel kanıtı cihaz [UI8] adımında (APK fazı) (commit
[`e78cf9ca9`](https://github.com/velnoxy/velnosub/commit/e78cf9ca9) +
[`9cc02a7ed`](https://github.com/velnoxy/velnosub/commit/9cc02a7ed)).
Kalan: zaman nesnesi, text_extents/frame köprüsü, MoonScript/karaskel paketi, DependencyControl, APK bağlamı (ve [UI0..] koşum şeritleri).

Yapılacaklar:
1. **Lua motoru:** Aegisub Lua 5.1 / LuaJIT API'sine göre yazılmış scriptleri çalıştırır. Android arm64'te LuaJIT kullan. (iOS kalemi 2026-10-20'de iptal; JIT kısıtı notu artık müteferri.)
2. **Aegisub Lua API'sinin tamamı** korunmalı: `aegisub.*` (log, progress, dialog, decode_path, cancel, text_extents, frame_from_ms, ms_from_frame, project_properties, video_size, vm API, gettext, lua_api eklemeleri: selection/cursor kontrolü vb. — bu fork'ta `lua_api` branch'i)
3. **`aegisub.dialog.display` / `aegisub.dialog.open|save`:** Script diyaloglarını (label, edit, intedit, floatedit, textbox, dropdown, checkbox, color, coloralpha, alpha, slider vb.) Qt/QML'de dinamik form olarak yeniden uygula. Bu, wx'e en bağımlı kısımdır.
4. **MoonScript:** `automation/include/moonscript.lua` ile gömülü gelir; doğrudan çalışmalı
5. **Dahili scriptler:** `automation/autoload/` (ör. karaoke templater, kara-templater, cleantags, macros) birebir paketlenir
6. **DependencyControl uyumu:** `automation/include/l0/` ve ilgili yapı, kullanıcı scriptleri için yol çözümleme (`aegisub.decode_path` ile `?user`, `?data`, `?script`, `?temp`, `?audio`, `?video`, `?local`) mobil dizinlere eşlenmeli
7. **Kullanıcı script yönetimi:** SAF ile klasörden script ekleme, Automation Manager ekranı (yükle/yeniden yükle/hata logu)
8. **Harici bağımlılıklar:** Birçok script `require("ffi")` veya Windows DLL'i (`requireffi`, `ILL`, `Yutils`, `clipper`, `lfs`, `re` vb.) kullanır. Android için bunların .so karşılıklarını derle/paketle ve bir uyumluluk tablosu üret (çalışan / kısmi / çalışmayan).
9. **Çalıştırma güvenliği:** Scriptler kullanıcı yetkisinde çalışır; ağ erişimi (`io`, `os.execute`, `io.popen`) mobilde kısıtlanabilir → `os.execute`/`io.popen` Android'de işlev dışı bırakılmalı veya sandbox'lanmalı, bu belgelenmeli

### Faz 6 — iOS (**İPTAL, 2026-10-20 — kullanıcı kararı**)

Bu faz kapsamdan çıkarıldı; iOS'e özel bir geliştirme yapılmayacak. Eski
kalemler (Qt/Metal, JIT'siz Lua, UIDocumentPicker) zaten Faz 4 egui
revizyonuyla çoğunlukla geçersizdi. Android hedefi tek platform olarak kalır;
bölüm tam silinmedi — iOS yeniden gündeme gelirse eski metin git
geçmişinden geri okunur.

### Faz 7 — Paketleme ve test
- Android: AAB/APK, ABI split, imzalama
- CI: GitHub Actions ile NDK build + çekirdek testleri + emülatörde smoke test
- Regresyon: referans `.ass` dosyaları + referans scriptler (çıktı karşılaştırma)
- Performans bütçesi: 1080p video seek < 150 ms, 5000 satırlık dosya grid kaydırma 60 fps

## 5. Bilinen riskler

| Risk | Etki | Önlem |
|---|---|---|
| wx bağımlılığı çekirdeğe sızmış | Faz 1 uzar | Bağımlılık haritasıyla önceliklendir, soyutlama katmanı |
| Script diyaloglarının dinamik doğası | UI karmaşıklığı | Veri güdümlü form üreteci |
| FFI/native script kütüphaneleri | Bazı scriptler çalışmaz | Uyumluluk tablosu, popüler kütüphanelerin Android derlemeleri |
| Büyük ses dosyaları | Bellek/depolama | Disk önbellekli PCM |
| Doğru kare seek | Typesetting için kritik | İndeksleme + BestSource benzeri doğruluk modu |
| Lisans | Dağıtım | Kod GPL uyumlu BSD tarzı; fftw nedeniyle birleşik çıktı GPLv2 → kaynak açma yükümlülüğü |
| Mağaza politikaları | Kullanıcı scripti çalıştırma | Politikaları araştır, gerekirse script desteğini sınırla |

## 6. Teslim edilecekler

1. `aegisub-core` çekirdek kütüphanesi (wx'siz) + masaüstü regresyon testleri
2. Android NDK build sistemi ve CI
3. Mobil UI (Qt/QML) — telefon ve tablet düzenleri
4. Tam Lua/MoonScript automation desteği + script uyumluluk raporu
5. Dokümantasyon: mimari, derleme, script yazarları için mobil notları
6. Çalışan APK/AAB

## 7. Çalışma kuralları (AI ajan için)

- Her aşama bitmeden bir sonrakine geçme; her fazın sonunda çalışan, test edilmiş bir çıktı olsun
- Masaüstü derlemesini bozma
- Büyük değişikliklerden önce plan yaz, belirsiz kararları (Qt vs Compose, LuaJIT vs PUC) gerekçeleriyle belgeleyip sor
- Çıktılar: kod + `docs/mobile/` altında kısa not dosyaları
- Bilinmeyeni uydurma; emin olmadığın yerde depodaki koda bak
