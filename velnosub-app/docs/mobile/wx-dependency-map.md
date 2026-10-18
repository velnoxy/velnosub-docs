# Faz 0 — src/ wx Bağımlılık Haritası (velnosub)

**Proje:** velnosub — Aegisub mobil port planı
**Anlık görüntü:** `arch1t3cht/Aegisub`, `feature` dalı, commit `9bfd500` ("Fix placeholder comboboxes on wxGTK")
**Tarih:** 2026-10-17
**Yöntem:** statik tarama — `grep` bazlı wx include/token envanteri + hedefli kod okuması. Üretici: `/root/velsub/analiz/wxmap.py` (yeniden çalıştırılabilir; sonuçlar iki ürünle çapraz kontrol edildi).

---

## 1. Özet Genel Tablo

Tarama kapsamı: `src/` altındaki tüm `.cpp / .h / .mm` (`bitmaps` & `res` dışarıda) = **344 dosya**.

| Grup | Dosya sayısı | Not |
|---|---:|---|
| Doğrudan `#include <wx/…>` yapan dosyalar | 160 | Baskın kısıt = widget kullanımı |
| wx'li (PCH) ama kendi kodunda sadece az sayıda token'lı | 58 | `agi_pre.h` ~70 wx başlığını tüm src TU'larına ekliyor |
| **Tamamen wx'siz** dosyalar | 126 | Çoğunluğu çekirdek model & medya sağlayıcıları |

`src/` üst dizin: 176 `.cpp` + 135 `.h` → `command/` 15 `.cpp` · 32 `dialog_*.cpp`
En ağır wx kullanıcıları (token): `dialog_colorpicker`(222) · `preferences`(162) · `dialog_style_manager`(162) · `dialog_kara_timing_copy`(127) · `audio_display`(117) · `subs_edit_box`(114)

### Katman-temiz referans noktaları

- **`libaegisub/` tam wx'siz** — doğrulandı (tek isabetler: `common/charset_conv.cpp` yalnızca yorum; `include/libaegisub/format.h` sadece forward-decl). Plan §2 varsayımı doğru.
- **`vendor/luabins/` wx'siz.**
- **`tests/` wx'siz** (29 test dosyası libaegisub'u test ediyor; gtest/gmock). Faz 1 regresyon kanalı hazır.

---

## 2. Sınıflandırma

### Sınıf C — Çekirdek model, wx tamamen yok ("aegisub-core" taşınan adaylar)

