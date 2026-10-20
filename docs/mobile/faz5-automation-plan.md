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
| ✓ Çözüm mimarisi | TÜM flyweight alan yazmaları `src/core/velnosub_core_edit.cpp` (core.so içinde derlenir — F1 kolu) ile tepsi kolda: `velnosub_core_edit_line_text(edit_line)` + `velnosub_core_edit_first_text(edit_first + cli)`; abi/cli yalnız vekil çağırır. OKUMALAR fabrika dokunmuyor → güvenli. abi.so/cli'den artık hiçbir Text mutasyonu geçmiyor |
| ✔ regresyon şeridi | abi-smoke'a `[AB7]` eklendi: aynı tutamaçta 3 tur × 4 satır atama + her turda geri-okuma teyidi — bu alçak-frekans hatayı kalıcı yakalar; nm defteri 40 sembol |
| Koşumlar | host: mini 40-commit+save+reopen RC=0 (evvelki SIGSEGV), `MALLOC_CHECK_=3` ile 0,1,2,3 sırası yeşil (evvelki abort), Clean Tags manager RC=0 (evvelki SEGFAULT); cihaz: abi-smoke `[AB7]` ✓ + Clean Tags manager RC=0 + `Form dene` (DUMAN-FORMU @7) + `Ana harf` RC=0 — birebir |
| Paket | 86.865.035 B, 37 dosya; `share/lua/autos/` (upstream birebir kopyalar) + `share/lua/inc/cleantags.lua` + `share/lua/lua-noop.lua` eklendi (refresh script bloğu) |
| Sırada | P5.2 devam (zaman nesnesi, insert/delete ABI), DependencyControl zemini, moonscript/karaskel paketi |
