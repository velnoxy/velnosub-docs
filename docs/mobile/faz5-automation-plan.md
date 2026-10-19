# velnosub Faz 5 — Automation (LuaJIT) planı ve künyeleri

Master plan Faz 5: LuaJIT aarch64 (JIT/FFI) ile automation diziminin mobilde
çalışması; `aegisub.*` yüzeyi, DependencyControl, karaoke templater.
Bu dosya dilim planını ve künye/koşum kayıtlarını tutar.

## Dilim planı

- **P5.0** — LuaJIT 2.1 zemini: `tools/cross/build-luajit.sh` (host + arm64 + x86_64
  statik; source `subprojects/luajit` wrap v2.1), `velnosub-automation` duman CLI
  (aegisub.*-stub + gömülü script) ✔ host koşumu; arm64 cross sıradaki adım.
- **P5.1** — çekirdek bağlaması: automation → `libvelnosub-core.so` C yolu
  (P4.1 ABI şeridinin tekrar kullandığı disiplin; Lua `AssFile` görünümü).
- **P5.2** — `aegisub.*` gerçek yüzeyi (makro altyazı düzenleme çağrıları).
- **P5.3** — script yükleme düzeni + DependencyControl uyum zeminleri.

## P5.0 — LuaJIT zemini ✔ (2026-10-19, host koşumu)

| Kayıt | Değer |
|---|---|
| Kod commit'leri | [`46cbecd32`](https://github.com/velnoxy/velnosub/commit/46cbecd32) (build-luajit.sh + automation TU + meson hedefi), [`ebe19435d`](https://github.com/velnoxy/velnosub/commit/ebe19435d) (kopyala tüm repo: dynasm kızıl referansı), [`c27d781e5`](https://github.com/velnoxy/velnosub/commit/c27d781e5) (install kök Makefile'dan), [`0074297d6`](https://github.com/velnoxy/velnosub/commit/0074297d6) (meson doğrudan çatısı: pkg-config devre dışı), [`f8bbc7259`](https://github.com/velnoxy/velnosub/commit/f8bbc7259) (TU luajit.h katılımı) |
| Script | `tools/cross/build-luajit.sh host|arm64|x86_64`; `JOBS` env destekli; statik (`BUILDMODE=static`), NDK r29 aracı zinciri cross'ta |
| Host kurulum | `/root/velsub/ndk-deps/luajit-host` (include/luajit-2.1 + lib/libluajit-5.1.a; `LJHOST=0`) |
| Meson hedefi | `velnosub-automation`: native → `luajit-host`, cross → `ndk-deps/<abi>` üstü yolu; `lua.hpp` üstbilgi yoksa hedef bilinçli atlanır |
| Host koşumu | `NINJA=0` (`[9/11] Linking target tools/cross/velnosub-automation`); smoke `AUTOSMOKE=0`: `[AU0] LuaJIT kuruldu: LuaJIT 2.1.1791988261`, `[AU1] aegisub tablo kayıtlı`, `[AUL] P5.0 zemin dumanı başladı`, `[AUL] arg: 0, toplam: 1499999500000` (1..1e6 döngüsü Σ(i·3−2) değeri birebir), `[AU2] script koştu`, `[AU3] tamam` |
| Derinleşen bulgular | (1) kopyalama `src/.` ile yapılmıştı: `buildvm ../dynasm` kızıl referansı kırıldı → tüm-repo kopyası. (2) `make install` kuralı kök Makefile'da (`src/` değil). (3) meson 1.1+ dış `PKG_CONFIG_PATH` yutmuyor; `host-luajit.ini` native-file `pkg_config_libdir` da bu konakta yutuldu → pkg-config devre dışı, doğrudan üstbilgi+kütüphane çatısı (kayıt `0074297d6`). (4) `LUAJIT_VERSION` `<luajit.h>`'ta |
| Sırada | arm64 cross kurulum (ndk-deps/arm64-v8a içine pc dahil) + reconfigure → cihaz dumanı; P5.1 başlangıcı |