- **ASS modeli:** `ass_attachment`, `ass_dialogue`, `ass_entry`, `ass_export_filter`, `ass_karaoke`, `ass_override`, `ass_parser`, `ass_style_storage`  *(ass_style: 2 token wxString — bkz. Sınıf B)*
- **Format I/O:** `subtitle_format_{ass,encore,microdvd,mkv,srt,ssa,transtation,txt}` *(ttxt hariç — wxXML)*
- **Metin/altyazı yardımcıları:** `string_codec`, `text_file_reader`, `text_file_writer`, `thesaurus`, `spellchecker`, `spellchecker_hunspell`
- **Geometri/renk/analiz:** `colorspace`, `fft`, `vector2d/3d`* *(vector2d.h'de 1 wxPoint konversiyonu → Sınıf B)*, `spline`, `spline_curve`, `resolution_resampler` *(yalnız kullanılmayan intl include)*, `fold_controller`, `initial_line_state`, `selection_controller`, `context`
- **Medya sağlayıcılar:** `libaegisub/audio/` (convert, dummy, hd, lock, pcm, ram — zaten çekirdekte) + `src/` tarafındaki `audio_provider_{factory,ffmpegsource,bestsource,avs,vs,dummy,yuv4mpeg}`, `audio_player` soyut tabanı, `video_provider_{cache,dummy,ffmpegsource,bestsource,avs,vs,yuv4mpeg}`, `bestsource_common`, `async_video_provider` (event başlık satırı hariç) — hepsi wx'siz, mobil medya katmanının (Faz 3) temeli hazır
- **Automation çekirdeği:** `auto4_lua_assfile.cpp` (yalnız 1 wxString token'ı), `auto4_lua_factory.h` (tamamen temiz)

### Sınıf B — Çekirdek mantığı olan ama **hafif wx dokunuşu** içeren dosyalar (Faz 1 mikro-düzeltmeleri)

| Dosya | wx içeriği | Temizleme stratejisi |
|---|---|---|
| `search_replace_engine.cpp` | wxMessageBox (1) | Onay sorusunu callback'e bağla: GUI'de dialog, headless'ta otomatik onay |
| `charset_detect.cpp`, `mkv_wrap.cpp` | wxGetSingleChoiceIndex | `ChoiceSelector` callback arayüzü (GUI'de dialog, mobilde native, headless'ta default) |
| `subtitle_format_ttxt.cpp` | wxXmlDocument / wxXmlNode | TTXT XML parse'ı bağımsız XML kütüphanesine taşı (ör. pugixml); mobil önceliği düşük format |
| `subtitles_provider_libass.cpp` | wxThread | std::thread |
| `font_file_lister.cpp` | wxString, wxUniChar | agi::fs / std::string |
| `ass_style.cpp` | wxString (2) | isim/font alanları std::string; `agi::Color` zaten libaegisub'da |
| `format.h` (i18n formatter) | wxGetTranslation | format arayüzü gövdede kalır, gettext backend korunur |
| `vector2d.h` | wxPoint (1) | wxPoint dönüşümü UI tarafına taşınır |
| `value_event.h`, `async_video_provider.h` | wxDECLARE_EVENT sınıfı | event soyutlama sınırları Faz 1'de burada çizilir |
| `hotkey.cpp` | intl, msgdlg | `agi::hotkey` zaten libaegisub'da + testli; `src/hotkey.cpp` menü/entegrasyon katmanına ayrılır |
| `utils.{h,cpp}` (32 token) | wxClipboard, wxFileSelector, wxExecute… | UI hizmetleri provider arayüzüne bağlanır (bkz. § 6.2) |

### Sınıf A — UI bağımlı, aegisub-core'a taşınmayacak (Faz 4'te yeniden yazma)

- Dialogs (32): colorpicker, preferences, style manager/editor, kara_timing_copy, shift/timing/translation/attachments/… · `dialog_progress` — progress sink UI
- Grid/EditBox: `base_grid`, `grid_column`, `subs_edit_box`, `subs_edit_ctrl`, `timeedit_ctrl`
- Audio/Video görüntüleyiciler: `audio_display`, `audio_box`, `audio_karaoke`, `audio_renderer*`, `video_display`, `video_slider`, `video_box`, `frame_main`
- Visual typesetting araçları: `visual_tool_*` · `visual_feature` · `image_position_picker`
- Çizim primitifleri: `pen`, `gl_wrap`, `gl_text`, `subs_preview` (libass önizleme) — wxDC/wxGLCanvas bağlı
- Komut UI yüzü: command/{app,audio,automation,edit,grid,help,menu,tool,video,recent,subtitle,time,timecode}.cpp
- Menü/toolbar: `menu`, `toolbar`, `tooltip_manager`
- yardımcı UI: `validators`, `colour_button`, `toggle_bitmap`, `placeholder_ctrl` (wx generic control şablonları — Faz 4'te UI tarafında yeniden uygulanır)

### PCH etkisi

`src/include/agi_pre.h` her src TU'suna ~70 wx başlığı ekliyor → bazı "temiz" dosyalar aslında PCH üzerinden wx görüyor. Faz 1'de çekirdek için **wx'siz bir preheader** (ör. `agi_core_pre.h`) ayrılması gerekiyor; `lagi_pre.h` (libaegisub) zaten wx'siz ve test edilmiş.

---

## 3. wx başlık frekansları (en sık kullanılanlar, include eden dosya sayısı)

```
 45 wx/sizer.h      35 wx/dialog.h     28 wx/stattext.h    28 wx/msgdlg.h
 27 wx/textctrl.h   23 wx/button.h     21 wx/checkbox.h    20 wx/intl.h
 19 wx/combobox.h   15 wx/string.h     12 wx/spinctrl.h    11 wx/settings.h
 11 wx/event.h      10 wx/timer.h      10 wx/colour.h      10 wx/choicdlg.h
  9 wx/radiobox.h    9 wx/panel.h       9 wx/bitmap.h       8 wx/window.h
  7 wx/valgen.h      7 wx/toolbar.h     7 wx/menu.h         7 wx/listbox.h
  7 wx/dcmemory.h · … *(devamı — toplam ~121 farklı başlık)*
```

Bu sıralama Faz 1 soyutlamasının öncelik sırasını da gösteriyor: widget olmayan wx cephesi (`wx/timer`, `wx/event`, `wx/intl`, `wxClipboard`, `wxThread`) soyutlamanın ilk hedefi; widget sınıfları (sizer/dialog/textctrl…) ise UI yeniden yazımına (Faz 4) ait.

## 4. wx TİP frekansı (token toplamı; soyutlama listesi kaynağı)

```
 408 wxString        254 wxCommandEvent    245 wxWindow         211 wxSizerFlags
 168 wxButton        164 wxSizer           160 wxDefaultPosition 134 wxBoxSizer
 125 wxEVT_BUTTON    124 wxTextCtrl        123 wxSize           113 wxVERTICAL
  79 wxArrayString    76 wxColour           73 wxStaticText      69 wxCheckBox
  68 wxBitmap         66 wxMessageBox       62 wxMouseEvent      62 wxComboBox
  60 wxDialog         49 wxPoint            48 wxDataViewItem    42 wxMenu
  40 wxPen            38 wxSpinCtrl         35 wxRect            32 wxDC
  26 wxSystemSettings 23 wxFont             21 wxStyledTextCtrl  20 wxVariant
```

## 5. Automation/auto4 — Faz 5'in kritik haritası

| Dosya | wx inc | wx rolü | Faz 1/5 · strateji |
|---|---:|---|---|
| `auto4_base.cpp` | 3 | `aegisub.text_extents` ölçümü: Win GDI / **`wxMemoryDC + wxFont`** (diğer platformlar) | **FontMeasureProvider ABI**: mobilde FreeType+HarfBuzz ölçümü (libass zaten freetype kullanıyor), masaüstünde wx/GDI korunur |
| `auto4_lua.cpp` | 3 | clipboard (wxTextDataObject), wxGetTranslation, wxLog* | `Clipboard`+`I18n`+`Log` arayüzleri |
| `auto4_lua_dialog.cpp` | 9 | **Tüm script dialog builder'ı** (button/checkbox/combobox/spinctrl/stattext/valgen) | Deklaratif şemaya ayır: Lua tablosu → `DialogSchema` verisi; UI katmanı şemayı render eder. Faz 1'de şema/veri çekirdeğe ayrıştırılır, Faz 4/5'te UI render devreye girer |
| `auto4_lua_progresssink.cpp` | 1 | wxFileDialog (open/save flags) | FileDialog arayüzü |
| `auto4_lua.h / auto4_lua_factory.h` | 1 / 0 | hafif | factory wx'siz — çekirdeğe taşınır |
| `auto4_lua_assfile.cpp` | 0 | temiz | çekirdek |

Automation'ın diğer kritik altyapısı zaten wx'siz: `automation/` (kayıtlı dahili scriptler ve `include/` Lua kütüphaneleri, DependencyControl dahil) — bu klasör olduğu gibi paketlenir; `po/` gettext yolu; hotkey registry libaegisub'da (testli).

---

## 6. Faz 1 somut planı (haritadan çıkan)

**Hedef:** masaüstünde çalışan wx-free bir `aegisub-core` hedefi; masaüstü uygulamasının bu çekirdek üzerinden derlenmeye devam etmesi (regresyon yok).

### 6.1 Dosya akışı

- (a) Yeni meson hedefi eklenir: `src/core/meson.build` → statik kütüphane `aegisub-core` = `libaegisub` + §2'deki Sınıf C dosyaları ve Sınıf B'nin temizlenmiş halleri
- (b) Mevcut masaüstü hedefi (`aegisub`) kaynak listesi bu dosyalardan boşaltılır ve `link_with: aegisub-core` ile bağlanır — davranış değişmez, derleme düzeni değişir
- (c) Hiçbir dosya dizin değiştirmez (git geçmişi ve incelemeyi bozmamak için; ileride istenirse gruplandırma ile dosya taşıma da yapılabilir)

### 6.2 Soyutlama katmanı (`src/core/compat/`; arayüz dosyaları üst düzey)

| Soyutlama (hedef ABI) | wx karşılığı | Kullanım yerleri | Uygulama önerisi |
|---|---|---|---|
| `agi::str` ↔ `std::string` | `to_wx/from_wx` (compat.h) | 50+ dosya | Mevcut dönüşümler UI tarafında kalır; çekirdekte std::string |
| `agi::Rect/Point/Size` | wxRect / wxPoint / wxSize | provider + event sınırları | libaegisub'a küçük geometri tipleri eklenir |
| `agi::sys::Clipboard` | wxClipboard, wxTextDataObject | utils, auto4_lua | masaüstünde wx impl; mobilde platform servisi |
| `agi::sys::FileDialog` | wxFileDialog (+ aile) | utils, auto4_lua_progresssink, style manager… | provider arayüzü |
| `agi::sys::SingleChoice` | wxGetSingleChoiceIndex | charset_detect, mkv_wrap | callback: GUI dialog / mobil UI / headless default |
| agi event sistemi | wxDECLARE_EVENT / wxDEFINE_EVENT / wxEvent | value_event.h, async_video_provider.h | çekirdekte bağımsız event payı; UI köprüsü masaüstünde wx event'e çevirir |
| `agi::font::Measure` | wxFont + wxMemoryDC (ve Win32 GDI) | auto4_base.cpp | masaüstünde mevcut impl korunur; mobilde FreeType+HarfBuzz |
| Thread | wxThread | subtitles_provider_libass | std::thread |
| Timer | wxTimer | audio/video controller ve paneller | soyut timer (impl: wx masaüstü, std::chrono tabanlı mobil) |
| Log | wxLog* | auto4_base, auto4_lua, misc | libaegisub log altyapısına (zaten mevcut) |
| i18n `_("…")` | wxLocale / wxGetTranslation | 20+ dosya | gettext arka ucu (`po/`) korunur — Qt/Android uyumlu |
| Renk | wxColour | colour_button, bazı dialoglar | agi::Color zaten çekirdekte; wxColour sadece UI katmanında |

### 6.3 Kabul kriterleri

- `meson compile aegisub-core` ve `meson test` masaüstünde yeşil
- `meson compile aegisub` (masaüstü GUI) davranış farkı olmadan derlenir/çalışır
- `src/core/` içinde hiçbir `.cpp/.h` dosyasında `#include <wx/…>` yok (grep tabanlı CI kontrolü eklenir)
- Sınıf B dosyalarının temizleme süreci § 2 tablosundaki plan doğrultusunda yürütülür; her küçük değişiklik masaüstü GUI derlemesine açıklanmış fark olarak kaydedilir

### 6.4 Faz 1'de kesinlikle yapılmayacaklar

- UI/dialog yeniden yazımı (Faz 4)
- Komutların UI'den ayrıştırılması (Faz 4 — command palette planlanırken)
- libaegisub'da davranış değişikliği (regresyon testleri mevcut; sadece gerekiyorsa soyut tipler eklenir)

## 7. Build durumu — Faz 0 masaüstü derleme denemesi

**Ortam:** Android cihaz üzerinde Termux + PRoot Ubuntu 26.04 (arm64, 8 çekirdek, ~3 GB boş RAM).
**Konfigürasyon başarılı** (`meson setup build --buildtype=release -Dbestsource=disabled -Dvapoursynth=disabled -Davisynth=disabled`):
- sistem kütüphaneleri: wx 3.2.9 · libass 0.17.4 · icu-uc 78.2 · zlib 1.3.1 · hunspell 1.7.2 · fontconfig 2.17.1 · fftw3f 3.3.10 · alsa · ffms2 5.0.0 · GL · PulseAudio (+ALSA)
- LuaJIT alt-projesi aarch64'te derlendi: **JIT + FFI aktif** (Faz 5 için kritik)
- gettext/msgfmt kuruldu; tests alt dizini gtest wrap'i indirip test hedeflerini kurar

**Derleme SONUCU:** `ninja -j4` tam derleme **başarılı** → `build/aegisub` (11.1 MB, arm64 ELF) + `libluajit.a` + `libaegisub.a`.

İz: ilk deneme 373/401'de durdu — `tools/version.sh` `--depth 50` kısa klonda eski SVN-baz commit aralığını çözemediği için (`git rev-list --count <eski-hash>..HEAD` "Invalid revision range") `git_version.h` boş makroyla yazılıyordu ve `version.cpp` derlenemiyordu. Klona tam geçmiş alındı (`git fetch --unshallow` → 8731 commit) ve derleme tamamlandı. Gelecekte fazlar için geçmiş zaten zorunluydu.

**Test sonucu:** `meson test` → **345 test geçti, 6 test başarısız**. Başarısızların tamamı `lagi_acs` sınıfının izin senaryoları (`CheckFile{Read,Write}ExReadDenied`, `CheckFileWriteExWriteDenied`, `CheckDir{Read,Write}Ex{Read,Write}Denied`); nedenleri projemizin kodu değil, ortamdır: testler `chmod 000 data/file_access_denied` bekler ama proot'ta süreç **uid 0 (root)** olduğundan çekirdek DAC izin bitlerini bypass eder ve `ReadDenied`/`WriteDenied` istisnası fırlamaz (bkz. `tests/tests/access.cpp`, `tests/setup.sh`). Normal bir kullanıcı oturumlu masaüstünde bu 6 test de geçer.

## 8. Ortam uyarıları (plan §7.3 gereği)

- **Android paylaşımlı depolamada (FUSE) klon sorunları:** FUSE tüm dosyaları 660 moduna sabitliyor ve exec/symlink'i engelliyor; `meson` "not executable" hatası da bu yüzden çıktı — git'in 755 olarak izlediği 15 dosya (`tools/version.sh` vb.) chmod ile onarıldı. Kaynak ağaç bu yüzden dahili depoya taşındı (bkz. `aegisub-upstream-YERINDEN-TASINDI.md`).
- **Derleme sdcard'da yapılmamalı** (noexec + symlink engeli); kaynak ağaç dahili `/root/velsub/aegisub-upstream` (Termux kendi alanı) — aynı sebepten.

```

---

### Değişiklik geçmişi
- 2026-10-17: Faz 0 haritası; sınıflandırma + Faz 1 çekirdek önerisi + ortam notları.
```
