# Bahçem Mobile — Kapsamlı Proje Durum Raporu

**Tarih:** 2026-07-19 · **Kural:** Bu belge SADECE mevcut proje dokümanlarının incelenmesiyle hazırlandı. Hiçbir kod yazılmadı, hiçbir proje dosyası değiştirilmedi — sadece belgeler okundu. **Bu rapor bilinçli olarak proje deposunun DIŞINDA (docs/ klasörüne yazılmadı) tutuldu** — kullanıcının "hiçbir dosyayı değiştirme" talimatına tam uyum için.

**İncelenen kaynaklar:** `BUILD_INFO.md`, `docs/module-status.md` (382 satır, tam okundu), `docs/roadmap/01-current-state-and-roadmap.md`, `docs/roadmap/02-final-ui-design.md`, `docs/adr/` (26 ADR), `README.md`, `docs/apk-beta-readiness-checklist.md`, ve ilgili sprint teknik raporları.

**🔴 Metodolojik not (baştan belirtilmesi gerekiyor):** Bu inceleme sırasında, proje belgeleri arasında **güncellik tutarsızlıkları** bulundu — bazı üst-düzey özet belgeler (ana roadmap dosyasının "Özet Tablo"su, `apk-beta-readiness-checklist.md`) eski sprint'lerden kalma bilgiler içeriyor, ama `docs/module-status.md`'nin **modül bazlı detay bölümleri** tutarlı bir şekilde güncellenmiş durumda. Bu rapor, çelişki olduğunda **en son tarihli, en detaylı kaynağı** esas almıştır — her çelişki Bölüm 7'de ayrıca belgelenmiştir.

---

## 1. Genel Proje Durumu

