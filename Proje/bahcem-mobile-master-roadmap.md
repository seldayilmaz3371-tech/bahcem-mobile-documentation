# Bahçem Mobile — MASTER ROADMAP

**Tarih:** 2026-07-21 · **Kural:** Bu belge SADECE mevcut kod tabanının ve belgelerin incelenmesiyle hazırlandı. Hiçbir kod yazılmadı, hiçbir dosya değiştirilmedi. İlk proje planı değil, bugünkü gerçek kod tabanı esas alındı.

**Temel gerçek metrikler (doğrulandı):** 137 commit · 681 otomatik test · Şema Sürümü 12 · 26 ADR · Sürüm `0.1.0-beta.1`

---

## 1. Projedeki Tüm Modüller

| # | Modül Adı | Durum | Tamamlanma % | Geliştirildiği Sprintler |
|---|---|---|---|---|
| 1 | Altyapı (SQLite şifreleme, kilit ekranı, i18n, marka) | ✅ Resmen kapandı, FROZEN | 100% | Sprint 1.x |
| 2 | Parseller + Ağaçlar | ✅ Resmen tamamlandı, FROZEN | 100% | Sprint 2.1-2.6 |
| 3 | Gözlemler + Fotoğraflar + Toplu Ağaç | ✅ Resmen tamamlandı, FROZEN | 100% | Sprint 3.1-3.10.1 |
| 4 | Router Migration + Finans | ✅ Resmen kabul edildi, FROZEN (bkz. Bölüm 6 — bir açık nokta var) | 98% | Sprint 4.0-4.3.2 |
| 5 | Bakım Yönetimi | ✅ Resmen tamamlandı, FROZEN | 100% | Sprint 5.1-5.5 |
| 6 | AI Altyapısı (Provider/Tool/Context/Sohbet) | 🟡 Kod kalitesi Production seviyesinde, gerçek cihaz doğrulaması bekliyor | 80% | Sprint 6, 7.1-7.5, 10.6 |
| 7 | Hasat | ✅ Tam işlevsel | 100% (kod) | Sprint 8.1-8.3 |
| 8 | Dashboard | ✅ Tam işlevsel (buton girişli, ana ekran değil) | 100% (kod) | Sprint 8.4-8.5 |
| 9 | Fotoğraf Analizi (AI) | ✅ Tam işlevsel, navigasyon dahil | 90% (geçmiş/karşılaştırma yok) | Sprint 9.1-9.2, 10.5, 10.6 |
| 10 | Saha Operasyonları (Toplu İşlemler) | ✅ Tam işlevsel | 95% (Filtre/Şablon/Biçme-etiket eksik) | Sprint 10.1-10.4 (+ Düzeltme Paketi) |
| 11+ | Hava Durumu / Harita / Sesli Asistan / RAG / Raporlar / Ayarlar Hub Genişletmesi / Finans+Hasat İşletme-Seviyesi Revizyonu | ⚪ Hiç başlamadı | 0% | — |

**Not — "Tamamlanma %" nasıl hesaplandı:** Modül 1-5, 7-8 için kullanıcının kendi kabul kararları/resmi kapanışları temel alındı (%100 = kod + kullanıcı onayı). Modül 6, 9, 10 için "kod tamamlandı ama gerçek cihaz doğrulaması/bazı alt özellikler eksik" durumu dikkate alınarak %100'ün altında bir değer verildi — bu, **kesin bir ölçüm değil, bu raporun kendi değerlendirmesidir**.

---

## 2. Sprint Geçmişi (Sprint 1'den Bugüne)

