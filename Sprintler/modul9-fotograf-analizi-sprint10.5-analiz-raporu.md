# Modül 9 (Fotoğraf Analizi) — Sprint 10.5 Öncesi Analiz Raporu

**Tarih:** 2026-07-20 · **Kural:** Bu belge SADECE mevcut kod tabanının incelenmesiyle hazırlandı. Hiçbir kod yazılmadı, hiçbir dosya değiştirilmedi, hiçbir commit oluşturulmadı. Bilinçli olarak proje deposunun dışına yazıldı.

**İncelenen gerçek dosyalar:** `src/modules/photoAnalysis/*` (tamamı), `src/modules/photos/PhotoGalleryScreen.tsx`, `src/modules/photos/data/photo.repository.interface.ts`, `src/router/routes.ts`, `src/router/AppRouter.tsx`, `src/router/useTreeForRoute.ts` (emsal deseni), `src/i18n/locales/en/common.json` (`photo` namespace'i).

---

## 1. Mevcut Durum

### Roadmap İddiasının Doğrulanması

Roadmap raporundaki "Kod hazır, ancak PhotoGalleryScreen → PhotoAnalysisScreen navigasyon entegrasyonu eksik" ifadesi **gerçek kod tabanında doğrulandı — hâlâ doğru**:

- `src/router/AppRouter.tsx`'te `PhotoAnalysisScreen` **hiç import edilmiyor**.
- `src/router/routes.ts`'te `photoAnalysis` adında bir rota **hiç tanımlı değil**.
- `PhotoGalleryScreen.tsx`'teki `PhotoListItem` bileşeninde SADECE "Sil" (`photo.deleteButton`) butonu var — hiçbir "Analiz Et" butonu veya `onAnalyze`/`onViewAnalysis` callback prop'u yok.

### Tamamlanan Kısımlar (Sprint 9.1-9.2'den, Gerçek Dosyalardan Doğrulandı)

| Katman | Durum | Kanıt |
|---|---|---|
| `PhotoAnalysisScreen.tsx` | Tam, çalışır durumda | idle/analyzing/error/ready durumları, disclaimer, sonuç gösterimi |
| `usePhotoAnalysis.ts` | Tam, çalışır durumda | `getActiveAiProvider()` + `readFileAsBase64()` + `provider.analyzeImage()` zinciri |
| `photoAnalysisPrompt.ts` | Tam | Teşhis/tedavi yasaklarını modele yansıtan sistem promptu |
| `GeminiProvider.analyzeImage()` | Tam (Sprint 9.2) | Gerçek `inlineData` API şekli doğrulanmış |
| `photoRepository.getById()` | Zaten mevcut | `IPhotoRepository` sözleşmesinde tanımlı, kullanıma hazır |
| Testler | Mevcut (4 test `PhotoAnalysisScreen.test.tsx` + 5 test `usePhotoAnalysis.test.ts`) | Sprint 9.2'de yazıldı |

### Eksik Kalan Kısımlar

1. **Route tanımı** — `photoAnalysis` rotası `routes.ts`'te yok.
2. **Route wrapper** — `AppRouter.tsx`'te `PhotoAnalysisScreenRoute` yok.
3. **Giriş noktası UI'ı** — `PhotoGalleryScreen`'de "Analiz Et" butonu yok.
4. **i18n anahtarları** — `photo` namespace'inde `analyzeButton` gibi bir anahtar yok (gerçek dosyadan doğrulandı, `photo.*` altında sadece 10 anahtar var, analiz butonu yok).
5. **Route-seviyesi veri yükleme mekanizması** — bu, roadmap raporunda açıkça belirtilmemiş ama gerçek kod incelemesiyle ortaya çıkan bir bulgu (Bölüm 4'te detaylandırıldı).

### Roadmap ile Uyum

Roadmap'in "sadece navigasyon eksik" tespiti kabaca doğru ama teknik olarak eksik bir basitleştirme içeriyor — Bölüm 4'te açıklanan "route-seviyesi veri yükleme" ihtiyacı, roadmap raporunda ayrı bir madde olarak görünmüyor.

---

## 2. PhotoGalleryScreen ↔ PhotoAnalysisScreen Entegrasyonu — Tam Olarak Nasıl Eksik?

Üç ayrı, birbirine bağımlı eksik var:

1. **UI girişi yok:** `PhotoListItem` (Sil butonunun yanına) bir "Analiz Et" butonu eklenmeli, bu buton bir `onAnalyze(photo)` callback'ini tetiklemeli.
2. **Rota yok:** `PhotoAnalysisScreen`'e giden hiçbir URL/rota tanımlı değil.
3. **Veri köprüsü yok:** `PhotoAnalysisScreen`'in props arayüzü `photo: Photo` (tam nesne, sadece ID değil) bekliyor — bir rotaya bağlanırsa, route wrapper'ın URL'deki `photoId`'den gerçek `Photo` nesnesini asenkron olarak yüklemesi gerekiyor. Bu, basit bir "buton tıkla → git" bağlantısından fazlası.

---

## 3. Etkilenecek Dosyalar (Liste — Hiçbiri Değiştirilmedi)

| Dosya | Değişiklik Türü | Gerekçe |
|---|---|---|
| `src/router/routes.ts` | Düzenleme (ekleme) | `photoAnalysis` rota deseni + `buildPath.photoAnalysis()` |
| `src/router/AppRouter.tsx` | Düzenleme (ekleme) | `PhotoAnalysisScreen` import, yeni route wrapper, rota kaydı |
| `src/modules/photos/PhotoGalleryScreen.tsx` | Düzenleme (ekleme) | `PhotoListItem`'a buton + `onAnalyze` prop, `PhotoGalleryScreen`'in kendi props'una `onAnalyze` |
| `src/i18n/locales/en/common.json` | Düzenleme (ekleme) | `photo.analyzeButton` (ve muhtemelen ilgili anahtarlar) |
| `src/i18n/locales/tr/common.json` | Düzenleme (ekleme) | Aynı anahtarların Türkçe karşılığı |
| `src/modules/photos/PhotoGalleryScreen.test.tsx` | Düzenleme (ekleme) | Yeni buton/callback için test |
| `src/router/AppRouter.test.tsx` | Düzenleme (ekleme) | Gerçek navigasyon testi (PhotoGallery → PhotoAnalysis → geri) |

**Muhtemelen gerekecek (Bölüm 4'te gerekçelendirildi), ama henüz kesin değil:**

| Dosya | Değişiklik Türü | Gerekçe |
|---|---|---|
| Yeni bir dosya (ör. `src/router/usePhotoForRoute.ts`) VEYA `AppRouter.tsx` içinde inline `useState`/`useEffect` | Yeni | Route wrapper'ın `photoId`'den `Photo` nesnesini yüklemesi için — `useTreeForRoute.ts`'in kanıtlanmış deseni |

**Değişmeyecek olanlar (doğrulandı, gerçek bir bulgu):** `usePhotoAnalysis.ts`, `PhotoAnalysisScreen.tsx`, `photoAnalysisPrompt.ts`, `photo.repository.ts`, `GeminiProvider.ts`, `AIProvider.interface.ts` — hiçbiri değişmiyor.

---

## 4. Navigasyon Dışında Eksik Kalan Bağımlılıklar

| Kategori | Durum |
|---|---|
| Route | Eksik (Bölüm 1-3'te detaylandırıldı) |
| Hook | Muhtemelen 1 yeni küçük hook/inline mantık gerekiyor — `photoId`'den `Photo` nesnesini yüklemek için (`useTreeForRoute` emsali). `usePhotoAnalysis` kendisi değişmiyor. |
| Context | Etkilenmiyor — AI Context Engine bu akışta kullanılmıyor (fotoğraf analizi, sohbet bağlamından bağımsız çalışıyor, Sprint 9.2'nin kendi tasarımı). |
| Repository | Değişmiyor — `photoRepository.getById()` zaten var ve yeterli. |
| Provider | Değişmiyor — `GeminiProvider.analyzeImage()` zaten Sprint 9.2'de tamamlandı ve test edildi. |
| Permission | Etkilenmiyor — AI izin kontrolü (`getActiveAiProvider()`) zaten `usePhotoAnalysis`'in içinde, değişmiyor. |
| UI | Eksik — `PhotoListItem`'a buton (Bölüm 3). |
| Error Handling | Değişmiyor — `usePhotoAnalysis` zaten `mapAiError()` ile hataları çeviriyor, `PhotoAnalysisScreen` zaten hata durumunu gösteriyor. Route wrapper seviyesinde SADECE "geçersiz/bulunamayan photoId" durumu için yeni bir dal gerekebilir (`useTreeForRoute`'un `null` durumu emsali). |
| Loading | Yeni bir loading durumu gerekebilir — route wrapper, `Photo` nesnesini yüklerken bir yükleniyor göstergesi göstermeli (`useTreeForRoute` emsali). |
| State Management | Etkilenmiyor — mevcut `useState`/hook deseni yeterli, yeni bir state yönetim aracı gerekmiyor. |

---

## 5. Mimari Etki Analizi

| Soru | Cevap |
|---|---|
| Mevcut mimariyi etkiliyor mu? | Hayır, bozucu bir etki yok — sadece ekleme (yeni rota, yeni buton, route wrapper). Mevcut hiçbir dosyanın davranışı değişmiyor, sadece PhotoGalleryScreen'e yeni bir opsiyonel prop eklenecek. |
| Migration gerektiriyor mu? | Hayır — hiçbir şema değişikliği yok, veri modeli aynı kalıyor. |
| Repository değişikliği gerektiriyor mu? | Hayır — `photoRepository.getById()` zaten var ve tam olarak ihtiyacı karşılıyor. |
| Veri modeli değiştiriyor mu? | Hayır — `Photo` domain tipi hiç değişmiyor. |
| AI Provider katmanını etkiliyor mu? | Hayır — `GeminiProvider`/`AIProvider.interface.ts` hiç değişmiyor, Sprint 9.2'de tamamlandı. |

**Sonuç:** Bu, mimari olarak düşük riskli, saf bir navigasyon + UI ekleme işi. Tek gerçek karmaşıklık noktası, route wrapper'ın asenkron veri yüklemesi (Bölüm 6'da risk olarak ele alınıyor).

---

## 6. Risk Analizi

| Risk | Olasılık | Etki | Açıklama |
|---|---|---|---|
| Route wrapper tasarım kararı | Orta | Düşük-Orta | `PhotoAnalysisScreen`'in `photo: Photo` (tam nesne) beklemesi, route wrapper'ın `useTreeForRoute` benzeri bir asenkron yükleme deseni kurmasını gerektiriyor. Bu, diğer bazı route wrapper'lardan (sadece string ID geçenlerden) biraz daha karmaşık — ama kanıtlanmış bir emsal var (`useTreeForRoute.ts`), riski düşürüyor. |
| "3. tekrar" eşiği sorusu | Düşük | Düşük | `useTreeForRoute`, Sprint 7.1'de "3. tekrar" eşiğinde soyutlanmıştı (Engineering Protocol Bölüm 21). Photo için bu ilk kullanım olacağı için, ayrı bir `usePhotoForRoute` hook'u yazmak yerine route wrapper içinde inline bir `useState`/`useEffect` yazmak da savunulabilir — bu, geliştirme sırasında netleştirilmesi gereken küçük bir tasarım kararı. |
| PhotoGalleryScreen'in mevcut testlerinin bozulması | Düşük | Düşük | Yeni bir opsiyonel prop eklemek (`onAnalyze`), mevcut testleri büyük ihtimalle bozmaz — ama proaktif test-dosyası tip kontrolü (established pattern) gerekecek. |
| Kalıcı saklama olmadığı için "hangi fotoğraf analiz edildi" bilgisinin kaybolması | Bilgi amaçlı, risk değil | — | Sprint 9.2'nin bilinçli kararı (necessity analizi) — bu sprintte de değişmiyor, sadece hatırlatma. |
| AI kapalıyken kullanıcı deneyimi | Düşük | Düşük | Zaten çözülmüş — `usePhotoAnalysis`, AI kapalıysa `AI_001` hatasını döner, `PhotoAnalysisScreen` bunu gösterir. Test edilmiş bir davranış. |

**Genel risk seviyesi: Düşük.** En büyük belirsizlik, route wrapper'ın veri yükleme deseni tercihidir — ama bunun için gerçek bir kanıtlanmış emsal (`useTreeForRoute`) zaten mevcut.

---

## 7. Tahmini Sprint 10.5 Kapsamı

**Roadmap raporunda görünmeyen ek iş bulundu mu? Evet, bir tane — ama küçük ve roadmap'in "navigasyon" şemsiyesinin doğal bir parçası:**

Sprint 10.5, sadece "PhotoGalleryScreen'e bir buton ekle ve routes.ts'e bir satır ekle" ile tamamlanamaz — çünkü `PhotoAnalysisScreen`'in props arayüzü tam bir `Photo` nesnesi istiyor, bu da route wrapper seviyesinde bir asenkron veri yükleme adımı gerektiriyor. Bu, yeni bir modül veya kapsam genişlemesi değil — sadece roadmap'in "navigasyon entegrasyonu" ifadesinin, teknik olarak "route + route-seviyesi veri yükleme + UI girişi" üçlüsünü kapsadığının netleştirilmesi.

**Bu iş, tamamen mevcut, kanıtlanmış desenlerin (`useTreeForRoute` emsali) tekrarıdır — yeni bir mimari kategori değildir.**

### Somut Kapsam Maddeleri

1. `routes.ts`'e `photoAnalysis` rota deseni + `buildPath` fonksiyonu.
2. Route wrapper (`PhotoAnalysisScreenRoute`) — `photoId`'den `Photo` nesnesini yükleyen mantık (yeni küçük hook veya inline).
3. `PhotoGalleryScreen`'e "Analiz Et" butonu + `onAnalyze` callback prop'u.
4. i18n anahtarları (EN/TR).
5. Testler: buton/callback testi, route wrapper'ın loading/not-found durumları, gerçek uçtan uca navigasyon testi (`AppRouter.test.tsx`).
6. Standart teslimatlar: BUILD_INFO, teknik rapor, module-status güncellemesi.

---

## 8. Sprint Sonunda Oluşacak Kullanıcı Deneyimi

Fotoğraf çekildikten sonra kullanıcının geçeceği adımlar (Sprint 10.5 tamamlandığında):

1. Kullanıcı bir Gözlem'in Fotoğraflar ekranında (`PhotoGalleryScreen`), daha önce çekilmiş/kaydedilmiş bir fotoğrafın yanında artık bir "AI ile Analiz Et" butonu görür.
2. Butona basar → `PhotoAnalysisScreen`'e yönlendirilir (yeni bir rota/ekran).
3. Ekran açılırken kısa bir yükleniyor durumu görülebilir (fotoğraf verisi route parametresinden yüklenirken).
4. Fotoğraf büyük gösterilir, altında sabit bir uyarı metni ("Bu genel bir AI gözlemidir, profesyonel bir teşhis değildir...").
5. Kullanıcı "Analiz Et" butonuna basar (bu, Sprint 9.2'den beri var olan, henüz tetiklenmemiş adım).
6. Analiz sırasında bir yükleniyor göstergesi (spinner + "Analiz ediliyor...") görünür.
7. Sonuç geldiğinde, AI'nin serbest metin gözlemi bir kart içinde gösterilir (teşhis veya tedavi önerisi yok, sadece gözlemsel bir açıklama).
8. Eğer AI kapalıysa veya bir hata oluşursa, kullanıcı çevrilmiş, anlaşılır bir hata mesajı görür (ham teknik hata değil).
9. Kullanıcı "Geri" butonuyla Fotoğraflar ekranına döner — analiz sonucu kalıcı olarak saklanmaz, ekrandan çıkınca kaybolur (Sprint 9.2'nin bilinçli kararı, bu sprintte değişmiyor).

---

## Rapor Özeti (İstenen Format)

### 1. Mevcut Durum
Modül 9'un kod katmanı (Provider/Hook/Screen/Prompt) Sprint 9.2'de tamamlanmış ve test edilmiş durumda. Navigasyon katmanı (route, route wrapper, giriş butonu) hiç kurulmamış — roadmap iddiası doğrulandı.

### 2. Eksikler
Route tanımı, route wrapper, `PhotoGalleryScreen`'e giriş butonu, i18n anahtarları, ve (roadmap'te açıkça görünmeyen ama gerekli) route-seviyesi asenkron veri yükleme mekanizması.

### 3. Etkilenecek Dosyalar
7 kesin dosya (Bölüm 3'teki tablo) + muhtemelen 1 yeni küçük dosya/inline mantık (Photo route veri yükleme).

### 4. Risk Analizi
Genel risk düşük — tüm ihtiyaç duyulan alt yapı (repository, provider, hook) zaten var ve test edilmiş. Tek belirsizlik, route wrapper'ın veri yükleme deseni tercihi — ama kanıtlanmış bir emsal (`useTreeForRoute`) mevcut.

### 5. Sprint 10.5 Kapsamı
Navigasyon + route-seviyesi veri yükleme + UI girişi — üçü birlikte "navigasyon entegrasyonu" şemsiyesi altında, yeni bir mimari kategori veya modül değil.

### 6. Tahmini İş Yükü
Küçük-orta ölçekli bir sprint — Sprint 8.3 (Hasat navigasyon entegrasyonu) veya Sprint 8.5 (Dashboard navigasyon entegrasyonu) ile karşılaştırılabilir büyüklükte, çünkü onlar da benzer route wrapper + buton + test desenini izlemişti.

### 7. Sprint 10.5'e Başlanabilir mi?
Evet. Hiçbir bloke edici bağımlılık yok, hiçbir eksik alt yapı yok, mimari risk düşük. Roadmap'in "sadece navigasyon" ifadesi teknik olarak biraz eksik (route-seviyesi veri yükleme adımını açıkça belirtmiyor) ama bu, kapsamı büyütmüyor — sadece "navigasyon" kelimesinin teknik karşılığını netleştiriyor. Sprint 10.5, kanıtlanmış desenlerin (route + wrapper + buton, Hasat/Dashboard emsali) tekrarıyla düşük riskle tamamlanabilir.