| Alan | Değer |
|---|---|
| **Mevcut sürüm** | `0.1.0-beta.1` (`package.json`/`android/app/build.gradle`, Sprint 7.4'te gerçekten uygulandı) |
| **Son tamamlanan sprint** | Sprint 10.3 (2026-07-18) — Saha Operasyonları UX iyileştirmeleri + navigasyon + bağımsız UX öz-denetimi |
| **Son tamamlanan modül (tam işlevsel)** | Modül 10 — Saha Operasyonları (Toplu İşlemler) — Sprint 10.3'te "TAM İŞLEVSEL" durumuna ulaştı |
| **Son BUILD_INFO commit** | `86a877c` |
| **Toplam otomatik test** | 607 (BUILD_INFO.md'ye göre, Sprint 10.3 sonu) |
| **Toplam ADR** | 26 (gerçek dosya sayımıyla doğrulandı — roadmap belgesindeki "25" rakamı eskimiş) |
| **Genel tamamlanma yüzdesi** | **Belgelerde sayısal bir "%" değeri resmi olarak kayıtlı değil.** Roadmap belgesinin kendi tahmini ("~35-40%") Sprint 8.4 civarından kalma ve GÜNCEL DEĞİL — Modül 7-10 o zamandan beri tamamlandı. Kesin bir yüzde **uydurulmuyor**; bunun yerine Bölüm 2'deki modül tablosu somut veriyi sunuyor. |

---

## 2. Tamamlanan Modüller

| # | Modül Adı | Durum | Tamamlanan Özellikler | Eksik Kalan Noktalar |
|---|---|---|---|---|
| 1 | Altyapı | ✅ Resmen kapandı (2026-07-15) | Temel altyapı | Yok (kapalı) |
| 2 | Parseller + Ağaçlar | ✅ Resmen tamamlandı (v0.2.0) | Parsel/Ağaç tam CRUD, arama/sıralama/sayfalama, Reference/Parcel Mode, Android geri tuşu, Error Code Standard, ADR 0022 düzeltmesi | Git Tag v0.2.0 **hiç uygulanmadı** (öneri olarak kaldı) |
| 3 | Gözlemler + Fotoğraflar + Toplu Ağaç | ✅ Resmen tamamlandı (v0.3.0) | Gözlem CRUD (5 tür), Fotoğraf (Kamera/Galeri/kalıcı depolama), Toplu Ağaç Oluşturma (`runInTransaction` ilk kullanımı) | Fotoğraf galerisinde gerçek thumbnail üretimi yok; Git Tag v0.3.0 uygulanmadı |
| 4 | Router Migration + Finans | ✅ Resmen kabul edildi — ACCEPTED/FROZEN/PRODUCTION READY | HashRouter migration, Finans (cost/sale, kuruş INTEGER), P0-001 Kamera/LockScreen hotfix'i **kod olarak** tamamlandı | 🔴 P0-001 hotfix'in (Sprint 4.3.2) gerçek cihaz doğrulaması belgede "bekleniyor" olarak kayıtlı kalmış (Bölüm 7'de detaylandırıldı); Git Tag v0.4.0 uygulanmadı; liste virtualization yok |
| 5 | Bakım Yönetimi | ✅ Resmen tamamlandı (Sprint 5.1-5.5, FROZEN) | Bakım Kayıtları (7 tür, audit-log), Bakım Planları, navigasyon, Yaklaşan/Geciken/Bugün görünümü | Yok — dondurulmuş modül |
| 6 | AI Altyapısı | 🟡 Kod/Navigasyon/Mobil UX tamamlandı, **Production Ready DEĞİL** | Provider Registry, Tool Registry (5 salt-okunur araç), Context Engine, Sohbet, Bundle optimizasyonu (744kB→396kB), Mobil UX, Beta versiyon numaraları uygulandı | 🔴 Gerçek cihaz TAM doğrulaması bekliyor; 🔴 İmzalama (keystore) — Beta Release'in **tek kritik engeli**; imzalı APK testi yapılmadı |
| 7 | Hasat | ✅ Tam işlevsel (Sprint 8.3) | Migration, Repository, Hook/Form/Screen, Parsel/Ağaç/Referans navigasyonu | Finans entegrasyonu sadece tasarım aşamasında, kesin karar bekliyor |
| 8 | Dashboard | ✅ Tam işlevsel (Sprint 8.5) | Özet hook, Screen, Parseller ekranından buton girişi (ana ekran DEĞİL, bilinçli karar) | Bilinen ölçek sınırı (varsayılan limit aşılırsa eksik gösterir) |
| 9 | Fotoğraf Analizi (AI) | 🟡 İlk çalışan akış tamamlandı (Sprint 9.2) | `AIProvider.analyzeImage()`, Gemini Vision entegrasyonu, `usePhotoAnalysis`, `PhotoAnalysisScreen` | 🔴 **Navigasyon entegrasyonu hâlâ bekliyor** — PhotoGalleryScreen'den erişim yok; kalıcı saklama bilinçli olarak yok |
| 10 | Saha Operasyonları (Toplu İşlemler) | ✅ **Tam işlevsel** (Sprint 10.3) | Repository katmanı, ağaç seçim UI'ı + arama, Toplu Gözlem/Bakım formları, Son Kullanılan İşlem, Ardışık İşlem Sihirbazı, Undo güvenliği, navigasyon | Filtre sistemi (veri modeli desteklemiyor), İşlem Şablonları/Favoriler (tasarlandı, ertelendi), gerçek cihaz performans testleri çalıştırılmadı |

---

## 3. Tamamlanan Sprintler

| Sprint | Modül | Amaç/İçerik | Önemli Mimari Not |
|---|---|---|---|
| 2.1-2.6 | Modül 2 | Parsel/Ağaç repository, hook, form, screen, navigasyon, Android geri tuşu, ADR 0022 | Foreign key zorlaması bulgusu (ADR 0022) |
| 3.1-3.10.1 | Modül 3 | Observation/Photo repository-hook-form-screen, Toplu Ağaç Oluşturma, P0/P1 hotfix | `runInTransaction()`'ın ilk gerçek kullanımı (3.10); native transaction parametresi düzeltmesi (ADR 0023) |
| 4.0-4.3.2 | Modül 4 | Router migration (HashRouter), Finans modülü, Bağımsız Mimari Denetim, P0-001 hotfix | Para birimi INTEGER-kuruş kararı (Şema Sürüm 7); Router katmanının repository erişimi istisnası (Engineering Protocol Bölüm 21) |
| 5.1-5.5 | Modül 5 | Bakım Kayıtları (audit-log), Bakım Planları, navigasyon, Yaklaşan/Geciken görünümü | Audit-log deseni, `dueStatus` filtresi |
| 6, 7.1-7.5 | Modül 6 (AI) | Provider/Tool Registry, Context Engine, Sohbet (ADR 0024), navigasyon, bundle optimizasyonu, mobil UX, AI davranış doğrulaması, Beta versiyon uygulaması, Release Signing mimarisi (ADR 0025/0026) | ADR 0024 (AI mimarisi); `useTreeForRoute` soyutlaması (3. tekrar eşiği) |
| 8.1-8.5 | Modül 7 (Hasat) + Modül 8 (Dashboard) | Hasat migration/repository/UI/navigasyon; Dashboard hook/screen/navigasyon | Hasat-Finans işletme mimarisi tartışması başladı; Dashboard'ın ana ekran YAPILMAMASI kararı |
| 9.1-9.2 | Modül 9 (Fotoğraf Analizi) | Kod öncesi analiz, Gemini Vision entegrasyonu | `getActiveAiProvider()` çıkarması; AI analiz sonucu saklama kararı ertelendi |
| 10.1-10.3 | Modül 10 (Saha Operasyonları) | Repository katmanı + performans testi, Toplu İşlemler UI, Saha UX iyileştirmeleri + navigasyon | "Toplu Ağaç zaten var" ve "5 bakım türü tek mekanizma" bulguları; `localPreferences` ile Son Kullanılan İşlem |

**Not:** Modül 1'in sprint bazlı ayrıntısı bu incelemede tam olarak okunmadı (belgede özetlenmiş) — gerekirse ayrıca okunabilir.

---

## 4. Devam Eden Çalışmalar

Belgelere göre şu anda **aktif olarak kod yazımı sürmekte olan** bir çalışma **yok** — Sprint 10.3, resmi olarak kapatılmış son sprint. Ancak aşağıdaki maddeler **"bekleyen, karara bağlı olmayan" açık uçlar**:

1. **Modül 6 — Gerçek cihaz TAM doğrulaması** — henüz çalıştırılmadı.
2. **Modül 6 — İmzalama (keystore)** — mimarisi belgelendi (ADR 0026) ama gerçek keystore kullanıcının kendi ortamında henüz oluşturulmadı.
3. **Modül 9 — Navigasyon entegrasyonu** — kod hazır, PhotoGalleryScreen'den erişim henüz bağlanmadı.
4. **Modül 7 — Hasat/Finans entegrasyon kararı** — 3 seçenek tasarlandı, kesin karar bekleniyor.
5. **Modül 10 — Gerçek cihaz performans testleri** — test planı hazır, kullanıcının kendi cihazında çalıştırılması bekleniyor.
6. **Modül 4 — P0-001 hotfix'in gerçek cihaz doğrulaması** — belgede "bekleniyor" statüsünde kalmış görünüyor.

---

## 5. Planlanan Sprintler

Belgelerde **numaralandırılmış, resmi olarak onaylanmış** bir "sonraki sprint" listesi bulunmuyor — her modülün kendi "Sonraki Adım" notu var:

| Olası Sprint | İçerik | Öncelik | Tahmini Bağımlılık |
|---|---|---|---|
| Modül 9 Navigasyon | PhotoGalleryScreen → PhotoAnalysisScreen bağlantısı | Orta — kod zaten hazır | Yok, bağımsız yapılabilir |
| Modül 10 UX Düzeltmeleri | UX öz-denetiminin 8 maddesi (en kritiği: Undo görsel vurgusu) | Orta-Yüksek | Yok |
| Hasat-Finans Kararı | 3 seçenekten birinin seçilip uygulanması | Kullanıcı kararına bağlı | Yok |
| Modül 6 İmzalama + Gerçek Cihaz Doğrulama | Rehberin kullanıcı tarafından uygulanması, ardından tam test seti | **En Yüksek (Beta'nın önündeki tek engel)** | Kullanıcının kendi ortamında keystore oluşturması |
| Filtre/Şablon/Favoriler | Tasarlanan ama ertelenen özellikler | Düşük (kanıtlanmamış ihtiyaç) | Beta geri bildirimi |
| Finans+Hasat İşletme-Seviyesi Revizyon | Büyük bir mimari değişiklik | Belirtilmiş ama sıra netleşmemiş | Gerçek bir ADR gerekiyor (henüz yazılmadı) |

---

## 6. Planlanan Modüller

`docs/module-status.md`'nin resmi "Henüz Başlamayan Modüller" listesi:

- **Hava Durumu (Weather)**
- **Harita (Map)**
- **Sesli Asistan** (bilinçli ertelendi)
- **RAG/Embedding** (bilinçli ertelendi)
- **Ayarlar (Settings)** — 🔴 Nüans: `SettingsScreen.tsx` zaten var (Sprint 7.1'den beri), sadece "AI" girişini içeren bir hub olarak. Bu listedeki "Ayarlar" muhtemelen Dil/Tema/Yedekleme gibi genel ayarların eklenmesini kastediyor — belge bunu netleştirmiyor.
- **Raporlar (Reports)**

Ayrıca roadmap'in Product Owner Kararı bölümünde (henüz kodlanmamış):
- **Finans + Hasat İşletme-Seviyesi Revizyonu**

---

## 7. Teknik Borçlar

### Belgelerde Doğrudan Kayıtlı

| Kategori | Borç | Kaynak |
|---|---|---|
| Performans | Liste virtualization yok (4 modülde) | Modül 4 Bağımsız Denetimi — bilinçli erteleme |
| Performans | `TreeSelectorList`'te 500 satırlık liste sanallaştırma yok | Sprint 10.2/10.3 |
| Performans | `Dashboard`'ın varsayılan limit (50) aşılırsa eksik gösterme riski | Sprint 8.4 |
| Refactor İhtiyacı | 4 modülde tekrarlanan List bileşen deseni | Modül 4 Bağımsız Denetimi — YAGNI gerekçesiyle ertelendi |
| UX Eksikliği | Fotoğraf galerisinde gerçek thumbnail üretimi yok | Modül 3 kapanışı |
| UX Eksikliği | UX öz-denetiminin 8 maddesi (en kritiği: Undo butonunun görsel ayrışmaması) | Sprint 10.3 |
| Güvenlik | API anahtarının "düz metin bulunamadığı" tam doğrulanmadı | Sprint 7.2/9.1 civarı |
| Doğrulama Eksikliği | ANR riski gerçek cihazda doğrulanmadı | Sprint 10.2/10.3 |

### 🔴 Bu İncelemede Bulunan Belge Tutarsızlıkları (Yeni Bulgular)

1. **`module-status.md`'nin en üst "Genel Özet" tablosu eskimiş** — Modül 10'u hâlâ "Sprint 10.1 tamamlandı" gösteriyor, ama aynı belgenin detaylı Modül 10 bölümü "TAM İŞLEVSEL (Sprint 10.3)" diyor — belgenin kendi içinde tutarsızlık.
2. **Ana roadmap belgesinin "Özet Tablo"su** çok eskimiş — "484 test", "25 ADR" gibi Sprint 7.5 civarından kalma bilgiler içeriyor.
3. **`apk-beta-readiness-checklist.md`** "Sprint 7.3'te güncellendi" notuyla kalmış — "Versiyon Numarası: karar bekliyor" diyor, ama Sprint 7.4'te gerçekten uygulandı.
4. **Modül 4'ün P0-001 hotfix'inin gerçek cihaz doğrulama durumu belirsiz** — belgede "bekleniyor" yazıyor, Sprint 7.2'nin genel onayının bunu kapsayıp kapsamadığı netleşmiyor. Varsayım yapılmadan açık bir soru olarak bırakılıyor.

---

## 8. Beta Öncesi Yapılması Gerekenler (Önem Sırasına Göre)

1. **🔴 En kritik: İmzalama (Keystore).** Kullanıcının kendi ortamında gerçek bir keystore oluşturması gerekiyor — belgelerin kendi ifadesiyle "Beta Release'in tek kritik engeli."
2. **Gerçek cihaz TAM doğrulama** — zorunlu test seti henüz tam çalıştırılmadı.
3. **İmzalı Beta APK üretimi.**
4. **Modül 4'ün P0-001 hotfix doğrulama durumunun netleştirilmesi.**
5. **Git Tag disiplini** — v0.2.0/v0.3.0/v0.4.0 hiç uygulanmadı; ilk gerçek tag, imzalı APK dağıtıldığında atılmalı.

---

## 9. Beta Sonrası Planlananlar

1. Modül 9 navigasyon entegrasyonu.
2. **Finans + Hasat İşletme-Seviyesi Revizyonu** — büyük bir mimari değişiklik, ADR henüz yazılmadı.
3. Bu revizyondan sonra: Dashboard'ın yeni veri modeline göre yeniden tasarımı.
4. Ayarlar Hub Genişletmesi, Hava Durumu, Raporlar, Harita, Sesli Asistan, RAG/Embedding.

---

## 10. Sürüm Yol Haritası

**🔴 Dürüstçe belirtilmesi gereken bir sınır:** Belgelerde resmi olarak onaylanmış bir "v1.0/v1.1/v1.2" sürüm planı bulunamadı. Var olan tek somut sürüm kararı `0.1.0-beta.1`dir. ADR 0025'in versiyonlama politikası bir çerçeve sunuyor ama hangi özelliklerin hangi sürüm numarasına denk geleceğine dair resmi bir eşleme yok. Bu bölümde sayısal bir sürüm planı **uydurulmuyor**:

- **`0.1.0-beta.1`** — mevcut sürüm, imzalama sonrası ilk gerçek Beta dağıtımı için hedefleniyor.
- **Sonraki beta sürümleri** (`0.1.0-beta.2`, vb.) — her yeni Beta dağıtımında `beta.N` artırılacak.
- **`1.0.0`** — planlanan tüm modüller tamamlanmadan kullanılmayacak. Hangi modüllerin "1.0 için zorunlu" sayılacağı resmi olarak kararlaştırılmamış.

---

## 11. Güncel Yol Haritası (Kronolojik)

| Sprint | Modül | Durum |
|---|---|---|
| 1.x | Altyapı | ✅ Tamamlandı |
| 2.1-2.6 | Parseller + Ağaçlar | ✅ Tamamlandı |
| 3.1-3.10.1 | Gözlemler + Fotoğraflar + Toplu Ağaç | ✅ Tamamlandı |
| 4.0-4.3.2 | Router Migration + Finans | ✅ Tamamlandı (P0-001 doğrulama belirsizliği hariç) |
| 5.1-5.5 | Bakım Yönetimi | ✅ Tamamlandı |
| 6, 7.1-7.5 | AI Altyapısı | 🟡 Devam Ediyor (imzalama/gerçek cihaz tam doğrulama bekliyor) |
| 8.1-8.3 | Hasat | ✅ Tamamlandı (Finans entegrasyonu ⚪ Planlandı) |
| 8.4-8.5 | Dashboard | ✅ Tamamlandı |
| 9.1-9.2 | Fotoğraf Analizi | 🟡 Devam Ediyor (navigasyon bekliyor) |
| 10.1-10.3 | Saha Operasyonları | ✅ Tamamlandı (Filtre/Şablon/Favoriler ⚪ Planlandı) |
| — | İmzalama + Gerçek Cihaz Tam Doğrulama | ⚪ Planlandı (Beta'nın önündeki tek engel) |
| — | Finans+Hasat İşletme-Seviyesi Revizyonu | ⚪ Planlandı (büyük mimari değişiklik) |
| — | Hava Durumu | ⚪ Planlandı |
| — | Harita | ⚪ Planlandı |
| — | Ayarlar Hub Genişletmesi | ⚪ Planlandı |
| — | Raporlar | ⚪ Planlandı |
| — | Sesli Asistan | ⚪ Planlandı (yüksek risk) |
| — | RAG/Embedding | ⚪ Planlandı (düşük öncelik) |

---

## 12. Genel Değerlendirme (Proje Yöneticisi Bakış Açısıyla)

### Güçlü Yönler

- **Disiplin gerçekten sürdürülüyor:** 26 ADR, her sprintte kod öncesi analiz, gerçek dosya kanıtları.
- **Modül 1-5, 7-8, 10 gerçekten "tam işlevsel"** — testlerle ve (bazı modüllerde) gerçek cihaz onaylarıyla desteklenen bir durum.
- **YAGNI disiplini gerçek maliyet tasarrufu sağlamış** — Sprint 10.1'in bulguları gereksiz geliştirmeyi somut biçimde önlemiş.

### Riskli Noktalar

1. **Belge güncelliği tutarsızlığı** — üst-düzey özet belgeler eskiyor, ileride yanlış karar riski taşır.
2. **Modül 6'nın Production Ready durumu uzun süredir "henüz değil"** — imzalama adımı kullanıcının kendi eylemine bağlı, proje ilerlemesini dıştan bir bağımlılığa bağlıyor.
3. **Gerçek cihaz doğrulamalarının çoğu hâlâ yapılmadı** — kod kalitesi yüksek olsa da, gerçek saha/cihaz kanıtı birikmiş bir açık.
4. **Finans+Hasat işletme-seviyesi revizyonu, henüz hiçbir ADR'ye bağlanmamış büyük bir mimari değişiklik** olarak bekliyor.

### Öncelikli Geliştirme Alanları

1. İmzalama + gerçek cihaz tam doğrulama (Beta'nın önündeki tek engel).
2. Belge güncelliğinin bir senkronizasyon turuyla düzeltilmesi.
3. Modül 9 navigasyonu (küçük, düşük riskli, hızlı kazanım).
4. Hasat-Finans kararının verilmesi.

### Önerilen Yol

Sprint 10.4'ün **kod geliştirmeden önce**, bu raporun bulduğu belge tutarsızlıklarının bir senkronizasyon geçişiyle düzeltilmesi faydalı olur. Bunun dışında, en yüksek değerli sıradaki adım teknik değil, kullanıcının kendi eylemine bağlı (imzalama) — bu nedenle paralel olarak Modül 9 navigasyonu gibi düşük riskli, bağımsız işler ilerletilebilir.
