#.not — Aegisub deposu dahili dizine taşındı

## Bu dizinde ne var?

- `aegisub-mobile-port.md` — Proje planı (okunan ana belge)
- `velsub-app/` — Velsub'a özgü kod, doküman ve notların geleceği dizin
  - `velsub-app/docs/mobile/` — Faz çıktısı not dosyaları (wx bağımlılık haritası vb.)

## Aegisub deposu nerede?

`aegisub-upstream` klonu bu dizinden **taşındı** → `/root/velsub/aegisub-upstream`

## Neden taşındı?

Planın 9. satırındaki uyarı doğrulandı (17 Ekim 2026 testi):

| Sorun | Test | Sonuç |
|---|---|---|
| Yürütme izni | `chmod +x test.sh` → `./test.sh` | ❌ Permission denied (FUSE noexec) |
| Symlink | `ln -s . testln` | ❌ Permission denied |
| Dosya yazma | doküman yazma | ✅ çalışıyor |

Derleme için kaynak ağaçta çalıştırılabilir dosyalar ve bazen symlink gerekir;
Android paylaşılan depolaması (FUSE) bunu vermiyor. Bu yüzden:

- **Kaynak ağaç + derleme dizini:** dahili uygulama alanı (Termux/proot dahili `/root/velsub/`), tam izinli
- **Doküman/çıktılar:** bu dizinde (paylaşılan, Android uygulamalarından erişilebilir)

Senkron ilkesi: kod değişiklikleri dahili depoda yapılır; Velsub'a özgü dokümantasyon
bu dizine yazılır. Fazlar ilerledikçe kullanıcı isteyirse dahili dizinden bu dizine
kod kopyaları (örn. yamalar, çıktılar) aktarılabilir.

Bu taşıma `aegisub-mobile-port.md` §7 kuralına (3. madde) uygun yapıldı.
