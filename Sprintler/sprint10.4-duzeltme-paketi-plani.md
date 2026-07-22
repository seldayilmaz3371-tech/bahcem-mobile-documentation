# Sprint 10.4 Düzeltme Paketi — Kesin Geliştirme Planı

**Tarih:** 2026-07-21 · **Kural:** Bu belge SADECE planlama amaçlıdır. Hiçbir kod yazılmadı, hiçbir dosya değiştirilmedi, hiçbir commit oluşturulmadı.

---

## 1. Sınıflandırma

| # | Bulgu | Sınıf | Gerekçe |
|---|---|---|---|
| 1 | `BulkOperationsScreen` → `BulkMaintenanceForm` arası `initialMaintenanceType` aktarımı eksik | B) Eksik Entegrasyon | Her iki taraf da kendi içinde doğru çalışıyor (menü doğru state üretiyor, form doğru bir dropdown sunuyor) — sadece aralarındaki bağlantı hiç kurulmamış |
| 2 | `MaintenanceRecordCard`, `startTime`/`endTime` göstermiyor | C) Eksik Özellik | Sprint 10.4'ün kapsamı hiç bu ekranı içermiyordu — kod hiç yazılmadı, "bağlanmamış" bir bağlantı değil, "hiç var olmayan" bir görüntüleme mantığı |
| 3 | `MaintenanceRecordForm`, `startTime`/`endTime`'ı hiç desteklemiyor | C) Eksik Özellik | Aynı gerekçe — Sprint 10.4'ün kendi teknik raporunda "kapsam dışı" olarak zaten açıkça yazılmıştı, bilinçli bir erteleme (ama görüntüleme kodu hiç yazılmadı) |
| 4 | "Biçme" kayıtları "Other" olarak görünüyor | C) Eksik Özellik (D değil) | Gerçek dosya incelemesiyle doğrulandı: Sprint 10.2'nin kararı SADECE yazma (kaydetme) tarafını kapsıyor — "kullanıcıya seçenek olarak gösteriliyor, arka planda `other` kaydediliyor." Görüntüleme (okuma/listeleme) tarafı için hiçbir açık karar alınmamış, hiç düşünülmemiş — bu yüzden D (bilinçli mimari karar, doğru çalışıyor) değil, C (bu açı hiç ele alınmamış) |

Not: Listede A) Gerçek BUG sınıfına giren hiçbir madde yok — hiçbir yerde "önceden doğru çalışan bir davranış bozulmuş" değil, hepsi ya eksik bağlantı ya da hiç yazılmamış kod.

---

## 2. Mimari Kararlar (Kullanıcının 3 Sorusu)

### Soru 1 — MaintenanceRecordCard Sulama saatlerini göstermeli mi?

