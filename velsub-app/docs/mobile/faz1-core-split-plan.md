# Faz 1 — Çekirdek Ayrıştırma İş Planı (Velsub)

**Tarih:** 2026-10-18 (UI kararı sonrası güncellenmiş)
**Amaç:** `wx-dependency-map.md` § 6 çerçevesini uygulanabilir iş paketlerine çevirmek.
**Karar kaydı:** UI = **Qt 6** (kullanıcı onayı, 2026-10-17/18). Masaüstü test hattı için Ubuntu apt'tan native arm64 Qt6 kullanılacağı planlanıyor; Android hedefi ise Faz 4'te Qt kaynak derlemesi + `QT_HOST_PATH` yöntemiyle hedeflenecek. Faz 2'deki NDK gereksinimi bu karardan bağımsızdır — çekirdek `.so` her hâlükârda NDK ister.

## 0. Faz 1 hedefleri

1. `aegisub-core` meson hedefi oluşturulur: `libaegisub` + wx bağımlılık haritasındaki Sınıf C dosyalarının tamamı + temizlenmiş Sınıf B dosyaları.
2. Soyutlama yüzeyi `src/core/compat/` altına konumlanır (harita § 6.2 tablosu).
3. **UI-bağımsızlık ilkesi:** aşağıdaki tüm arayüzler Qt'den bağımsız tiplerle (std:: + agi::) tanımlanır. Qt katmanı bu arayüzleri kendi nesneleriyle besleyecek; masaüstü wx GUI de aynısını wx nesneleriyle beslemeye devam edecek.

## 1. Soyutlama arayüzleri — olgulara dayalı taslak

### 1.1 Etkileşim ABI'leri (karar kullanıcıda olduğunda çağrılan yüzeyler)

| Arayüz | Masaüstü karşılığı (kod tespit) | Kullanım yerleri |
|---|---|---|
| `agi::sys::SingleChoice` | `wxGetSingleChoiceIndex` | `charset_detect.cpp`, `mkv_wrap.cpp` |
| `agi::sys::Confirm` | `wxMessageBox` (evet/hayır/prompt modları) | `search_replace_engine.cpp` (yalnız 1 nokta) |
| `agi::sys::FileDialog` (open/save + flag'ler) | `wxFileDialog` | `utils.cpp`, `auto4_lua_progresssink.cpp` |
| `agi::sys::Clipboard` (get/set) | `wxClipboard`, `wxTextDataObject` | `utils.cpp`, `auto4_lua.cpp` |

Dönüş ve veri tipleri `std::string` / `std::optional` / `std::vector<std::string>`. UI yoksa (headless/core testleri) default davranışlı bir impl servis edilir; masaüstünde wx impl, mobilde Qt impl atravır.

### 1.2 Yaşam döngüsü / çalışma zamanı ABI'leri

| Arayüz | Masaüstü karşılığı | Kullanım yerleri | Not |
|---|---|---|---|
| event dispatch temeli | `wxDECLARE_EVENT` / `wxDEFINE_EVENT` / `wxEvent` | `value_event.h`, `async_video_provider.h` | Çekirdekte std::function tabanlı küçük dispatcher; masaüstü GUI bunu wx event handler'larına bağlayan ince köprüyle kullanır |
| `agi::ui::Timer` | `wxTimer` | audio/video controller | Çekirdek arayüzü chrono tabanlı; masaüstünde wxTimer sarmalayıcısı |
| Threads | `wxThread` | `subtitles_provider_libass.cpp` | std::thread |
| `agi::font::Measure` | `wxFont` + `wxMemoryDC` (Win32'de GDI) | `auto4_base.cpp` (`aegisub.text_extents`) | Faz 1 masaüstünde wx impl korunur; mobilde FreeType+HarfBuzz (libass ile aynı FT paylaşılabilir) |

### 1.3 Veri ABI'leri

- strings: çekirdekte `std::string`; mevcut `to_wx/from_wx` UI tarafında kalır (`compat.{h,cpp}`).
- geometri: `agi::Point/Size/Rect` libaegisub'a eklenir (provider/event API'leri bunları kullanır; wx'e dönüşüm UI'da).

## 2. Sıralı iş paketleri (Faz 1)

| # | Paket | İçerik | Kabul |
|---|---|---|---|
| P1.1 ✓ | meson hattı | ✔ **bitti (2026-10-18, c2aef9bed):** `src/core/meson.build` + `-DAGI_CORE`; preheader PCH kurulmadı — çekirdek TU'ları iyice azaltıldı (`core/math_util.h` köprüsü, include swap'leri), PCH ileride hız için opsiyonel | core static lib yeşil (260 KB); masaüstü build yeşil (aegisub 11.1 MB); meson test 345/351 (baseline ile aynı, 6 lagı_acs uid-0 artefaktı) |
| P1.2 | string/geo temizliği | Sınıf B'de wxString/wxPoint içeren dosyaların çekirdek hallerinin std::'ye taşınması | build + grep: core klasöründe `#include <wx/>` = 0 |
| P1.3 | etkileşim ABI'leri | § 1.1'deki 4 arayüz + headless default impl + ilgili 4 dosyanın bağlanması | masaüstünde dialog davranışı değişmeden çalışıyor |
| P1.4 | event/timer/thread | § 1.2 tablosundaki üç madde | build + smoke: async video provider bildirimi işliyor |
| P1.5 | font::Measure | masaüstünde wx impl korunarak arayüzle karşılanır | text_extents smoke testi (Lua) |
| P1.6 | CLI smoke testi | `aegisub-core` ile: ASS aç → satır düzenle → kaydet → referansla karşılaştır | `meson test` içinde yeni test olarak |

