# Bahçem Mobile — AI Modülü Teknik Denetim Raporu (AI X-Ray)

**Tarih:** 2026-07-21 · **Kural:** Bu belge SADECE mevcut kod tabanının incelenmesiyle hazırlandı. Hiçbir kod yazılmadı, hiçbir dosya değiştirilmedi, hiçbir refactor/özellik eklenmedi. Emin olunmayan hiçbir şey kesin bilgi gibi sunulmadı — her iddia gerçek dosya alıntısı veya (Bölüm 5'te) resmi/güncel dış kaynakla desteklendi.

---

## 1. AI Mimarisi

### Dosya Envanteri (Gerçek, Tam Liste)

```
src/modules/ai/
├── AiChatScreen.tsx / .test.tsx
├── AiSettingsScreen.tsx / .test.tsx
├── bootstrapAi.ts / .test.ts
├── context/
│   ├── IContextEngine.interface.ts
│   └── KeywordContextEngine.ts / .test.ts
├── data/
│   ├── aiConversation.repository.ts / .interface.ts / .test.ts
│   └── aiSettings.repository.ts / .test.ts
├── domain/ai.types.ts
├── hooks/
│   ├── useAiChat.ts / .test.ts
│   └── useAiSettings.ts / .test.ts
├── prompt/
│   ├── promptBuilder.ts
│   ├── systemPrompt.ts
│   ├── contextPromptSection.ts
│   ├── toolPromptSection.ts
│   ├── userPromptSection.ts
│   └── prompt.test.ts
├── providers/
│   ├── AIProvider.interface.ts
│   ├── GeminiProvider.ts / .test.ts
│   └── ProviderRegistry.ts / .test.ts
├── session/
│   ├── AiSessionService.ts / .test.ts
│   └── getActiveAiProvider.ts / .test.ts
└── tools/
    ├── ToolDefinition.interface.ts
    ├── ToolRegistry.ts / .test.ts
    ├── registerReadOnlyTools.ts / .test.ts
    ├── parcel.tool.ts, tree.tool.ts, observation.tool.ts, maintenance.tool.ts, finance.tool.ts
    └── readOnlyTools.test.ts

src/modules/photoAnalysis/
├── PhotoAnalysisScreen.tsx / .test.tsx
├── hooks/usePhotoAnalysis.ts / .test.ts
└── photoAnalysisPrompt.ts
```

### İstenen Veri Akışının Gerçek Karşılığı

Senin şemanı gerçek koda eşlersem:

| Katman | Gerçek Dosya | Sorumluluk |
|---|---|---|
| Kullanıcı Girişi | PhotoGalleryScreen.tsx (buton) / AiChatScreen.tsx (metin kutusu) | Kullanıcının fotoğraf seçmesi veya soru yazması |
| Fotoğraf/Kamera | native/filesystem.ts (readFileAsBase64) | Kalıcı dosyayı base64'e okur (fotoğraf akışı için) |
| İzin Kontrolü | session/getActiveAiProvider.ts | AI açık mı, internet izni var mı, provider kayıtlı mı — her iki akışta da (sohbet + fotoğraf) ortak |
| Prompt Oluşturma | prompt/promptBuilder.ts + prompt/systemPrompt.ts (sohbet) / photoAnalysis/photoAnalysisPrompt.ts (fotoğraf) | Sistem talimatı + kullanıcı turu metnini modüler olarak birleştirir |
| Context Engine | context/KeywordContextEngine.ts | Sıfır AI çağrısı ile, ekran bağlamından ilgili veriyi metne çevirir — sadece sohbet akışında var |
| Tool Registry | tools/ToolRegistry.ts + 5 salt-okunur araç | Modelin çağırabileceği fonksiyonları tanımlar — sadece sohbet akışında var |
| Oturum Yönetimi | session/AiSessionService.ts (sohbet) / photoAnalysis/hooks/usePhotoAnalysis.ts (fotoğraf) | Tüm adımları sırayla yürütür, DB'ye kaydeder |
| Gemini Servisi | providers/GeminiProvider.ts | @google/genai SDK'sını sarar, retry mantığı içerir |
| API İsteği | GoogleGenAI.models.generateContent() (GeminiProvider içinde) | Gerçek HTTPS isteği |
| Yanıt İşleme | GeminiProvider.ts içindeki buildContents/dönüş değeri işleme | Ham API yanıtını AIProviderResponse'a çevirir |
| Hata Eşleme | core/errors/mapAiError.ts | Ham hataları kullanıcıya gösterilecek hata koduna çevirir |
| UI Gösterimi | AiChatScreen.tsx / PhotoAnalysisScreen.tsx | Sonucu/hatayı/yükleniyor durumunu gösterir |

Not: Senin şemandaki "Fotoğraf/Kamera → Prompt" ve "Serbest Soru → Prompt" iki ayrı, paralel akıştır — ortak olan tek katman getActiveAiProvider() (izin kontrolü) ve GeminiProvider (Gemini istemcisi). Context Engine ve Tool Registry sadece sohbet akışında var, fotoğraf akışında hiç kullanılmıyor.

---

## 2. Mevcut Durum — Kullanıcı Bugün Neler Yapabiliyor?

| Özellik | Durum | Kanıt |
|---|---|---|
| Serbest soru sorma (genel AI sohbeti) | Tamamlandı | AiChatScreen.tsx, /ai/chat rotası, gerçek Gemini çağrısı |
| Parsel bağlamlı soru sorma | Tamamlandı | /parcels/:parcelId/ai rotası, ParcelAiChatScreenRoute |
| Fotoğraf analizi (genel gözlemsel) | Tamamlandı | PhotoAnalysisScreen.tsx, Sprint 10.5'te navigasyona bağlandı |
| Uygulama verisi hakkında soru (parsel/ağaç/gözlem/bakım/finans sayıları) | Tamamlandı (araç çağırma ile) | 5 salt-okunur araç kayıtlı |
| Bitki tanıma (tür/çeşit tespiti) | Yok | Hiçbir kodda "tür tanıma" mantığı bulunamadı |
| Hastalık teşhisi | Yok (bilinçli olarak yasaklanmış) | photoAnalysisPrompt.ts: "Do NOT provide a definitive diagnosis" |
| Tedavi/ilaç/gübre önerisi | Yok (bilinçli olarak yasaklanmış) | Aynı dosya: "Do NOT recommend any treatment" |
| Önceki analizleri görüntüleme (geçmiş) | Yok | Fotoğraf analiz sonucu kalıcı saklanmıyor (bilinçli Sprint 9.1 kararı) |
| Sohbet geçmişine devam etme | Yok (bilinçli olarak ertelenmiş) | useAiChat.ts: "her ekran açılışında YENİ bir konuşma... YAGNI, henüz istenmedi" |
| Sesli soru/cevap | Yok | Hiçbir ses ile ilgili kod yok |
| Karşılaştırmalı fotoğraf analizi | Yok (bilinçli olarak kapsam dışı) | Sprint 9.2 teknik raporu, Öncelik 10 |

---

## 3. AI Modülünün Eksikleri

| Özellik | Durum |
|---|---|
| Hastalık/zararlı teşhisi | Planlandı değil — bilinçli olarak yasaklanmış |
| Tedavi/gübre/ilaç/sulama önerisi | Planlandı değil — aynı gerekçe |
| Bitki/tür tanıma | Eksik — hiç tasarlanmamış |
| Geçmiş analiz karşılaştırması | Kısmen hazır — Sprint 9.1'in tasarım belgesinde 3 seçenek değerlendirilmiş, hiçbiri uygulanmamış |
| Sesli asistan | Planlandı — roadmap'te ayrı bir modül, hiç başlamamış |
| RAG/Embedding (uzun süreli hafıza) | Planlandı — AiSessionService'in kendi yorumu: "Memory (ai_facts) arayüzde yer ayrılmış ama yazılmamış" |
| Model/sağlayıcı seçimi (kullanıcıya) | Eksik — AiSettingsScreen'in kendi yorumu: "Model seçimi bugün kullanıcıya sunulmuyor", sabit gemini-2.5-flash |
| Hasat verisi için AI aracı | Eksik — 5 araçtan hiçbiri Hasat modülünü kapsamıyor |

---

## 4. API Entegrasyonu Denetimi

| Kontrol Noktası | Değerlendirme | Kanıt |
|---|---|---|
| API Key okunuyor mu? | Sağlam | secureStorage.get(GEMINI_API_KEY), her çağrıda taze okunuyor |
| Android Keystore | Sağlam | @aparajita/capacitor-secure-storage, Android KeyStore ile şifreleniyor |
| Environment değişkenleri | Riskli/Farklı model | process.env kullanılmıyor — API anahtarı runtime'da kullanıcı tarafından giriliyor, build-time değil |
| Build süreci | Sağlam | npm run build bu oturumda defalarca gerçekten çalıştırılıp doğrulandı |
| Gradle | Riskli (bu ortamda doğrulanamadı) | ./gradlew assembleDebug denendi, network kısıtları nedeniyle indirilemedi |
| Manifest | Sağlam | INTERNET izni gerçekten tanımlı |
| Network | Sağlam (teorik) | Gemini API HTTPS üzerinden çalışıyor |
| HTTP isteği | Sağlam | @google/genai SDK'sının generateContent() metodu kullanılıyor |
| Model adı | Sağlam | gemini-2.5-flash — geçerli format |
| Prompt oluşturma | Sağlam | Modüler yapı, test edilmiş |
| JSON Parse | Sağlam | SDK'nın kendisi bunu yapıyor |
| Response işleme | Hatalı (kesin kanıtlı) | Bölüm 5'te detaylandırıldı — tool-calling akışında Gemini'nin API sözleşmesi ihlal ediliyor |
| Exception yönetimi | Sağlam (genel olarak) | mapAiError() tüm hataları sınıflandırıyor, ama Bölüm 5'in bulgusunu da genel SYS_001'e düşürüyor |
| UI State | Riskli | Bootstrap'ın asenkron yapısı, teorik bir race condition penceresi açıyor |

---

## 5. Gerçek Kullanımda Yaşanan AI Hatası — Kanıta Dayalı Değerlendirme

### En Güçlü Bulgu: Tool-Calling Akışında Gemini API Sözleşme İhlali

Kanıt Var.

AiSessionService.ts'in gerçek kodu incelendi: araç çağrısı gerektiren bir soru sorulduğunda (2. round-trip), provider.sendMessage(providerMessages, { pendingToolResults: toolResults }) çağrılıyor — ama providerMessages dizisine, modelin kendi function-call içeren ilk yanıtı hiç eklenmiyor.

GeminiProvider.ts'in buildContents() fonksiyonu da bunu doğruluyor: pendingToolResults varsa sadece { role: "user", parts: [functionResponse...] } ekleniyor — önceki { role: "model", parts: [functionCall...] } turu hiç yok.

Resmi Google dokümantasyonu (ai.google.dev/gemini-api/docs/generate-content/function-calling, bu denetim sırasında gerçekten arandı) şu örneği veriyor:
```
history = [
  { role: "user", parts: [...] },
  response1.candidates[0].content,   // modelin function-call içeren yanıtı, history'ye EKLENİYOR
  { role: "user", parts: [{ functionResponse: {...} }] }
]
```

Güncel, bağımsız bir 3. parti hata raporu (LiteLLM projesi, GitHub Issue #26755, arama sonucunda bulundu) bunu kesinleştiriyor: "A functionResponse part must appear in a user turn that comes immediately after a model turn containing the matching functionCall... When these rules are violated... Gemini generateContent returns 400."

Sonuç: Kullanıcı, araç çağrısı gerektiren bir soru sorduğunda (ör. "kaç parselim var" gibi uygulama verisine dayalı sorular), Gemini API'sinin 400 Bad Request döndürmesi API sözleşmesi gereği kaçınılmazdır. Bu, GeminiProvider.ts'in kendi isRetryableGeminiError() fonksiyonunda '"code":400' yeniden denenmeyen bir hata olarak zaten işaretlenmiş — yani hata anında, tekrar denenmeden kullanıcıya "Something went wrong" (SYS_001) olarak düşer.

Bu, kullanıcının "AI ekranından soru sorduğunda analiz başarılı tamamlanmadı, uygulama hata verdi" gözlemiyle tam olarak örtüşen, kesin kanıtlı bir kök neden adayıdır — özellikle soru, uygulama verisiyle ilgiliyse.

### Diğer 16 İhtimalin Tek Tek Değerlendirilmesi

| İhtimal | Değerlendirme |
|---|---|
| API Key yanlış | Kontrol Edilmeli — kod seviyesinde bir doğrulama yok |
| API Key okunmuyor | Kanıt Yok — secureStorage.get() çağrısı doğru anahtar adıyla yapılıyor |
| Android build içerisine aktarılmıyor | Kanıt Yok — API anahtarı build-time gömülü değil, runtime'da okunuyor |
| Environment okunmuyor | Kanıt Yok (kavram uygulanamaz) — bu proje process.env kullanmıyor |
| Android Manifest problemi | Kanıt Yok — INTERNET izni gerçekten tanımlı |
| Gradle problemi | Kontrol Edilmeli — bu ortamda Gradle çalıştırılamadı |
| HTTP isteği başarısız | Kontrol Edilmeli — gerçek cihazda ağ sorunları teorik olarak mümkün |
| Gemini API cevap formatı değişmiş | Kanıt Yok — response.text/functionCalls kullanımı güncel tip tanımlarıyla tutarlı |
| Model adı yanlış | Kanıt Yok — geçerli format, ama gerçek erişilebilirlik test edilemedi |
| Prompt oluşturulurken hata oluşuyor | Kanıt Yok — test edilmiş, null/undefined durumları ele alınmış |
| JSON Parse hatası | Kanıt Yok — SDK bunu yönetiyor |
| Timeout | Kontrol Edilmeli — açık bir timeout değeri yok |
| Rate Limit | Kontrol Edilmeli — kod doğru tanıyor ama gerçek kullanım bilinmiyor |
| Android Network Security | Kanıt Yok — varsayılan yapılandırma HTTPS'i engellemez |
| Exception yönetimi | Kanıt Yok (genel olarak sağlam) — ama teşhisi zorlaştıran bir yan etkisi var |
| UI State yönetimi | Kontrol Edilmeli — bootstrap'ın asenkron oluşu, teorik bir race condition |
| Sonucun ekrana basılması sırasında hata | Kanıt Yok — render mantığı basit, hata durumları ayrı ele alınmış |

### En Olası 5 Neden (Olasılık Sırasına Göre)

1. Tool-calling akışında Gemini API sözleşme ihlali (400 hatası) — kesin kod kanıtı + resmi dokümantasyon + güncel 3. parti hata raporu ile desteklenen, en güçlü aday.
2. API Key geçersiz/hatalı girilmiş — kontrol edilmeli.
3. Rate limit / kota aşımı — kod doğru tanıyor ama gerçek kullanım sıklığı bilinmiyor.
4. Bootstrap race condition — teorik, düşük olasılık ama sıfır değil.
5. Ağ/timeout sorunları — kontrol edilmeli, kod seviyesinde kanıt yok.

---

## 6. Sprint Analizi

| Durum | Sprintler |
|---|---|
| Tamamlanan | Sprint 6, 7.1-7.5 (AI Altyapısı), Sprint 9.1-9.2 (Fotoğraf Analizi), Sprint 10.5 (navigasyon) |
| Devam Eden | Yok — şu an aktif bir AI sprinti yok |
| Henüz Başlamamış | Sesli Asistan, RAG/Embedding, Bitki Tanıma, Hastalık Teşhisi, Tedavi Önerileri, Karşılaştırmalı Fotoğraf Analizi, Model Seçimi UI'ı |

AI geliştirme sürecinin aşaması: Temel altyapı tamamlanmış ve iyi mimarili, ama Production Ready değil — module-status.md'nin kendi notu: "Gerçek cihaz TAM doğrulaması bekliyor" ve bu denetimde bulunan tool-calling hatası, bunun neden hâlâ Production Ready olmadığının somut bir kanıtı olabilir.

---

## 7. Final Hedefi — Bugünkü Durum ile Vizyon Arasındaki Fark

1. Fotoğraf analizi (genel gözlem) — Bugün var
2. Bitki tanıma — Yok
3. Hastalık teşhisi — Bilinçli olarak yasaklanmış
4. Zararlı tespiti — Yok
5. Gübre/ilaç/sulama önerileri — Bilinçli olarak yasaklanmış
6. Budama önerileri — Yok
7. Hava durumu yorumlama — Hava Durumu modülü hiç başlamadı
8. Geçmiş analiz karşılaştırması — Tasarlandı, uygulanmadı
9. Ağaç geçmişini yorumlama — Kısmen (araçlarla veri okunabiliyor)
10. Parsel bazlı analiz — Kısmen
11. AI sohbet asistanı — Bugün var (temel haliyle)
12. Üretim/verim/risk önerileri — Yok

Temel fark: Bugünkü sistem, "veriye erişebilen ve genel gözlem yapabilen bir asistan" seviyesinde. Vizyon ise "tanı koyan, öneri üreten, geçmişi yorumlayan bir uzman sistem". Aradaki fark, hem yeni özellik geliştirme hem de bilinçli olarak konulmuş güvenlik sınırlarının gelecekte yeniden değerlendirilmesini gerektiriyor.

---

## 8. Teknik Borç (Öncelik Sırasına Göre)

1. Tool-calling API sözleşme ihlali (Bölüm 5) — en yüksek öncelik.
2. Hata mesajlarının aşırı genelleştirilmesi — mapAiError()'ın SYS_001 fallback'i gerçek Gemini hata kodlarını ayırt etmiyor.
3. Bootstrap race condition potansiyeli.
4. Hasat modülünün AI araçlarına entegre edilmemiş olması.
5. Model seçimi kullanıcıya sunulmuyor.
6. Sohbet geçmişine devam etme yok.

---

## 9. Production Hazırlık Analizi

| Öncelik | İş |
|---|---|
| Yüksek | Tool-calling akışını Gemini'nin gerçek API sözleşmesine uygun hale getirmek |
| Yüksek | Gerçek cihazda, araç çağırma gerektiren gerçek sorularla kapsamlı test |
| Yüksek | mapAiError()'ın Gemini'ye özgü hata kodlarını ayırt etmesi |
| Orta | Bootstrap'ın senkronizasyonunu güçlendirmek |
| Orta | Hasat aracının Tool Registry'ye eklenmesi |
| Düşük | Model seçimi UI'ı |
| Düşük | Sohbet geçmişine devam etme |

---

## 10. Genel Değerlendirme Tablosu

Not: Bu yüzdeler, kod tabanının ne kadarının yazıldığına dair bir değerlendirmedir — "gerçek cihazda kusursuz çalışıyor" anlamına gelmez.

| Alan | Durum (%) |
|---|---|
| AI Altyapısı | 90 |
| Gemini Entegrasyonu | 75 |
| Prompt Sistemi | 90 |
| Fotoğraf Analizi | 85 |
| Bitki Tanıma | 0 |
| Hastalık Analizi | 0 (bilinçli olarak yasaklı) |
| AI Sohbeti | 70 |
| AI UI | 85 |
| AI Testleri | 80 |
| Hata Yönetimi | 60 |
| Production Hazırlığı | 50 |
| Genel AI Tamamlanma Oranı | 65 |

---

## 11. Sonuç

### 1. Bugün AI modülü gerçekten hangi seviyede?

Mimari olarak olgun ve iyi tasarlanmış bir temel altyapı var — ama bu denetim, çalışan kodun içinde gerçek, kanıtlanabilir bir hata buldu (Bölüm 5). Bu, "kod teorik olarak doğru görünüyor ama pratikte çalışmıyor" durumunun somut bir örneği.

### 2. AI ekranında yaşadığımız hata büyük olasılıkla neden kaynaklanıyor?

En güçlü aday: Tool-calling akışında, AiSessionService'in Gemini'ye gönderdiği mesaj geçmişi, Gemini API'sinin resmi sözleşmesini ihlal ediyor — modelin kendi function-call yanıtı history'ye hiç eklenmiyor. Resmi Google dokümantasyonu ve güncel bir bağımsız hata raporu, bu ihlalin 400 Bad Request ile sonuçlandığını kanıtlıyor.

### 3. AI modülünü final vizyonuna ulaştırmak için bundan sonraki en doğru geliştirme sırası nedir?

1. Önce Bölüm 5'in bulgusunu kesin olarak doğrulamak (gerçek cihazda, araç çağırma gerektiren bir soru sorarak).
2. Doğrulanırsa, tool-calling akışını düzeltmek.
3. mapAiError()'ı genişletip gerçek Gemini hata kodlarını ayırt eder hale getirmek.
4. Ancak bu ikisi tamamlandıktan sonra yeni özellik geliştirmeye geçmek.