Öneri: Evet, göstermeli. Gerekçe: Kullanıcı özellikle "Sulama Başlangıç/Bitiş Saati" özelliğini saha ihtiyacı olarak talep etti (Sprint 10.4'ün orijinal isteği) — bir veriyi kaydedip hiçbir yerde göstermemek, özelliğin kullanıcı değerini önemli ölçüde azaltır. Kart zaten `metaText` içinde durum+tarih gösteriyor (`statusLabel · formattedDate`); saat bilgisi, SADECE `startTime`/`endTime` doluysa (yani Sulama kayıtları için), bu metaText'e küçük bir ek olarak eklenebilir (ör. "Tamamlandı · 19 Tem · 06:15–08:05"). Diğer bakım türlerinde bu alanlar zaten `null` olacağı için hiçbir görsel değişiklik olmaz — tasarım gereği gizli kalmıyor, koşullu olarak (veri varsa) gösteriliyor.

### Soru 2 — MaintenanceRecordForm bu saatleri düzenleyebilmeli mi, yoksa sadece görüntülemeli mi?

Öneri: Düzenleyebilmeli (Toplu formla tutarlı). Gerekçe: Kullanıcı sahada bir kaydı düzeltme ihtiyacı duyabilir (ör. saati yanlış girdi, düzeltmek istiyor) — sadece görüntüleme, bu ihtiyacı karşılamaz ve kullanıcıyı "yanlış kaydı silip yeniden toplu oluşturma" gibi gereksiz bir dolambaca zorlar. Mimari olarak bu düşük risklidir: `MaintenanceRecordForm` zaten `DateField` kullanıyor (tarih alanları için), aynı desene `TimeField` (Sprint 10.4'te zaten oluşturulmuş, test edilmiş bir bileşen) eklemek, sadece Irrigation türü seçiliyken görünür olacak şekilde (Bulk formunun `isIrrigation` desenini tekrarlayarak) uygulanabilir.

### Soru 3 — "Biçme" arka planda other olarak saklanmaya devam etmeli mi? UI'da nasıl gösterilmeli?

Öneri: Evet, `other` olarak saklanmaya devam etmeli (Sprint 10.2'nin kararı korunmalı — migration riski hâlâ geçerli), ama UI'da "Biçme" olarak gösterilmeli.

Gerekçe: Veritabanı seviyesinde `other` saklamanın nedeni (SQLite `CHECK` kısıtı değişikliğinin tablo yeniden oluşturmayı gerektirmesi) hâlâ geçerli — bu kararı değiştirmek gerçek bir migration riski doğurur, kapsam dışı. Ama görüntüleme katmanında `other` ile `Biçme`'yi ayırt etmenin gerçek bir veri kaybı olmadan mümkün olduğu doğrulandı: `maintenance_records` tablosunda "Biçme" ile "gerçekten Diğer" arasında bugün hiçbir ayrım yok (ikisi de `other`) — bu, kullanıcının önceden kabul ettiği bir sınır (Sprint 10.2'nin kendi notu: "hiçbir migration gerekmedi").

Mimari not: Bu, `MaintenanceType.Other`'ın TÜM kullanımlarını etkileyecek bir metin değişikliği olacağından (Tekli formda da "Other" seçeneği var) — kapsamı dikkatli tutmak gerekiyor: sadece Toplu İşlemler üzerinden oluşturulan `other` kayıtları "Biçme" olarak etiketlenmeli mi, yoksa TÜM `other` kayıtları mı? Bu ayrımı yapacak bir veri (ör. "kaynak" sütunu) yok — bu yüzden en gerçekçi, en düşük riskli öneri: TÜM `other` etiketini "Biçme"ye değiştirmemek, bunun yerine SADECE Toplu İşlemler'in kendi akışında zaten kullanılan `bulkOperations.mowingLabel` anahtarının aynısını, `MaintenanceRecordCard`/`MaintenanceRecordForm`'da da `maintenanceType === "other"` durumunda kullanmak. Bu, Tekli formdaki "Other" (gerçekten "diğer bir şey" anlamında girilmiş) kayıtlarla "Biçme" (Toplu İşlemler'den gelen) kayıtları ayırt edemez — bu, dürüstçe belirtilmesi gereken kalıcı bir sınırdır, İSTEĞE BAĞLI madde olarak işaretlenecek.

---

## 3. Sprint 10.4 Düzeltme Paketi — Önceliklendirilmiş İş Listesi

| Sıra | İş | Zorunlu/İsteğe Bağlı | Neden Yapılmalı | Etkilenecek Dosyalar | Risk | Regresyon Riski | Yeni Test |
|---|---|---|---|---|---|---|---|
| 1 | `BulkMaintenanceForm`'a `initialMaintenanceType` prop'u eklemek, `BulkOperationsScreen`'in bunu iletmesi | ZORUNLU | Kullanıcının bildirdiği ana belirti — menüden seçilen tür formda yansımıyor | `BulkMaintenanceForm.tsx`, `BulkOperationsScreen.tsx` | Düşük — tek noktalı, izole değişiklik | Yok (önceki raporda doğrulandı) | Evet — 5 menü butonunun her birinin doğru türle açıldığını kanıtlayan testler |
| 2 | `MaintenanceRecordCard`'a Sulama saatlerini (varsa) gösterme | ZORUNLU | Kaydedilen veri hiçbir yerde görünmüyor — özelliğin kullanıcı değeri eksik kalıyor | `MaintenanceRecordCard.tsx` | Düşük — koşullu ekleme, `startTime`/`endTime` `null` ise görünüm değişmez | Yok — diğer bakım türlerinde bu alanlar zaten `null`, görsel fark olmaz | Evet — Sulama kaydında saat gösterildiği, diğer türlerde gösterilmediği |
| 3 | `MaintenanceRecordForm`'a Sulama saatlerini gösterme + düzenleme desteği | ZORUNLU | Kullanıcı, sahada girilen bir saati düzeltebilmeli — "sadece görüntüleme" kullanıcı ihtiyacını karşılamıyor | `MaintenanceRecordForm.tsx` | Orta — mevcut bir formun genişletilmesi, dikkatli entegrasyon gerektiriyor | Düşük — yeni alanlar opsiyonel, mevcut kayıtlar (saat alanı `null`) etkilenmez | Evet — saat düzenleme + kaydetme + `isIrrigation` koşullu görünürlük |
| 4 | "Biçme" kayıtlarının Liste/Detay'da "Biçme" olarak gösterilmesi | İSTEĞE BAĞLI | Kullanıcı deneyimi tutarlılığı — ama gerçek bir veri kaybı/hata değil, sadece bir etiket farkı | `MaintenanceRecordCard.tsx`, `MaintenanceRecordForm.tsx` | Düşük | Yok — sadece görüntüleme metni değişiyor, veri değişmiyor | Evet — `other` türünün "Biçme" olarak gösterildiği |

---

## 4. Neden "İsteğe Bağlı" Olan Sadece Madde 4?

Madde 1-3, kullanıcının doğrudan bildirdiği ve doğrulanan belirtilerin gerçek nedenleri — bunlar yapılmadan sorun "çözüldü" denemez. Madde 4 ise, kullanıcının doğrudan bildirmediği, doğrulama sürecinde ortaya çıkan ek bir gözlem — gerçek bir işlevsellik sorunu değil (kayıt doğru saklanıyor, doğru sayılıyor, doğru filtreleniyor), sadece bir görüntüleme etiketi tercihi. Kullanıcı onaylarsa dahil edilebilir, aynı geliştirme turunda (aynı dosyaları etkilediği için modüle tekrar dönmeyi gerektirmez) yapılabilir.

---

## Sonuç Tablosu (İstenen Format)

| İş | Sınıfı | Zorunlu mu? | Risk | Test |
|---|---|---|---|---|
| `initialMaintenanceType` prop aktarımı | B) Eksik Entegrasyon | ZORUNLU | Düşük | 5 senaryo (her menü türü için doğru form açılışı) |
| `MaintenanceRecordCard` saat gösterimi | C) Eksik Özellik | ZORUNLU | Düşük | Sulama'da gösterim, diğerlerinde gizli kalma |
| `MaintenanceRecordForm` saat düzenleme | C) Eksik Özellik | ZORUNLU | Orta | Görüntüleme + düzenleme + kaydetme + koşullu görünürlük |
| "Biçme" etiketinin Liste/Detay'da gösterilmesi | C) Eksik Özellik | İSTEĞE BAĞLI | Düşük | `other` → "Biçme" etiket eşlemesi |

Bu 4 iş, tek bir geliştirme turunda, `maintenance` + `bulkOperations` modüllerini kapsayan tek bir commit serisiyle tamamlanabilir — aynı modüle ayrıca geri dönmeyi gerektirmez.