### P1.1 künyesi ve kararlar (c2aef9bed)
- **Yaklaşım sapması:** PCH (`agi_core_pre.h`) kurulmadı; bunun yerine çekirdek TU'larında `utils.h` → `core/math_util.h` include swap'i + `core/math_util.cpp` tanım dosyası kullanıldı (GUI'de tanım `utils.cpp`'de kalır, çakışma yok; masaüstü çekirdeğe bağlanınca tekilleşir).
- **`AssFile::Commit` sınırı:** `ass_file.h`'e `Commit(std::string const&)` overload'u **yalnız `-DAGI_CORE`'da görünürlü** olarak eklendi (masaüstünde `Commit("")` literalleri tek aday görür → upstream API yüzeyi değişmedi; `""` literal iki adaylı belirsizlik denemesi netleşti). Tanım `src/core/commit_compat.cpp`: wx'siz, **PushState'siz** (undo yığını UI'dır), açıklama UTF-8 tutulur.
- **Çeviri (i18n) kuralı:** çekirdek, `_()` çevirisi yapmaz; ham UTF-8 üretir ve çevriyi görüntüleyen UI yapar (masaüstü kolu `_("resolution resampling")` ile korunur).
- **`agi::format` inceliği:** masaüstünde `agi::format(std::string, …)` çağrısı `src/format.h` wxString overload'una örtük dönüşümle çözülür; core'da wx olmadığından `fmt.c_str()` ile `const Char*` yüklemesine bağlanır (aynı formatter, aynı çıktı).
- **`vector2d.h`:** `wxPoint` inline ctor + `wx/gdicmn.h` include `AGI_CORE` ile koşullu.
- **Ertelendi:** `spline.cpp` → P1.4 (`Spline(const VisualToolBase*)` koordinat çevirici bağı `CoordTranslator` ABI'ının konusu); `spline_curve.cpp` core'da.
- **GitHub:** repo `velnoxy/Velsub` = `arch1t3cht/Aegisub` fork'unun özdeş adı ile; `feature` dalı P1.1 commit'iyle itildi (dolaysı push `workflow` scope kabul etmedi; fork sayesinde geçmiş sunucuda hazır olduğundan yalnız yeni commit yollandı).

Her paketten sonra masaüstü hedefi (`aegisub`) yeniden derlenir; sürekli yeşil kalması zorunludur (plaz kuralı: "Masaüstü derlemesini bozma").

## 3. Faz 1 kapsamı dışında kalanlar

- UI-coupled Sınıf A dosyaları (32 dialog, grid'ler, edit box, visual tool'lar, GL çizim …) **dokunulmaz**.
- `command/*` UI etkileşim dosyaları Faz 4'e bırakılır (command palette tabanı orada işlenecek).
- `libaegisub` içinde davranış değişikliği yapılmaz (sadece soyut tipler eklenir).

## 4. Faz 2-3 ön notları

- **Faz 2 (Android cross-build):** NDK temini bu telefonda arm64-host sorununa sahiptir. Seçenekler: (a) Termux NDK repack (aarch64-host NDK), (b) NDK'yı x64 host üzerinde uzakta/CI'da kullanmak, (c) qemu-user'la x64 NDK'yı koşturmak (yavaş). Bu karar Faz 2 başında kullanıcıyla netleştirilir. Bağımlılık hedefleri: boost headers, icu, zlib, fftw (veya kissfft), hunspell, LuaJIT (vendor, aarch64), libass (freetype/harfbuzz/fribidi), ffmpeg+ffms2.
- **Faz 3 (medya):** Oboe (AudioProvider), FFmpeg tabanlı doğru kare seek, PCM disk önbellek, spektrum hesabı (mevcut `audio_renderer_spectrum` hesabı UI'de kalır, veri çekirdekte) — Faz 1'de sağlayıcı ABI'ları bu eylemlerin doğal ek noktasıdır.

## 5. Faz 1 riskleri

- `agi_pre.h` PCH'sini çekirdek TU'larından ayırmak ilk iş paketi (P1.1) sırasında derleme bağımlılık zinciri sorunları çıkarabilir; çözümün P1.1'e sıkıştırması gerekebilir.
- event ABI'sinin iki katman arasında (wx'li ması balkona impl + wx'siz core impl) kontrakt fecristasyonuna dikkat: aynı event tipleri için smoke test şart.
- Haiti Automation ABI'ları (§ 1.1) Faz 5'te Lua dialog'larıyla tekrar vinc; Faz 1'de geçtikleri yüzeyler neden velocity 됩니다 oldığında Faz 5'teше korunur.
