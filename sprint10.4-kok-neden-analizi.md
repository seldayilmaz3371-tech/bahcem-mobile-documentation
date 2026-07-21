# Sprint 10.4/10.5 — "Kod Var Ama UI'da Görünmüyor" Kök Neden Analizi

**Tarih:** 2026-07-21 · **Kural:** Bu belge SADECE mevcut kod tabanının incelenmesiyle hazırlandı. Hiçbir kod yazılmadı, hiçbir dosya değiştirilmedi, hiçbir commit oluşturulmadı.

---

## 🔴 KÖK NEDEN — Kesin Kanıtla Bulundu

**`BulkOperationsScreen.tsx`'te, menüden hangi bakım türüne tıklandığı bilgisi (`initialType`), `BulkMaintenanceForm`'a HİÇ İLETİLMİYOR.**

### Kanıt (Gerçek Kod, Satır Satır)

`BulkOperationsScreen.tsx`'te, kullanıcı menüde bir türe tıkladığında:
```
onClick={() => setView({ mode: "maintenance", initialType: item.type })}
```
`initialType` gerçekten state'e kaydediliyor. Ama SADECE birkaç satır sonrasında, `BulkMaintenanceForm`'un render edildiği yerde:
```
if (view.mode === "maintenance") {
  return (
    <BulkMaintenanceForm
      parcelId={parcelId}
      trees={trees}
      onBack={() => setView({ mode: "menu" })}
      initialSelectedTreeIds={carriedTreeIds ?? undefined}
      onApplyAnotherOperation={handleApplyAnotherOperation}
    />
  );
}
```
**`initialType` hiçbir yerde `BulkMaintenanceForm`'a geçirilmiyor.** TypeScript bunu hata olarak yakalamıyor çünkü `BulkMaintenanceForm`'un kendi `Props` arayüzünde böyle bir alan zaten hiç tanımlı değil:
```
interface BulkMaintenanceFormProps {
  parcelId: string;
  trees: Tree[];
  onBack: () => void;
  initialSelectedTreeIds?: string[];
  onApplyAnotherOperation?: (treeIds: string[]) => void;
}
```

Ve `BulkMaintenanceForm`'un kendi içinde, tür seçimi HER ZAMAN sabit bir varsayılanla başlıyor:
```
const [maintenanceType, setMaintenanceType] = useState<MaintenanceTypeValue>(MaintenanceType.Irrigation);
```

### Bunun Gerçek Sonucu

Kullanıcı menüde "Sulama", "Gübreleme", "İlaçlama", "Budama" veya "Biçme" — hangisine tıklarsa tıklasın, açılan form her zaman "Sulama" (Irrigation) seçili olarak açılıyor. Kullanıcı, formun içindeki dropdown'dan manuel olarak türü değiştirmedikçe, "İlaçlama"ya bastığında karşısına yine "Sulama" çıkıyor. Bu, muhtemelen kullanıcının "Toplu İlaçlama beklenen şekilde görünmüyor" gözleminin gerçek teknik nedeni.

Aynı kök neden, "Sulama Başlangıç/Bitiş Saati" gözlemini de açıklıyor — bu alanlar `isIrrigation = maintenanceType === MaintenanceType.Irrigation` koşuluna bağlı, ve `maintenanceType` her zaman `Irrigation` olduğu için, bu alanlar her zaman görünüyor (kullanıcı hangi türe tıklarsa tıklasın) — kullanıcı bunu "diğer türlerde neden Sulama Saati alanları çıkıyor, İlaçlama'ya özel bir şey yok" şeklinde yorumlamış olabilir.

---

## Kullanıcının 8 Sorusuna Tek Tek Cevap

### 1. Yeni componentler gerçekten kullanılıyor mu?

Evet, gerçekten kullanılıyor — eski component render edilmiyor. `BulkOperationsScreen`, `BulkMaintenanceForm`'u ve `BulkObservationForm`'u gerçekten import edip render ediyor. Eski/paralel bir form bulunamadı — proje genelinde `BulkMaintenanceForm` adında başka bir dosya yok.

### 2. BulkMaintenanceForm / BulkObservationForm uygulamada gerçekten açılıyor mu?

Evet, açılıyor — ama her zaman "Sulama" türü seçili olarak (yukarıdaki kök neden). Başka eski bir form kullanılmıyor.

### 3. Routing/Navigation/Menu/Action/Button — hangi component'i çağırıyor?

Gerçek zincir doğrulandı, tam sağlam:
`ParcelForm` (buton) → `ParcelsScreen` (`onViewBulkOperations`) → `AppRouter.tsx` (`navigate(buildPath.parcelBulkOperations(parcel.id))`) → `ROUTE_PATTERNS.parcelBulkOperations` → `BulkOperationsScreenRoute` → `BulkOperationsScreen` → (menü seçimine göre) → `BulkMaintenanceForm`/`BulkObservationForm`.

Routing katmanında hiçbir sorun yok — sorun, `BulkOperationsScreen`'in kendi iç veri aktarımında (Madde 3'ün cevabı).

### 4. Feature Flag / Condition / Permission / Environment sebebiyle UI gizleniyor olabilir mi?

Hayır. Gerçek kod incelemesiyle doğrulandı: `src/modules/bulkOperations/` içinde hiçbir feature flag, environment kontrolü veya permission kontrolü yok. `SelectField` standart bir HTML `<select>` — gizli değil, her zaman görünür.