| Sprint | Yapılan İşler | Durum |
|---|---|---|
| 1.x | Şifreli SQLite, migration sistemi, kilit ekranı (biyometrik/PIN), Secure Storage, i18n | ✅ Tamamen kapandı |
| 2.1-2.6 | Parsel/Ağaç repository-hook-form-screen, navigasyon, Android geri tuşu, ADR 0022 (FK) düzeltmesi | ✅ Tamamen kapandı |
| 3.1-3.9 | Gözlem/Fotoğraf repository-hook-form-screen, kamera/galeri entegrasyonu | ✅ Tamamen kapandı |
| 3.10 | Toplu Ağaç Oluşturma (`runInTransaction`'ın ilk kullanımı) | ✅ Tamamen kapandı |
| 3.10.1 | **P0/P1 gerçek cihaz hotfix'i** (revizyon aldı) | ✅ Yeniden doğrulandı |
| 4.0-4.0.1 | Router Migration (HashRouter, 5 üst-düzey rota) | ✅ Tamamen kapandı |
| 4.1-4.3 | Finans modülü (kuruş-INTEGER kararı, Error Code Standard'ın ilk UI tüketimi) | ✅ Tamamen kapandı |
| 4.3.1 | **Bağımsız Mimari Denetim + P1 düzeltmeleri** (revizyon aldı) — para birimi Şema Sürüm 7'ye taşındı | ✅ Tamamen kapandı |
| 4.3.2 | **P0-001 Hotfix** (Kamera Intent + LockScreen regresyonu) | 🟡 Kod tamamlandı — **gerçek cihaz doğrulaması hâlâ "bekleniyor" olarak kayıtlı** (Bölüm 6'da detaylandırıldı) |
| 5.1-5.5 | Bakım Kayıtları + Bakım Planları + navigasyon + Yaklaşan/Geciken görünümü | ✅ Tamamen kapandı |
| 6, 7.1-7.5 | AI Altyapısı (Provider/Tool/Context Engine, Sohbet, navigasyon, bundle optimizasyonu, mobil UX, Beta versiyon, Release Signing mimarisi) | ✅ Kod tamamlandı |
| 8.1-8.3 | Hasat (migration, repository, UI, navigasyon) | ✅ Kod tamamlandı |
| 8.4-8.5 | Dashboard (hook, screen, navigasyon) | ✅ Kod tamamlandı |
| 9.1-9.2 | Fotoğraf Analizi (kod öncesi analiz + Gemini Vision entegrasyonu) | ✅ Kod tamamlandı |
| 10.1-10.3 | Saha Operasyonları (repository, UI, saha UX iyileştirmeleri + navigasyon) | ✅ Kod tamamlandı |
| 10.4 | Geriye dönük tarih/saat + Sulama Başlangıç/Bitiş Saati | ✅ Kod tamamlandı — ama **navigasyon entegrasyon hatası** ortaya çıktı |
| 10.4 Düzeltme Paketi | `initialMaintenanceType` kök neden düzeltmesi + Liste/Detay saat gösterimi | ✅ **Revizyon — gerçek bir üretim hatası düzeltildi** |
| 10.5 | Fotoğraf Analizi navigasyon entegrasyonu | ✅ Kod tamamlandı |
| 10.6 | AI Production Ready (tool-calling kök neden düzeltmesi, hata görünürlüğü, Hasat aracı, prompt kalitesi) | ✅ **Revizyon — kesin kanıtlı bir AI hatası düzeltildi** |

### Hangi Sprintler Tamamen Kapandı (Kullanıcı Onaylı, Bir Daha Dönülmeyecek)
Sprint 1.x, 2.1-2.6, 3.1-3.9, 4.0-4.3.1, 5.1-5.5.

### Hangileri Revizyon Aldı
- **3.10.1**: P0/P1 gerçek cihaz hotfix'i.
- **4.3.1**: Bağımsız Mimari Denetim sonucu para birimi mimarisi değişti.
- **4.3.2**: Hâlâ "gerçek cihaz doğrulaması bekliyor" durumunda — teknik olarak henüz "kapanmamış" bir revizyon.
- **10.4**: Kod tamamlandıktan sonra kullanıcı "beklenen şekilde görünmüyor" bildirdi, kök neden analiziyle gerçek bir entegrasyon hatası (`initialMaintenanceType`) bulunup Düzeltme Paketi ile giderildi.
- **AI (Sprint 6-7.5 sonrası)**: Sprint 10.6'da, AI X-Ray denetiminin bulduğu kesin kanıtlı bir kök neden (tool-calling API sözleşme ihlali) düzeltildi.

---

## 3. Bugün Gerçekten Çalışan Özellikler

- Parsel/Ağaç tam CRUD, arama/sıralama/sayfalama, Reference/Parcel Mode.
- Gözlem CRUD (5 tür) + Fotoğraf (kamera/galeri/kalıcı depolama).
- Toplu Ağaç Oluşturma.
- Finans (maliyet/satış kaydı, kuruş hassasiyetinde).
- Bakım Kayıtları (7 tür) + Bakım Planları (Yaklaşan/Geciken/Bugün görünümü).
- Hasat kayıtları (parsel/ağaç bazlı).
- Dashboard özet ekranı.
- AI Sohbet (genel + parsel bağlamlı, 6 salt-okunur veri aracıyla — Parcel/Tree/Observation/Maintenance/Finance/**Harvest**).
- Fotoğraf Analizi (AI, gözlemsel, teşhis/tedavi önerisi olmadan) — navigasyona tam bağlı.
- Toplu İşlemler (Toplu Gözlem + Toplu Bakım — 5 tür tek formda, geriye dönük tarih/saat, Sulama Başlangıç/Bitiş Saati, Ardışık İşlem Sihirbazı, Son Kullanılan İşlem, Geri Al).
- Kilit ekranı (biyometrik/PIN), dil değiştirme (TR/EN, tam döngü doğrulanmış).

---

## 4. Altyapısı Hazır, Kullanıcıya Açılmamış Özellikler

| Özellik | Durum |
|---|---|
| Çoklu AI Provider desteği | `ProviderRegistry` zaten tam destekliyor — sadece Gemini kayıtlı, kullanıcıya seçim sunulmuyor |
| RAG hook noktası | `IContextEngine` arayüzü bunu öngörmüş, gerçek bir `SemanticContextEngine` implementasyonu yok |
| Fotoğraf analiz sonucu kalıcı saklama | Sprint 9.1'in tasarım belgesinde 3 seçenek değerlendirilmiş, bilinçli olarak uygulanmamış (Seçenek C seçildi) |

---

## 5. Hiç Başlanmamış Özellikler

- Hava Durumu modülü
- Harita modülü
- Sesli Asistan
- Conversation Memory (AI sohbet geçmişine devam etme + `ai_facts` uzun süreli hafıza)
- Bitki/Tür Tanıma (AI)
- Hastalık/Zararlı Teşhisi (AI) — **bilinçli olarak yasaklanmış**, sistem promptunun kendisi bunu engelliyor
- Tedavi/Gübre/İlaç/Sulama Önerileri (AI) — **bilinçli olarak yasaklanmış**
- Geçmiş Analiz Karşılaştırması (Fotoğraf)
- Model Seçimi UI'ı
- Raporlar modülü
- Ayarlar Hub Genişletmesi (Dil/Tema/Yedekleme gibi genel ayarlar — bugün Ayarlar sadece AI girişi içeriyor)
- Finans + Hasat İşletme-Seviyesi Revizyonu (büyük mimari değişiklik, henüz bir ADR'ye bağlanmamış)
- İşlem Şablonları / Favori İşlemler (Toplu İşlemler'in ertelenen genişlemeleri)
- Filtre Sistemi (Toplu İşlemler'de Aktif/Pasif/Hasta/Sağlıklı/Genç/Yaşlı) — **veri modeli bunu desteklemiyor**, gerçek bir bulgu

---

## 6. Teknik Borçlar

| Öncelik | Borç | Kaynak |
|---|---|---|
| **Yüksek** | P0-001 hotfix'in (Sprint 4.3.2) gerçek cihaz doğrulama durumu belirsiz — belgede hâlâ "bekleniyor" yazıyor | Modül 4 kayıtları |
| **Yüksek** | AI modülünün gerçek cihazda tam doğrulaması yapılmadı (Sprint 10.6'nın tool-calling düzeltmesi dahil) | Modül 6 kayıtları |
| Orta | "Biçme" olarak oluşturulan Toplu İşlem kayıtları, Liste/Detay ekranlarında "Diğer" olarak görünüyor | Sprint 10.4 Düzeltme Paketi analizi |
| Orta | Liste virtualization yok (4+ modülde, `TreeSelectorList` dahil) | Modül 4 Bağımsız Denetimi, Sprint 10.2 |
| Orta | Fotoğraf galerisinde gerçek thumbnail üretimi yok | Modül 3 kapanışı |
| Düşük | 4 modülde tekrarlanan List bileşen deseni (`EntityList<T>` soyutlaması yok) | Modül 4 Bağımsız Denetimi — YAGNI gerekçesiyle bilinçli erteleme |
| Düşük | `runInTransaction()`'ın ANR riski gerçek cihazda doğrulanmadı | Sprint 10.2/10.3 |
| Düşük | UX öz-denetiminin bulduğu 8 madde (en kritiği: Toplu İşlemler'de Undo butonunun görsel ayrışmaması) | Sprint 10.3 |

---

## 7. Mimari Açıdan Eksik Kalan Noktalar

- **Git senkronizasyonu**: Kullanıcının kendi ortamında yerel/GitHub geçmişleri arasında gerçek bir diverge tespit edilmişti (ayrı bir analiz sürecinde) — bu, kod mimarisi değil ama proje sürdürülebilirliği açısından çözülmesi gereken açık bir konu.
- **Sürüm/Tag disiplini**: v0.2.0/v0.3.0/v0.4.0 git tag'leri hiç uygulanmadı (öneri olarak kaldı).
- **İmzalama (Keystore)**: Mimarisi belgelendi (ADR 0026) ama gerçek keystore kullanıcının kendi ortamında henüz oluşturulmadı — Beta dağıtımının önündeki tek kritik engel.
- **CHANGELOG**: Proje genelinde geç (Sprint 10.4 Düzeltme Paketi'nde) oluşturuldu, geriye dönük tam bir liste içermiyor.
- **Finans/Hasat mimarisinin gelecekteki büyük revizyonu**: Roadmap'te belirtilmiş ama gerçek bir ADR'ye henüz bağlanmamış, kapsamı/zamanlaması belirsiz.

---

## 8. AI Modülünün Güncel Durumu

### Bugün Neler Yapabiliyor
- Serbest soru sorma (genel + parsel bağlamlı sohbet).
- Uygulama verisiyle ilgili sorular (6 salt-okunur araç: Parcel/Tree/Observation/Maintenance/Finance/Harvest) — **artık gerçek bir kök neden düzeltmesiyle güvenilir çalışıyor** (Sprint 10.6).
- Fotoğraf analizi (gözlemsel, kategorize edilmiş rehberlikle — yaprak/dal/meyve, çevresel faktör gözlemi, belirsizlik durumunda açık belirtme).
- 12 ayırt edici hata kodu ile kullanıcı dostu hata mesajları (kimlik doğrulama/kota/rate limit/ağ/timeout/araç bulunamadı vb.).

### Hangi AI Özellikleri Final Sürüme Kaldı
- Conversation Memory (yeni migration gerektiriyor).
- Bitki/Tür Tanıma.
- Hastalık/Zararlı Teşhisi ve Tedavi Önerileri (güvenlik kararı gerektiren, bilinçli olarak ertelenmiş).
- Geçmiş Analiz Karşılaştırması.
- Model Seçimi UI'ı.
- Sesli Asistan.
- RAG/Embedding (gerçek semantik context engine).

---

## 9. Final Sürüme Ulaşmak İçin Kalan İşler (Önem Sırasına Göre)

1. **İmzalama (Keystore) — kullanıcının kendi ortamında.** Beta dağıtımının tek kritik engeli.
2. **Gerçek cihazda tam doğrulama** — özellikle AI'nin tool-calling düzeltmesi ve P0-001 hotfix'inin durumu netleştirilmeli.
3. **Git senkronizasyonu tamamlanmalı** — yerel/GitHub geçmişi birleştirilmeli (ayrı süreçte başlatıldı).
4. **Toplu İşlemler'in bilinen küçük eksiklerinin kapatılması** — "Biçme" etiket sorunu, UX öz-denetiminin Undo görsel vurgusu önerisi.
5. **Hasat-Finans mimari kararının verilmesi** — Dashboard v2'nin önünde bir bağımlılık.
6. **Modül 9 navigasyonu** zaten tamamlandı (bu madde artık geçerli değil, referans için not düşüldü).
7. Beta sonrası: Ayarlar Hub genişletmesi, Hava Durumu, Raporlar, Harita — düşük öncelik.
8. En son: Sesli Asistan, RAG/Embedding, Bitki Tanıma/Hastalık Teşhisi (kapsamlı, ayrı güvenlik değerlendirmesi gerektiren büyük modüller).

---

## 10. Yeni Sprint Roadmap (Sprint 10.6'dan Sonra)

| Sprint | Amaç | Tahmini Kapsam |
|---|---|---|
| **10.7** | Git senkronizasyonu tamamlanması (kullanıcının ortamında) + P0-001 hotfix'in gerçek cihaz doğrulama durumunun netleştirilmesi | Kod değişikliği yok/minimal — süreç ve doğrulama odaklı |
| **10.8** | İmzalama + gerçek cihaz tam doğrulama turu (AI tool-calling düzeltmesi dahil tüm modüllerin cihazda son kontrolü) | Kullanıcı odaklı, Claude'un bu ortamda yapamadığı işler |
| **10.9** | Toplu İşlemler'in bilinen küçük eksikleri (Biçme etiketi, Undo görsel vurgusu, isteğe bağlı Filtre/Şablon değerlendirmesi) | Küçük, düşük riskli, tek modül |
| **11.0** | Hasat-Finans mimari kararı + gerçek ADR + migration | Orta ölçekli, büyük bir mimari karar içeriyor |
| **11.1** | Dashboard v2 (Hasat-Finans revizyonuna bağımlı) | Orta ölçekli |
| **12.0** | Ayarlar Hub genişletmesi (Dil/Tema/Yedekleme) | Küçük-orta ölçekli |
| **13.0** | Hava Durumu modülü (yeni bir dış veri kaynağı/API kararı gerektirebilir — kullanıcı onayı şart) | Yeni modül, kapsamı netleşmemiş |
| **14.0** | Raporlar modülü | Yeni modül |
| **15.0** | Harita modülü | Yeni modül, konum izinleri/harita SDK kararı gerektirir |
| **16.0+** | Conversation Memory, RAG/Embedding, Bitki Tanıma, Hastalık Teşhisi, Sesli Asistan | Büyük, güvenlik açısından hassas kararlar gerektiren AI genişlemeleri — her biri ayrı, dikkatli bir sprint |

**Not:** Bu sıralama, bu raporun kendi değerlendirmesidir — kesin bir taahhüt değil, kullanıcının onayı/önceliklendirmesiyle değişebilir.

---

## 11. Genel Tamamlanma Oranı

### Modül Bazında
| Modül | % |
|---|---|
| 1-5 (Altyapı, Parsel/Ağaç, Gözlem/Fotoğraf, Router/Finans, Bakım) | 100% (kullanıcı onaylı) |
| 6 (AI Altyapısı) | 80% |
| 7 (Hasat) | 100% (kod), gerçek cihaz onayı ayrı |
| 8 (Dashboard) | 100% (kod) |
| 9 (Fotoğraf Analizi) | 90% |
| 10 (Saha Operasyonları) | 95% |
| 11+ (başlamamış modüller) | 0% |

### Genel Proje Bazında
**Kesin bir sayısal yüzde, bu raporun kendi tahminidir — belgelerde resmi olarak kayıtlı değildir.** Tamamlanan modül sayısı (10 modül, çeşitli olgunluk seviyelerinde) ile hiç başlamamış modül sayısı (7+ büyük özellik alanı) birlikte değerlendirildiğinde, bu raporun değerlendirmesi: **proje çekirdek işlevselliği (saha veri yönetimi) açısından ~%85-90 olgun, genişletilmiş vizyon (AI'nin ileri yetenekleri, Hava Durumu, Harita, Raporlar) açısından ~%35-40 olgun.**

### AI Bazında
Sprint 10.6'nın kendi teknik raporundaki değerlendirme (bu incelemede teyit edildi): **Genel AI Tamamlanma Oranı ~80%** (kök neden düzeltmesi + hata görünürlüğü + Hasat aracı sonrası, önceki ~65%'ten yükseldi) — ama bu, **kullanıcıya açık özelliklerin genişlemesinden değil, doğruluk ve görünürlük kazanımlarından** kaynaklanıyor. Gerçek cihaz doğrulaması hâlâ eksik olduğu için bu oran "kod olgunluğu" anlamındadır, "üretimde kanıtlanmış" anlamında değildir.
