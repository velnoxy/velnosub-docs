# velnosub Faz 5 — Automation (LuaJIT) planı ve künyeleri

Master plan Faz 5: LuaJIT aarch64 (JIT/FFI) ile automation diziminin mobilde
çalışması; `aegisub.*` yüzeyi, DependencyControl, karaoke templater.
Bu dosya dilim planını ve künye/koşum kayıtlarını tutar.

## Dilim planı

- **P5.0** — LuaJIT 2.1 zemini: `tools/cross/build-luajit.sh` (host + arm64 + x86_64
  statik; source `subprojects/luajit` wrap v2.1), `velnosub-automation` duman CLI
  (aegisub.*-stub + gömülü script) ✔ host koşumu; arm64 cross sıradaki adım.
- **P5.1** — çekirdek bağlaması: automation → `libvelnosub-core.so` C yolu
  (P4.1 ABI şeridinin tekrar kullandığı disiplin; Lua `AssFile` görünümü) ✔
  (FFI → `libvelnosub-abi.so` hattıyla; aşağıda künye).
- **P5.2** — `aegisub.*` gerçek yüzeyi (makro altyazı düzenleme çağrıları) ✔
  ilk dilim: file verisi + dialog pompası; aşağıda künye (devam: zaman
  nesnesi, filtre kayıtları).
- **P5.4** — dialog form omurgası: `aegisub.dialog` kontrol sınıfları + dinamik
  egui eşlemesi ✔ ABI/widget kısmı tamam (künye: faz4-ui-plan P4.5 III) —
  Lua pompası (`dialog.display`) P5.2 ile bağlandı ✔.
- **P5.3** — script yükleme düzeni + DependencyControl uyum zeminleri ✔
  ilk dilim: manager taraması + gerçek upstream script koşumu + flyweight
  kökten çözüm; aşağıda künye (kalan: DependencyControl, moonscript).

## P5.0 — LuaJIT zemini ✔ (2026-10-19, host koşumu)

