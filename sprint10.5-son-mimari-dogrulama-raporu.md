# Sprint 10.5 — Son Mimari Doğrulama Raporu

**Tarih:** 2026-07-20 · **Kural:** Bu belge SADECE mevcut kod tabanının incelenmesiyle hazırlandı. Hiçbir kod yazılmadı, hiçbir dosya değiştirilmedi, hiçbir commit oluşturulmadı.

---

## 1. API Maliyeti Analizi

### Gerçek Durum (Kod İncelemesiyle Doğrulandı)

`usePhotoAnalysis.ts` ve `GeminiProvider.ts` gerçekten okundu — **hiçbir cache mekanizması yok**. Tek "cache" referansı, API anahtarının okunmasıyla ilgili (`GeminiProvider`'ın kendi yorumu: anahtar her seferinde yeniden okunuyor, cache'lenmiyor — bu, ayarlar ekranından anahtar değiştirildiğinde tutarsızlık olmasın diye bilinçli bir karar, fotoğraf analiz *sonucuyla* ilgisi yok).

**Kullanıcının senaryosu için gerçek cevap:** Evet, her "Analiz Et" tıklaması — aynı fotoğraf, birkaç dakika arayla, ertesi gün fark etmeksizin — **gerçek, ücretli bir Gemini API çağrısı** yapar. Hiçbir tekrar önleme yok.

### Bu Bilinçli Bir Tercih mi?

**Kısmen — ama tam değil.** Sprint 9.1'in necessity analizi (`usePhotoAnalysis.ts`'in kendi yorumunda görülüyor) **sadece kalıcı (veritabanı) saklamayı** değerlendirmiş ve bilinçli olarak reddetmişti — gerekçe, "Karşılaştırmalı Fotoğraf" özelliğinin kapsam dışı olması. Ama bu analiz, **geçici (oturum içi) bir cache**'i hiç ele almamış. Bu, dürüstçe belirtilmesi gereken bir boşluk: mevcut "cache yok" davranışı, kalıcı saklamanın reddedilmesinin **dolaylı bir sonucu**, ama session-cache seçeneği ayrıca **hiç tartışılmamış**.

### Avantaj/Dezavantaj Analizi

| Seçenek | Avantaj | Dezavantaj |
|---|---|---|
| **Cache yok (bugünkü durum)** | En basit, sıfır ek karmaşıklık, "her analiz taze" garantisi | Kullanıcı yanlışlıkla iki kez basarsa veya aynı fotoğrafı tekrar ziyaret ederse gereksiz maliyet |
| **Session cache (in-memory, `Map<photoId, result>`)** | Aynı uygulama oturumunda tekrar tıklamaları önler, network gecikmesi yok | Ek state yönetimi (nerede tutulacak, ne zaman temizlenecek, "yeniden analiz et" nasıl bypass edilecek); sadece "bugün, aynı oturum" senaryosunu kapsar — "ertesi gün" senaryosunu kapsamaz |
| **Kalıcı cache (veritabanı, `photo_analyses` tablosu)** | "Ertesi gün" senaryosunu da kapsar; gelecekteki Zaman Çizelgesi/Karşılaştırma özelliğinin temelini atar | Migration + repository + "ne zaman geçersiz sayılır" tasarım kararı gerektirir — Sprint 9.1'in tasarım belgesi bunu zaten **"Fotoğraf Analizi'nin kendisi geliştirilmeye başlandığında, gerçek bir ADR ile değerlendirilmeli"** demişti |

### Öneri

**Session cache dahi bu sprintte eklenmemeli.** Gerekçe: (1) Henüz gerçek bir kullanıcı şikayeti/ihtiyacı yok — YAGNI; (2) Sprint 10.5'in kapsamı zaten navigasyon olarak netleşti, bunu genişletmek "scope creep" olur; (3) Kalıcı cache zaten Sprint 9.1'in kendi notuyla gelecekteki bir ADR'ye bağlanmıştı. **Bu bulguyu Bölüm 6'da "yapılmayacaklar" listesine, Bölüm 7'nin sonunda ise "gelecek sprint önerisi" olarak kaydediyorum.**