### 5. Yeni migration çalışıyor mu? Schema doğru yükleniyor mu?

Evet. `CURRENT_SCHEMA_VERSION = 12` doğrulandı (Sprint 10.4'ün eklediği `start_time`/`end_time` sütunları dahil). Bu, Sprint 10.4'ün kendi migration testleriyle (4 test) ayrıca doğrulanmıştı — bu analizde tekrar teyit edildi, gerçek bir sorun bulunmadı.

### 6. TimeField componenti gerçekten kullanılıyor mu?

Evet, gerçekten kullanılıyor — proje içinde "durmuyor". `BulkMaintenanceForm.tsx` ve `BulkObservationForm.tsx`'te gerçekten import edilip render ediliyor (tarih/saat alanları için 2 kez, Sulama başlangıç/bitiş için 2 kez daha — toplam 4 kullanım `BulkMaintenanceForm`'da).

### 7. Commit edilen kod eksik mi? Yoksa entegrasyon mu eksik?

Kod eksik değil — entegrasyon eksik. Tüm dosyalar (migration, domain, repository, `TimeField`, `dateInputConversion.ts` yardımcıları, form güncellemeleri) gerçekten mevcut ve doğru yazılmış. Eksik olan tek şey, `BulkOperationsScreen`'in menü seçimini (`initialType`) alt forma iletmemesi — bu, tek bir prop'un unutulduğu, küçük ve nokta atışı bir entegrasyon eksikliği.

### 8. "Toplu Sulama", "Toplu İlaçlama" vb. ekranları gerçekten var mı?

Bu, açıkça belirtilmesi gereken önemli bir nokta: Hayır, bunlar hiçbir zaman ayrı ekranlar olarak tasarlanmadı. Sprint 10.1'in kendi mimari kararı (`docs/saha-operasyonlari-mimari-analiz.md`, Kritik Bulgu 2) şuydu: "Toplu Sulama/Gübreleme/İlaçlama/Budama/Biçme — 4 ayrı özellik DEĞİL, `MaintenanceType` enum'unun 4 değeri — TEK bir 'Toplu Bakım Kaydı Oluşturma' mekanizması." Yani gerçek tasarım, menüde 5 ayrı buton, ama hepsi AYNI `BulkMaintenanceForm`'u açıyor, sadece formun içindeki dropdown'ın başlangıç değeri farklı olmalı.

Bu, kullanıcının "ayrı ekranlar" beklentisiyle kısmen uyuşmuyor olabilir — ama bu bir eksik geliştirme değil, bilinçli bir mimari karar. Sorun, bu kararın doğru uygulanmamış olması: menü butonlarının "hangi türe tıklandığını" forma iletmesi gerekiyordu, iletmiyor.

---

## Etkilenen Dosyalar (Değiştirilmedi — Sadece Tespit)

| Dosya | Sorun |
|---|---|
| `src/modules/bulkOperations/BulkOperationsScreen.tsx` | `initialType`, `BulkMaintenanceForm`'a hiç geçirilmiyor |
| `src/modules/bulkOperations/BulkMaintenanceForm.tsx` | `initialMaintenanceType` gibi bir prop kabul etmiyor, `maintenanceType` state'i hep sabit `Irrigation` ile başlıyor |

Not: Bu iki dosya dışında hiçbir dosyada gerçek bir sorun bulunmadı — migration, repository, `TimeField`, i18n, routing hepsi doğrulanmış şekilde çalışıyor.

---

## Çözüm Önerisi (Onay Bekleniyor — Henüz Uygulanmadı)

1. `BulkMaintenanceFormProps`'a `initialMaintenanceType?: MaintenanceTypeValue` eklenmeli.
2. `maintenanceType` state'inin başlangıç değeri, `initialMaintenanceType ?? MaintenanceType.Irrigation` olmalı (lazy initializer, Sprint 10.3'teki `useTreeSelection`'ın `initialSelectedIds` deseniyle tutarlı — `useEffect` gerektirmeden).
3. `BulkOperationsScreen`'in `view.mode === "maintenance"` dalında, `initialMaintenanceType={view.initialType}` olarak gerçekten iletilmeli.
4. Bu değişiklik test edilmeli: her 5 menü butonunun, açılan formda doğru türün seçili olduğunu kanıtlayan gerçek testler eklenmeli (mevcut testlerde bu senaryo hiç kontrol edilmemiş — bu da, sorunun neden daha önce test tarafından yakalanmadığını açıklıyor).

Bu değişiklik küçük, tek noktalı, mevcut mimariyi bozmuyor — yeni bir özellik değil, var olan bir bağlantının tamamlanması.

---

## Neden Testler Bu Sorunu Yakalamadı? (Ek Bulgu)

Mevcut `BulkOperationsScreen.test.tsx` ve `BulkMaintenanceForm.test.tsx` incelendiğinde, hiçbir testin "menüde X türüne tıklandığında, açılan formda X türünün seçili olduğu" senaryosunu kontrol etmediği görüldü. Testler, formun kendi başına doğru çalıştığını (varsayılan Irrigation ile) ve menünün doğru forma geçtiğini ayrı ayrı doğruluyordu — ama ikisi arasındaki veri aktarımını hiç test etmemişti. Bu, gerçek bir test kapsamı boşluğu — çözüm uygulanırken bu boşluk da kapatılmalı.