| Kayıt | Değer |
|---|---|
| Kod commit'leri | [`46cbecd32`](https://github.com/velnoxy/velnosub/commit/46cbecd32) (build-luajit.sh + automation TU + meson hedefi), [`ebe19435d`](https://github.com/velnoxy/velnosub/commit/ebe19435d) (kopyala tüm repo: dynasm kızıl referansı), [`c27d781e5`](https://github.com/velnoxy/velnosub/commit/c27d781e5) (install kök Makefile'dan), [`0074297d6`](https://github.com/velnoxy/velnosub/commit/0074297d6) (meson doğrudan çatısı: pkg-config devre dışı), [`f8bbc7259`](https://github.com/velnoxy/velnosub/commit/f8bbc7259) (TU luajit.h katılımı) |
| Script | `tools/cross/build-luajit.sh host|arm64|x86_64`; `JOBS` env destekli; statik (`BUILDMODE=static`), NDK r29 aracı zinciri cross'ta |
| Host kurulum | `/root/velsub/ndk-deps/luajit-host` (include/luajit-2.1 + lib/libluajit-5.1.a; `LJHOST=0`) |
| Meson hedefi | `velnosub-automation`: native → `luajit-host`, cross → `ndk-deps/<abi>` üstü yolu; `lua.hpp` üstbilgi yoksa hedef bilinçli atlanır |
| Host koşumu | `NINJA=0` (`[9/11] Linking target tools/cross/velnosub-automation`); smoke `AUTOSMOKE=0`: `[AU0] LuaJIT kuruldu: LuaJIT 2.1.1791988261`, `[AU1] aegisub tablo kayıtlı`, `[AUL] P5.0 zemin dumanı başladı`, `[AUL] arg: 0, toplam: 1499999500000` (1..1e6 döngüsü Σ(i·3−2) değeri birebir), `[AU2] script koştu`, `[AU3] tamam` |
| Derinleşen bulgular | (1) kopyalama `src/.` ile yapılmıştı: `buildvm ../dynasm` kızıl referansı kırıldı → tüm-repo kopyası. (2) `make install` kuralı kök Makefile'da (`src/` değil). (3) meson 1.1+ dış `PKG_CONFIG_PATH` yutmuyor; `host-luajit.ini` native-file `pkg_config_libdir` da bu konakta yutuldu → pkg-config devre dışı, doğrudan üstbilgi+kütüphane çatısı (kayıt `0074297d6`). (4) `LUAJIT_VERSION` `<luajit.h>`'ta |
| arm64 cross | LuaJIT `ndk-deps/arm64-v8a` altına kuruldu (include/luajit-2.1 + lib + pc); `LJA64=0`, `RCONF_A64=0`; `build-android-arm64/tools/cross/velnosub-automation` 763.168 B (21:56) |
| Cihaz koşumu (2026-10-20) | ✔. Termux proot'ta DOĞRUDAN koştu (adb'siz): `[AU0..AU3]` tamam, `Σ=1499999500000` birebir. Not: çıktı `> dosya` yönlendirmeli koş — `| head/tail` boru altında ikili takılıyor (proot boru tuzağı) |
| x86_64 cross | LuaJIT `ndk-deps/x86_64` altına kuruldu: `libluajit-5.1.a` 1.069.102 B (NDK readelf: arşiv üyeleri REL, `Machine: Advanced Micro Devices X86-64`), `luajit.pc` 344 B; `LJX64_RC=0` (23:21) |
| x86_64 automation | `build-android-x86_64/tools/cross/velnosub-automation` 801.264 B (23:31) |
| Meson önbellek tuzağı | Aynı bağımsız değişken kümesiyle `--reconfigure` çalışırken eski `Has header "lua.hpp" : NO` sonucu coredata önbelleğinden döndü; el yordamıyla clang derleme + bağlama sorularının ikisi yeşil olmasına rağmen hedef eklenmedi. Çözüm: resmi `meson configure <dir> --clearcache` + reconfigure (masa7 setup'u LuaJIT kurulmadan önce koştuğu için) |
| Sırada | cihaz dumanı (arm64: cli/ass-render/automation/abi-smoke; paket reçetesi faz2-device-test) + P5.1 başlangıcı |

## P5.1 + P5.2 (ilk dilim) — aegisub.* gerçek yüzeyi ✔ (2026-10-20, host + cihaz)

| Kayıt | Değer |
|---|---|
| Kod commit | [`0328d339d`](https://github.com/velnoxy/velnosub/commit/0328d339d) (feature dalı) |
| Mimari | **FFI → `libvelnosub-abi.so`**: LuaJIT `ffi.load` + `ffi.cdef` ile UI'nin kullandığı AYNI ABI kolu; ayrı bir çekirdek bağlama yazılmadı (P4.1 disiplininin tekrarı yerine tek yüzey) — `velnosub-aegisub.lua` shim'i, `VELNOSUB_AEGISHIM` env ile yüklenir; env yoksa P5.0 stub davranışı birebir korunur |
| Dosya verisi | `velnosub_abi_init/open/event_count/line_text/line_times/edit_line/save/close/last_error`; satır tablosu `{text=, start_time=, end_time=}` (zamanlar ASS biçimli dizge — gerçek zaman nesnesi P5.2 devam); `subs[i]` okuma/yazma (edit_line→COMMIT_DIAG_TEXT) |
| subs nesnesi bulgusu | LuaJIT'te (5.1-Z!) tablo `__len` güvenilmez (`#subs` 0 dönüyordu) — subs **newproxy-userdata** olarak kuruldu: `__index/__newindex/__len` üçü de işler, `#subs` birebir |
| dialog pompası | `aegisub.dialog.display(dlg, buttons)` → P4.5 III dialog ABI'sı: tanım tablosu→DSL satırları (`ctl\|sınıf\|ad\|başlik\|değer[\|ek]`; label 4-hücreli, checkbox→0/1, intedit/floatedit ek=min,max, dropdown ek=öğeler); cevap uygulama `dialog_set` (tip dönüşümlü), düğme metni dönüşü, değerler name-anahtarlı tabloya int/bool/dizge çözümlü |
| Pompa CLI | runner'a `aegisub.__pump` çağrısı eklendi: `--ass D --macro M --btn N --val ad=deger --sel i --save Y`; şerit `[AUS0..AUS5]` + `[AUSR]` kayıtlar + `[AUP0]` |
| Demo makrolar | `velnosub-demo-macro.lua`: **"Ana harf"** (seçim {2,3} metinlerini upper — içerik teyitli) + **"Form dene"** (dialog pump: `metin=duman-formu, sayi=7, buyuk=true` → satır 1 `DUMAN-FORMU @7`) |
| Koşumler | host RC=0 (`f4-o1.ass` save, `[AUS4] kalıcılık: yeniden açılış satır 1: DUMAN-FORMU @7`), **cihaz proot RC=0 paket-içi birebir** (`/tmp/f4-dev.ass`); "Ana harf" host RC=0; P5.0 şeridi değişmedi ([AU0..AU3]) |
| ABI dokunuşu | dialog DSL'de label satırı için boş `ad` izni (aegisub label name'siz kurılır) — üç kol + smoke yeniden yeşil |
| Paket | `share/lua/velnosub-aegisub.lua` + `share/lua/velnosub-demo-macro.lua` eklendi (refresh script bloğu; 86.879.944 B, 33 dosya) |
| Sırada | P5.2 devam (zaman nesnesi, subtitle.insert_line/delete_line, filtre kayıtları), P5.3 script düzeni (dir scan, DependencyControl) |

## P5.3 (ilk dilim) — script manager + gerçek upstream koşumu + flyweight kök çözümü ✔ (2026-10-20, host + cihaz)

| Kayıt | Değer |
|---|---|
| Kod commit | [`c362063a5`](https://github.com/velnoxy/velnosub/commit/c362063a5) (feature dalı) |
| Manager taraması | runner'a `--dir D` / `--script F` / `--include F` içinden yazar noktası: dizindeki `*.lua` eşgörünümlü sıralı yükleme, `[AUZ0]` tarama + `[AUZ1]` yüklendi + `[AUZ2]` include izleri; `--macro/--btn/--val/--sel/--save` pump AYNI |
| Shim ekleri | aegisub 2.x `include("x.lua")` globali (`VELNOSUB_AEGIINC` ;  ayırımlı dizin, BOM temizli okuma), `aegisub.decode_path` (`?user/?data/?script/?temp`), `set_undo_point` (kayıt notlu stub), `register_filter`, `progress.task/set`, satır tablosuna `class` + `comment` (ABI `velnosub_abi_line_meta`: 39→**40** sembol; olaylar hep "dialogue", Comment olayı bayrağı) |
| Yeni ABI | `velnosub_abi_line_meta(h, idx, buf, n)` → `"dialogue|0/1"` dizgesi (upstream scriptlerin class/comment alanına erişimi için) |
| Gerçek upstream koşumu | `automation/autoload/strip-tags.lua` **birebir kopya** (BAĞImlılıksız): `--macro "Strip tags" --sel 4 --sel 6` → ABI idx3/idx5 içeriğinde `{By the way…}` anahtar dizisi ve `{\an1}` etiketi birebir kalktı, seçimsiz satırlar dokunulmadı; `cleantags-autoload.lua` **birebir kopya** + `include cleantags.lua`: manager dizin taraması 2 script yükledi, Clean Tags 40 satırı temizledi, kayıt, ikinci tutamaç teyidi (`[AUS6]` idx2/idx4 içerikleri birebir) |
| ✔ kök neden çözümü | **boost::flyweight çapraz-DSO yığın bozulması**: `Text/Style/Actor/Effect/ExtradataIds`/attachment alanları `boost::flyweight` (refcounted varsayılan). Meyers-singleton fabrika INLINE statik olduğu için DSO başına ayrı örneklenir: `libvelnosub-core.so` yükleyici F1'e, `libvelnosub-abi.so`/`velnosub-cli` yazma kolü F2/F3'e — yükleme F1'de, yazma F2'de → `assign` yabancı F1 düğümünü F2 tablosunda erase etmeye çalışan `free(): invalid size` (mini2 minibindirme Lua'sız C++ tekrarı: 0,1,2→3 sırasında deterministik; 3-first sıralamada esrarengiz geçer; 40-düzenleme Clean Tags akışı SEGFAULT; teardown'da "double free or corruption") |
| ✔ Çözüm mimarisi | TÜM flyweight alan yazmaları `src/core/velnosub_core_edit.cpp` (core.so içinde derlenir — F1 kolu) ile tepsi kolda: `velnosub_core_edit_line_text(edit_line)` + `velnosub_core_edit_first_text(edit_first + cli)`; abi/cli yalnız vekil çağırır. OKUMALAR fabrika dokunmuyor → güvenli. abi.so/cli'den artık hiçbir Text mutasyonu geçmiyor |
| ✔ regresyon şeridi | abi-smoke'a `[AB7]` eklendi: aynı tutamaçta 3 tur × 4 satır atama + her turda geri-okuma teyidi — bu alçak-frekans hatayı kalıcı yakalar; nm defteri 40 sembol |
| Koşumlar | host: mini 40-commit+save+reopen RC=0 (evvelki SIGSEGV), `MALLOC_CHECK_=3` ile 0,1,2,3 sırası yeşil (evvelki abort), Clean Tags manager RC=0 (evvelki SEGFAULT); cihaz: abi-smoke `[AB7]` ✔ + Clean Tags manager RC=0 + `Form dene` (DUMAN-FORMU @7) + `Ana harf` RC=0 — birebir |
| Paket | 86.865.035 B, 37 dosya; `share/lua/autos/` (upstream birebir kopyalar) + `share/lua/inc/cleantags.lua` + `share/lua/lua-noop.lua` eklendi (refresh script bloğu) |
| Sırada | P5.2 devam (zaman nesnesi, insert/delete ABI), DependencyControl zemini, moonscript/karaskel paketi |

## P5.2 (devam) — satır ekleme/silme ABI'sı + auto4 yöntemleri ✔ (2026-10-20, host + cihaz)

| Kayıt | Değer |
|---|---|
| Kod commit | [`8d4eb608a`](https://github.com/velnoxy/velnosub/commit/8d4eb608a) (feature dalı) |
| Çekirdek vekiller | `velnosub_core_insert_line(AssFile&, int idx, metin, bas_zaman, son_zaman, stil)` — idx 0-taban, düğüm konumun ÖNÜNE (idx == adet → sona); zamanlar ASS dizgesi (`agi::Time(std::string)` ile çözüm), boşsa varsayılan (0/5000); Text/Style flyweight yazımı **yine core.so kolu** (P5.3 kök neden kuralı). `velnosub_core_delete_line` — intrusive erase + delete, `COMMIT_DIAG_ADDREM` |
| Yeni ABI | `velnosub_abi_line_insert(h, idx, metin, bas, son, stil)` + `velnosub_abi_line_delete(h, idx)` — nm defteri 40→**42** sembol; aralık denetimleri (insert 0..adet, delete 0..adet-1) |
| Shim | subtitle nesnesine auto4 yöntemleri: `subs:insert_line(konum, satır)` (konum 1-taban, satır tablosu `.text` zorunlu; `.start_time`/`.end_time` ASS dizge geçişi; `.style` isteğe), `subs:delete_line(konum)`, `subs.n_lines` özelliği; adet sayacı ekle/sil ile canlı güncellenir |
| Demo makro | `velnosub-demo-macro.lua` üçüncü makro **Ekle sil dene**: seçili satırın ÖNÜNE `"önce: "` önekli kopya (`text+times` kopyalanır, style mimlenir) ekler, asıl satırı siler (net önekleme; adet değişmez). `--sel 4` → `[AUL] ekle sil: 4. satır öncesi/sonrası` + `[AUS6]` kalıcılık: `"önce: Now going to test..."` + `0:00:07.00,0:00:09.00` **birebir** |
| Regresyon | abi-smoke `[AB8]` ekleme/silme şeridi: 41→42 satır (metin `ab8 ekleme domatesi`, `0:00:01.00 / 0:00:02.00` birebir) → geri 41; host + cihaz RC=0; `Ekle sil dene` host+cihaz birebir; Clean Tags manager regresyonu yerinde (RC=0) |
| Paket | 86.880.742 B, 37 dosya; ABI 42 sembol |
| Kalan | `aegisub.time` nesnesi (aritmetik ve dizge dönüşümleri), `aegisub.text_extents`, `aegisub.frame_from_ms`/`ms_from_frame` (video bilgi köprüsü), DependencyControl zemini, moonscript/karaskel paketi |

## P5.2 (kalanı) — ms zamanları + video köprüsü + text_extents ✔ (2026-10-20, host + cihaz, üç dilim)

Not: upstream `aegisub` tablosunda **`aegisub.time` nesnesi YOK** (auto4_lua
`register` listesinde 18 ad: text_extents/frame_from_ms/ms_from_frame/
video_size/keyframes/decode_path/cancel/lua_automation_version/file_name/
gettext/project_properties/...). Zaman nesnesi kaydı upstream'e aykırıydı;
P5.2 kalanı yerine GERÇEK boşluk kapatıldı.

### Dilim A — ms zamanları [`b66794856`](https://github.com/velnoxy/velnosub/commit/b66794856)

| Kayıt | Değer |
|---|---|
| C çekirdeği | `velnosub_core_set_line_times(AssFile&, idx, ms_bas, ms_son)` — `d.Start = agi::Time(int ms)` (`Time(int ms=0)` üretici kanıt), `COMMIT_DIAG_TIME`; flyweight disiplindeki TU'da (mutasyon tek kol) |
| Yeni ABI | `velnosub_abi_line_times_set(h, idx, long long, long long)` — nm defteri 42→**46** sembol |
| Shim | `zaman_ms` ("0:00:01.23" → 1230; negatif "-0:00:X") + `ms_zaman` (ms → "h:mm:ss.cc", yansımalı); `satir_oku` **start_time/end_time artık MS TAMSAYISI** (P5.1 dizgesel sınır kalktı — upstream auto4 satır alanları ms'dir); `insert_line` ms sayısını ASS dizgesine çevirir; `__newindex` alan yazımı: `text`→edit_line, `start_time`/`end_time`→line_times_set (ms-sayı veya ASS dizgesi kabulü, verilmeyen alan mevcut kalır) |
| Şerit | abi-smoke `[AB10]`: set→`0:00:01.23 / 0:00:05.68` dizge teyiti + aralık-dışı red + eski değerlere geri dönüş döngüsü (sonraki şeritler orijinal durumu görür) |
| Kanıt | 3-lane RC=0; `[AB10]` host+cihaz BİREBİR (52 şerit satırı eş, diff yalnız kayıt-yolu arg'ı + libass sürüm rozeti — her iki kötüleşme ortam farkıdır); pompa `Ekle sil dene` RC=0: satır 10 kopyası `0:02:40.00→0:03:00.00` birebir (ms→ASS çevrimi kayıpsız) |

### Dilim B — video köprüsü + zarif yüzey [`634b588a9`](https://github.com/velnoxy/velnosub/commit/634b588a9) + [`1b9c4e41a`](https://github.com/velnoxy/velnosub/commit/1b9c4e41a)

| Kayıt | Değer |
|---|---|
| Shim | `frame_from_ms`/`ms_from_frame`/`video_size` — **video yüklü değilken nil** (upstream auto4_lua davranışı birebir: `Timecodes().IsLoaded()` değilse pushnil); arg tip kancaları (tamsayı değilse hata); `keyframes` → boş küme; `lua_automation_version = 4`; `file_name` → pump açık dosyadan temel ad (yoksa nil — upstream `subsController->Filename()` boşken nil); `gettext` → özdeş (en); `project_properties` → upstream 14 alan (automation_scripts/export_filters/export_encoding/style_storage/video_zoom/ar_value/scroll_position/active_row/ar_mode/video_position + audio/video/timecodes/keyframes dosya yolları) nötr mobil değerleriyle |
| Log etiketi | `[AUSB]` (pump'un "yeniden açılış" satırıyla çakışmayan ayrık etiket) |
| Duman | `tools/cross/lua-autos/velnosub-b-dumani.lua` (testmedia/lua-autos kopyası sürüm altına alındı); `B duman` makrosu: nil'ler + boş küme + sürüm 4 + dosya adı + gettext + 14 alan liste baskısı |
| Kanıt | pump RC=0: `[AUSB] frame_from_ms: video yüklü değil → nil` vb. 6 AUSB satırı + 14 alan yoklanır; kalıcılık `[AUS4]` zinciri zarar görmedi |

### Dilim C — `aegisub.text_extents` [`8ef817bff`](https://github.com/velnoxy/velnosub/commit/8ef817bff)

| Kayıt | Değer |
|---|---|
| Kol | ABI `velnosub_abi_text_extents(yol_font, boyut, kalin, italik, harf_bos, olcek_x, olcek_y, metin, out4[4])`; **upstream auto4_base.cpp CalculateTextExtents non-WIN32 dalının yapısıyla**: spacing≠0 → harf harf (kerning yok) + `fontsize/ink-yükseklik` normallemesi, spacing=0 → tek geçiş kerning'li (`FT_HAS_KERNING`); girişte `out4` sıfır küme (hata bitse bile — upstream davranışı); `scalex/scaley/100` telafisi; FreeType 26.6 sabit nokta → /64 piksel |
| Sıfır-ink | boşluk gibi ink'siz harflerin ölçek zemini `ascender−descender` (wx font metrik yüksekliği karşılığı; yükseklik ölçümüne girmez) |
| Sentetik | italik: 0.2 eğme matrisi (libass katsayısı — ilerletmeyi değiştirmez); kalin: ~%4 ilerletme artışı; fontname eşleşmesi cihaz font-servis fazına kalmakta — ölçüm tek paket fontu'ndan (pkg'ta DejaVuSans.ttf) |
| meson | root'a `velnosub_ft2_dep = dependency('freetype2', fallback: ['freetype2','ft2_dep'])` — subproject hattında libass ile AYNI freetype derlemesine biner; host'ta pkg-config | 
| nm defteri | 46→**47** sembol |
| Shim | `text_extents(stil_tablosu, metin)`: `class=='style'` kanca; uyumsuz sınıf → hata **"Not a style entry"** (upstream dizgesi); araç font yolu `VELNOSUB_FONT` (varsayılan `./DejaVuSans.ttf`) |
| Şerit | abi-smoke `[AB11]`: 28px *"HAHA VelnoSub"* → g=220.000 y=21.000 (DejaVu hint'li tam sayı ilerletmeler; FT probe ile teyit), 56px → oran **2.0000 birebir**, kalin 234.560 (%4 × 13 harf), `scale_x %150` → 330.000 (1.5x birebir), spacing 4px artışı, boş metin → rc 0 + sıfır küme, kötü font yolu + 600px boyut redleri |
| Pompa | demo `Ölç dene` makrosu: **host+cihaz BİREBİR** (`g=260.00 y=21.00 d=0.00 l=0.00`; dialog sınıf red `Not a style entry`); kalıcılık zinciri zarar görmedi |
| Kanıt | 3-lane RC=0; `[AB11]` host↔cihaz BİREBİR (52 satır eş, diff yalnız kayıt-yolu arg'ı + libass rozeti) |
| Paket | 86.980.103 B, 37 dosya; velnosub-ui 5.651.784 B, abi-smoke 395.544 B |
| Kalan | DependencyControl zemini, moonscript/karaskel paketi (karaskel artık text_extents + satır tabloları_READY), video köprüsünün FPS'li genişlemesi (apkg fazıyla) |

## Karaskel zinciri (P5 ikinci kuşak) — LuaJIT 5.2 zemini + upstream subs semantiği ✔ (2026-10-20/21, host + cihaz)

### Dilim L — LuaJIT 5.2 uyum zemini [`46e2dd0e7`](https://github.com/velnoxy/velnosub/commit/46e2dd0e7)

| Kayıt | Değer |
|---|---|
| Build | `build-luajit.sh` host+arm64+x86_64 koluna `XCFLAGS="-DLUAJIT_ENABLE_LUA52COMPAT"` — upstream meson wrap'iyle (subprojects/packagefiles/luajit 15-16) aynı payda; üç lane yeniden derlendi (~15 dk/kol, seri) |
| Kök neden | LuaJIT `ffh_pairs(L,MM_ipairs)` yalnız `LJ_52 \|\| tviscdata` şartıyla `__ipairs` metamethoduna bakar — 5.2 uyumsuz userdata'da ipairs tabloda bile HATA atar; `__ipairs`'i shim'in kendisi de tanımlıyor (upstream auto4_lua_assfile.cpp 770) — ikisi birden şart |
| Teşhis | shim `[AUST2] LuaJIT 5.2 uyum zemini: <bool>` her pump koşumunda; `table.pack` mevcutluğu 5.2 modu kanıtı |
| Kanıt | pump host+cihaz RC=0; dilim-C öncesi kanıt zinciri zarar görmedi (Ölç/Form/Ana harf/Ekle sil birebir) |

### Dilim V — sanal satır uzayı [`d7c627114`](https://github.com/velnoxy/velnosub/commit/d7c627114)

| Kayıt | Değer |
|---|---|
| Uzay | subs dizinleri upstream LuaAssFile lines = [Script Info] + Styles + Events; `#subs`/`subs.n` = TÜM satırlar, `subs.n_lines` olay sayısı (dilim-A önü korunur) |
| Info tarama | `bilgi_satirlari()` AssParser::ParseScriptInfoLine kuralları bire bir: boş satır kayım değiştirmez; `;` + `Collisions:` satırı yutulur; `:` yoksa yutulur (küt boşluklu anahtar, değer trim-left); 19 işçi öznitelik anahtarı + `Automation Settings ` öneki Info'ya DÖNÜŞMEZ; `[script info]` küçük-harf karşılaştırması; kurucu başlangıç durumu info |
| Stil alanları | ABI 13-kanal `stil_dizgesi` → upstream AssEntryToLua style dalı eşlemesi: fontname (font DEĞİL), colorN `GetAssStyleFormatted() + "&"`, margin_t/b tek slot = Margin[2] (margin_v KARASKEL'İN kendisi kurar), relative_to=2, class="style" |
| Dolaşım | sayısal `__index` (1..n, aralık kırması upstream CheckBounds dizgesi) + `__ipairs` + `__len=to` + `__newindex` olay bölgesi (text/zaman yazımı, ABI idx'de ofset kayması) |
| insert/delete | olay bölgesi sabit: bilgi/stil bölgesindeki konum dürüst reddi; upstream `insert(before)` 1..lines+1 semantik kayımlandırılmış |
| Seçim | pump `--sel` upstream `selected_rows` uyumuyla TAM-uzay dizine çevrilir (offset=info+styles; auto4_lua.cpp 911 kanıt); demo makrolarda seçim baskıları `aktif=ofset+sel` gösterir |
| Duman | `Dolaş dene`: 51 satır = 8 info + 2 stil + 41 olay (ass-format-tests); ilk 12 satır class'lı baskı; `subs[0]`/`subs[n+1]` kırması upstream dizgesiyle kanıtlı |
| Kanıt | host+cihaz RC=0 birebir; Ölç g=260.00 y=21.00 birebir (regresyon zinciri sağlam) |

### Dilim 1 — `subs.styles` okuma proxy [`b00e46304`](https://github.com/velnoxy/velnosub/commit/b00e46304)

| Kayıt | Değer |
|---|---|
| Yüzey | `styles.n` / `styles[i]` (1..N) / `styles[ad]` (tanım sırası, ilk eşleşme; yoksa nil) / `__len` / `__ipairs`; her dönüş taze ABI (`velnosub_abi_style_count/style_of`), karaskel `styles[l.name]` hattına tam uyum |
| Duman | `Stil dene`: n=2; Default Arial 20 align 2 marj 10×4 relative_to=2 renk1 `&H00FFFFFF&`; Alt Times 40 align 8; yok-stil nil; ölçüm kardeşliği y=19.00 |
| Kanıt | host+cihaz RC=0 BİREBİR |

### Dilim 2 — `subs.script_resolution()` [`024f79b4f`](https://github.com/velnoxy/velnosub/commit/024f79b4f)

| Kayıt | Değer |
|---|---|
| Kurallar | upstream AssFile::GetResolution: PlayResX/Y Info'dan `iequals`-ilk-eşleşme (şim key:lower) + atoi baştan-tam-sayı; Gabest: (0,0)→384×288; x=0→y==1024?1280:floor(y·4/3); y=0→x==1280?1024:floor(x·3/4) |
| Sunum | `testmedia/karaskel-dene.ass` (PlayRes yok + `Video File:` işçi öznitelik Info dışı + Kara stil/kara olayları dilim-3 payı) + `testmedia/res-yarim.ass` (yalnız PlayResY:480 → 4:3 kuralı) |
| Kanıt | 3 koşum RC=0 BİREBİR iki kutup: 640×480 / 384×288 / 640×480 |

### Dilim 3 — `aegisub.parse_karaoke_data` [`d7a58624b`](https://github.com/velnoxy/velnosub/commit/d7a58624b)

| Kayıt | Değer |
|---|---|
| Durum makinesi | upstream AssKaraoke::SetLine(dia,false,false) saflua: k-tag = ad `\k` önek (büyük/küçük duyarsız); `\K`→`\kf`; `(sure>0 \|\| metin≠"")` push; push sonrası upstream YALNIZ metin/ovr temizler — baş zamanı ESKİ sure ile birikir (`start += duration`), yeni sure = param·10 ms; son sibek koşulsuz (taksiz metin dahi 0-sure sibek); ovr chunk: `{`+tag dizgesi…+`}`, k-tag chunk'ı kapar, `\p1/\p0` çizim devri düz metni offset'li ovr'a toplar |
| Alanlar | auto4 kara[0] boş sibek sabiti + 1..n: duration/start_time/end_time (satıra bağıl ms), tag, text (ovr aralıklı dikt dize — GetText(false) karşılığı; map sıralı diken), text_stripped (saf metin) |
| Duman | `Kara dene`: karaskel-dene.ass 5 olay el-izlenmiş beklenende birebir (ka/ra/oke zamanları; ilk-tag-öncesi 0-sure sibek sürprizi; `{!not}` yorum ovr; karaoke-siz satır) |
| Kanıt | host+cihaz RC=0 BİREBİR — 11 sibek satırı, "5 olay birebir" |

### Dilim 4 — karaskel vendor [`ca72a257a`](https://github.com/velnoxy/velnosub/commit/ca72a257a)

| Kayıt | Değer |
|---|---|
| Vendor | `karaskel.lua`/`karaskel-auto4.lua`/`utils.lua`/`utils-auto4.lua`/`unicode.lua` upstream automation/include orijinalleri BOM'lu birebir → pkg `share/lua/inc/` (refreshDevice scripti "karaskel vendor kümesi"); host pump `VELNOSUB_AEGIINC=automation/include` (orijinal orijinaller) |
| require kancaları | `package.preload["aegisub.unicode"]` (upstream unicode.moon karşılıkları: charwidth b<128→1/<224→2/<240→3/yok→4 + chars/len/codepoint; to_*_case `__unicode_impl` doğal kolu mobilde yok — dürüst red) + `package.preload["aegisub.util"]` (upstream util.moon gövdeleri saf-Lua birebir: copy, deep_copy seen haritası, ass_color/ass_alpha/ass_style_color, extract_color 3-deneme profili, HSV_to_RGB Hi dalları, HSL_to_RGB, trim/headtail/words, clamp/interpolate/_color/_alpha) |
| Satır alanları | ABI `line_meta` dizgesi `dialogue|comment` → +layer/style/actor/effect/margin_t(b=c) (upstream LuaDialogueToLua 164-182; flyweight `.get()`, 47 sembol sabit — kanal büyümesi); shim `satir_oku` buna göre alan kümesini doldurur |
| debug.out | seviye-1 argüman + `msg:format(...)` düzeltmesi (önceki hal `out(4, "…%f", x)` çağrısında "4" basardı) |
| Duman | `Karaskel dene`: include("karaskel.lua") → collect_head(subs,false) → preproc_line_text/size/pos; res=384×288 (Gabest), stil.n=2, korrektör 1.000 (video nil), `margin_v` değerini gülüştürür; kırpık "karaoke" kara.n=3; konum sol=143.5 orta=192 sağ=240.5 x=192 h=center v=bottom (el-hesap birebir); etkin marj 10×4; sibek g 0/29/25/43 (toplam = satır g 97) |
| Kanıt | host(upstream include'larla)+cihaz(pkg include'larla) RC=0 BİREBİR; cihaz Ölç dene g=260.00 y=21.00 regresyon değerleri zarar görmez; masaüstü gtest **345/351** + velnosub-core-smoke Ok — 6 `lagi_acs.*Denied` testi proot-kök ortamının chmod bypass'ı — kök hesap izin-denetim testlerini geçemez (ortam kısıtı; bu zincirle ilgisiz) |
| Paket | device-pkg.tar.gz **87.002.454 B / 44 dosya** (bin+lib+share/lua/inc karaskel kümesi); core.so 117 MB unstripped bırakıldı (künye açıklaması: hata ayıklama yararı) |
| Kalan | DependencyControl zemini; moonscript apkg yolu; video köprüsü FPS'li genişleme (apkg fazıyla); satır `raw` alanı + extradata tabloları (upstream extradata — mobil zincir ileride) |

## APK alpha hazırlığı (PAP fazı 1) ✔ [`b6ca32cb2`](https://github.com/velnoxy/velnosub/commit/b6ca32cb2) — 2026-10-21

| Kayıt | Değer |
|---|---|
| Yer | `tools/cross/apk-alpha/` — AndroidManifest.xml (com.velnoxy.velnosub 0.1.0-alpha, minSdk 24/targetSdk 28, extractNativeLibs=true) + MainActivity.java (assets→filesDir kopyalayıcı + pump üç düğme: Paket kopyala / Dolaş dene / Kara dene; nativeLibraryDir'deki `libvelnosub_automation.so` — velnosub-automation ELF — VELNOSUB_* + LD_LIBRARY_PATH kancalarıyla koşar) + strings.xml (etiket) + build-apk.sh |
| Derleme hattı | Debian arm64 `aapt 0.2` (yerli ikilisi) + `javac -encoding UTF-8` (android.jar 34) + `d8 --min-api 24` (bt jar) + `zip` (lib deflate) + Debian `zipalign 4` + `apksigner` (bt jar; Java olduğundan bt sarmalayıcıyla koşar) + keytool debug keystore |
| Ortam notu | Google build-tools r34 ikilileri (aapt2/zipalign) x86_64 — aarch64 hatta koşmaz (ilk deneme SIGILL); Debian'ın arm64 ikilileri sorunu çözer; qemu-x86_64 prootta askıda kalır (yaklaşım düşürüldü) |
| Kanıt | `velnosub-0.1.0-alpha.apk` **12.777.457 B**, `apksigner verify` OK, `aapt dump badging` (location body): package/sdks/label tam, launchable-activity MainActivity; lib/arm64-v8a 14 native + assets gövde + classes.dex 8.4 KB |
| İçdenetim | APK gömülü native seti device-pkg ile aynı kanıt zincirinden şaşmaz; APK düğmelerinin sardığı pump koşumları iki kutupta zaten kanıtlı |
| Yol | HANDOFF.md el-kitabı repo'da; GitHub release yayın adımı kullanıcıya ait (contributor tarafı kullanıcıda); APK zemini apkg fazı 2 ile genişleyecek |

## Faz5 kapani — 2026-10-21 gece durumu

- Dilim zinciri tamam: L ✔ V ✔ 1 ✔ 2 ✔ 3 ✔ 4 ✔ — karaskel hattı iki kutupta pump
  koşumlarıyla canli; makro duman kümesi Dolaş/Stil/Çöz/Kara/Ölç/Ana harf/Form/Ekle sil
- Desktop: gtest **345/351 + velnosub-core-smoke Ok** (6 lagi_acs Denied — proot-kök
  ortam kısıtı; zincirle ilgisiz)
- nm: `velnosub_abi_*` **47 sembol** sabit (dilim 4 kanal büyümesi ABI fonksiyon
  eklemeden çözüldü)
- Paket: device-pkg.tar.gz 87.002.454 B / 44 dosya (karaskel vendor kümesi dahil)
- APK alpha: imzali velnosub-0.1.0-alpha.apk **12.777.457 B** (yayın forum kullanıcıda; el-kitabı HANDOFF.md)