---

## 2. Route Wrapper Tasarımı — Neden `useTreeForRoute` Deseni

**Neden aynı deseni kullanacağım:** `useTreeForRoute`, route parametresinden bir repository nesnesini asenkron yükleme, `"loading"`/`null` (bulunamadı)/gerçek nesne durumlarını yönetme, ve **race condition koruması** (`cancelled` flag'i — component unmount olduktan sonra `setState` çağrılmasını engelliyor) problemini **zaten çözmüş, test edilmiş, production'da çalışan** bir desen. Photo için aynı problemi sıfırdan çözmek yerine bu deseni taklit etmek, hem güvenilirlik miras alır hem kod okunabilirliğini korur (aynı problem, aynı çözüm şekli — geliştiriciler arası tanıdıklık).

**Ek bir gerçek detay (bu analizde yeni bulundu):** `useTreeForRoute`, ayrıca `useBackButtonFallback` hook'unu da kullanıyor — `"loading"` durumundayken donanım geri tuşunun nasıl davranacağını yönetiyor. Route wrapper tasarımı bunu da taklit etmeli (Bölüm 7'nin adımlarına dahil edildi).

**Neden yeni bir mimari (genel `usePhotoForRoute` hook'u) oluşturmuyorum:** `useTreeForRoute`'un kendisi, Engineering Protocol Bölüm 21'in **"3. tekrar eşiği"** ilkesinin sonucuydu — Sprint 5.3'te 2. tekrarda bilinçli olarak ertelenmiş, Sprint 7.1'de 3. tekrarda soyutlanmıştı. Photo için bu **ilk kullanım** olacak — ayrı, genel bir soyutlama oluşturmak, henüz kanıtlanmamış bir "tekrar edilecek" varsayımı olurdu. En doğru yaklaşım: `PhotoAnalysisScreenRoute`'un kendi içinde, `useTreeForRoute`'un desenini **taklit eden inline bir `useState`+`useEffect`** yazmak — genel bir hook oluşturmadan.

**Neden Engineering Protocol ile uyumlu:** Protocol Bölüm 21'in kendi metni bu kararı doğrudan destekliyor: "gelecekte 2. veya 3. bir route wrapper da benzer bir doğrudan repository erişimine ihtiyaç duyarsa, bu artık bir desen haline gelir ve o zaman gerçek bir soyutlama değerlendirilmelidir." Photo, 1. kullanım — inline yeterli, soyutlama erken olur.

---

## 3. UI/UX Değerlendirmesi

| Konu | Öneri | Gerekçe |
|---|---|---|
| **Konum** | `PhotoListItem` içinde, "Sil" butonundan **önce** | Yıkıcı olmayan bir eylem, yıkıcı olan "Sil"den görsel olarak ayrı dursun — arada bir buton olması, yanlışlıkla "Sil"e basma riskini de dolaylı olarak azaltır |
| **Görünüm** | Mevcut `lock-screen__button` deseni, hafif bir vurgu (`border: 2px solid var(--color-primary)`) | Dashboard/Toplu İşlemler'de kullanılan "bu bir AI/öne çıkan özellik" vurgu deseniyle tutarlı |
| **İkon seçimi** | **İkon kullanılmamalı, sadece metin** ("AI ile Analiz Et") | Proje genelinde (lucide-react kütüphanesi mevcut olsa da) hiçbir yerde ikon kullanılmıyor — tutarlılık için yeni bir görsel dil icat edilmemeli |
| **Mobil kullanım kolaylığı** | `min-height: 52px` standardı korunmalı | Eldivenli/güneş altı kullanım için zaten kurulu proje standardı |
| **Yanlışlıkla basılmasını önleme** | Ek bir onay diyaloğu **gerekmiyor** | "Analiz Et" veri silmiyor, sadece bir API çağrısı yapıyor — "Sil"in aksine yıkıcı değil, onay diyaloğu gereksiz sürtünme yaratır |
| **Erişilebilirlik** | Ek `aria-label` gerekmiyor | Buton metni (i18n ile) zaten kendi başına açıklayıcı, mevcut buton desenleriyle (sadece metin içeriği) tutarlı |

---

## 4. Performans Değerlendirmesi

| Soru | Cevap |
|---|---|
| Gereksiz render oluşturur mu? | Hayır, **gerekli** bir render — `useTreeForRoute` deseninde loading→hazır iki aşamalı render, kullanıcıya geri bildirim vermek için normal ve beklenen bir React davranışı. |
| Loading ekranı gerekli mi? | **Evet** — `useTreeForRoute`'un tüm emsalleri (TreeHarvestScreenRoute, TreeAiChatScreenRoute vb.) zaten "loading" durumunda `t("common.loading")` gösteriyor, established pattern. |
| Skeleton gerekli mi? | **Hayır** — proje genelinde hiçbir yerde skeleton (iskelet animasyonu) kullanılmıyor, sadece basit "Yükleniyor..." metni. Yeni bir görsel dil icat etmemek için eklenmemeli. |
| Mevcut desen yeterli mi? | **Evet** — `photoRepository.getById()`, `id` üzerinden (PRIMARY KEY) tek bir basit SQLite sorgusu, milisaniyeler içinde tamamlanır. Dashboard'ın Sprint 8.4'teki ölçümleri bu tür tekli sorguların çok hızlı olduğunu zaten kanıtladı. |

---

## 5. Test Planı

| Test Adı (Taslak) | Amaç | Doğrulanacak Davranış |
|---|---|---|
| `PhotoAnalysisScreenRoute — geçerli photoId ile önce 'loading' sonra ekran gösterir` | Route wrapper'ın asenkron yükleme akışı | `"loading"` → `Photo` geçişinin gerçekten çalıştığı |
| `PhotoAnalysisScreenRoute — geçersiz/olmayan photoId ile güvenli bir rotaya yönlendirir` | Hata durumu (silinmiş/geçersiz fotoğraf) | `useTreeForRoute`'un `null` durumu emsaliyle tutarlı bir `Navigate` fallback |
| `PhotoAnalysisScreenRoute — photoId undefined ise de aynı fallback'e gider` | URL manipülasyonu/eksik parametre koruması | Route parametresi eksikse çökme değil, güvenli yönlendirme |
| `PhotoGalleryScreen — 'AI ile Analiz Et' butonu HER fotoğraf satırında görünür` | UI girişinin varlığı | Buton, listedeki her `PhotoListItem`'da render ediliyor |
| `PhotoGalleryScreen — butona tıklamak onAnalyze'ı DOĞRU fotoğraf nesnesiyle çağırır` | Callback doğruluğu | Yanlış fotoğrafın analiz edilmediğinin kanıtı |
| `AppRouter — PhotoGalleryScreen'den 'Analiz Et'e tıklamak GERÇEK navigasyonla PhotoAnalysisScreen'e gider` | Uçtan uca gerçek navigasyon | Doğru URL/hash, doğru fotoğrafın gösterildiği |
| `AppRouter — PhotoAnalysisScreen'den geri dönüş PhotoGalleryScreen'e doğru şekilde gider` | Geri navigasyon (buton + donanım geri tuşu) | Hem görünür buton hem `backButtonListeners` senaryosu |

**Not:** `i18n/keySymmetry.test.ts` zaten mevcut ve otomatik — yeni i18n anahtarları eklendiğinde ayrıca yeni bir test yazmaya gerek yok, mevcut test bunları otomatik kapsıyor.

---

## 6. Sprint Kapsamı — Kesinlikle Yapılmayacaklar (Scope Creep Riski)

| Madde | Neden Dahil Edilmiyor |
|---|---|
| ❌ Session veya kalıcı cache eklemek | Bölüm 1'in sonucu — henüz kanıtlanmış bir ihtiyaç yok, ayrı bir kapsam genişlemesi olur |
| ❌ Teşhis/tedavi önerisi eklemek | Sprint 9.2'nin kendi yasağı, hâlâ geçerli |
| ❌ Karşılaştırmalı fotoğraf analizi (Zaman Çizelgesi) | Roadmap'in kendisi bunu ayrı, gelecekteki bir özellik olarak işaretlemişti |
| ❌ Genel `usePhotoForRoute` soyutlaması oluşturmak | Bölüm 2'nin gerekçesi — 1. kullanımda inline yeterli, erken soyutlama |
| ❌ İkon kütüphanesi (lucide-react) entegre etmek | Bölüm 3'ün kararı — proje genelinde tutarlılık için sadece metin |
| ❌ Skeleton loading UI eklemek | Bölüm 4'ün kararı — mevcut basit "Yükleniyor..." deseni yeterli |
| ❌ AI analiz sonucunu kalıcı olarak saklamak | Sprint 9.1/9.2'nin bilinçli kararı, değişmiyor |
| ❌ Fotoğraf galerisinin bilinen teknik borcunu (thumbnail üretimi) bu sprintte çözmek | Modül 3'ün ayrı, kapsam dışı bir teknik borcu |
| ❌ Tekli Bakım Kaydı formuna ilgisiz bir Sprint 10.4 özelliği taşımak | İlgisiz, karıştırılmamalı |

---

## 7. Son Karar — Sprint 10.5 Nihai Teknik Planı

| # | Adım | Neden Gerekli |
|---|---|---|
| 1 | `routes.ts`'e `photoAnalysis: "/photos/:photoId/analysis"` deseni + `buildPath.photoAnalysis(photoId)` ekle | Diğer tüm ekranların (established pattern) ayrı bir rotaya sahip olmasıyla tutarlılık |
| 2 | `AppRouter.tsx`'e `PhotoAnalysisScreenRoute` wrapper'ı ekle — `useParams` ile `photoId` al, `useTreeForRoute`'un desenini (race condition koruması + `useBackButtonFallback` dahil) taklit eden inline bir `useState`+`useEffect` ile `photoRepository.getById()` çağır | `PhotoAnalysisScreen`'in `photo: Photo` prop gereksinimini karşılamanın tek güvenli yolu |
| 3 | `PhotoGalleryScreen.tsx`'e `onAnalyze?: (photo: Photo) => void` prop'u + `PhotoListItem`'a buton ekle (Bölüm 3'ün UI kararlarıyla) | Kullanıcının gerçek giriş noktası |
| 4 | `AppRouter.tsx`'teki `PhotoGalleryScreenRoute`'a `onAnalyze` callback'ini (`navigate(buildPath.photoAnalysis(photo.id))`) bağla | Butonu gerçek navigasyona bağlamak |
| 5 | i18n anahtarları (EN/TR) — `photo.analyzeButton` | Globalization Policy gereği |
| 6 | Testler (Bölüm 5'teki 7 madde) | Her davranışın gerçekten çalıştığının kanıtı |
| 7 | Doğrulama: `tsc -b`, tam test suite, `lint`, `build`, `cap sync` — hepsi gerçekten çalıştırılacak | Standart teslim disiplini |
| 8 | Teslim belgeleri: BUILD_INFO güncellemesi, Sprint 10.5 Teknik Raporu, `module-status.md` güncellemesi (Modül 9'un "navigasyon tamamlandı" durumuna geçmesi) | Standart teslim disiplini |

### Risklerin Yeniden Değerlendirilmesi

- **Route wrapper'ın race condition korumasının doğru kopyalanması** — tek gerçek dikkat noktası. `cancelled` flag'i atlanırsa, component unmount olduktan sonra bir `setState` çağrısı React uyarısı/potansiyel hafıza sızıntısı riski taşır. Bu, kod yazılırken özellikle doğrulanmalı (test edilebilir bir davranış).
- Bunun dışında **genel risk düşük** kalmaya devam ediyor — tüm alt yapı (repository, provider, hook) hazır ve test edilmiş, tasarım kararları kanıtlanmış emsallere dayanıyor.

### Gelecek Sprint Önerisi (Bu Sprinte Dahil Değil — Sadece Kayıt)

Session-seviyesi (in-memory) bir analiz sonucu cache'i, gerçek bir kullanıcı şikayeti (ör. "aynı fotoğrafı yanlışlıkla iki kez analiz ettim, boşuna maliyet oluştu") ortaya çıkarsa değerlendirilebilir. Bugün için YAGNI gereği ertelendi.
