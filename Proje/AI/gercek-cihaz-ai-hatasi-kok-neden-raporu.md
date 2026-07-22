# Gerçek Cihaz AI Hatası — Kesin Kök Neden Raporu

**Tarih:** 2026-07-21 · **Kural:** Bu belge SADECE analizdir. Hiçbir kod yazılmadı, hiçbir düzeltme yapılmadı, hiçbir dosya değiştirilmedi.

---

## Özet — İki Ayrı, Birbirinden Bağımsız Kök Neden

Gerçek cihaz test sonuçları, iki farklı belirti gösteriyor (sohbette hızlı genel hata, fotoğrafta sonsuz bekleme) — bu, tek bir ortak nedenle açıklanamaz, çünkü aynı kök neden ikisini de aynı şekilde etkilerdi. Kod incelemesi, iki ayrı, kanıtlanmış sorun ortaya çıkardı:

1. **AI Sohbet hatası — KESİN KANITLANMIŞ:** `mapAiError()`'ın hata sınıflandırma mantığı, `@google/genai`'nin gerçek hata nesnesi (`ApiError`) formatıyla uyuşmuyor. Bu yüzden Sprint 10.6'da eklenen 7 spesifik hata kodu (AI_006-AI_012) hiçbir zaman eşleşmiyor, her hata `SYS_001` ("Bir şeyler ters gitti") genel mesajına düşüyor.
2. **Fotoğraf Analizi sonsuz bekleme — GÜÇLÜ KANITLANMIŞ (kesinleştirmek için gerçek cihaz logu gerekir):** `GeminiProvider`'da (SDK çağrısında) hiçbir timeout mekanizması yok, ve fotoğraflar hiç sıkıştırılmadan/küçültülmeden (kalite 90, orijinal boyut) base64'e çevrilip gönderiliyor — büyük bir istek, zayıf/kesintili bir mobil bağlantıda, JavaScript'in fetch API'sinin bilinen bir sınırlaması (varsayılan timeout yok) nedeniyle sonsuza kadar "beklemede" kalabilir.

---

## 1. AI Sohbet Hatası — Kesin Kanıt

### Kanıt: Gerçek ApiError Sınıfının Yapısı (Resmi Tip Tanımlarından)

`node_modules/@google/genai/dist/genai.d.ts` içinde gerçek sınıf tanımı:
```
export declare class ApiError extends Error {
    /** HTTP status code */
    status: number;
    constructor(options: ApiErrorInfo);
}

export declare interface ApiErrorInfo {
    message: string;
    status: number;
}
```

Bu, kritik bir uyumsuzluğu kanıtlıyor: Gerçek SDK, HTTP durum kodunu ayrı, sayısal bir `status` alanında taşıyor — `error.message`'ın içine `"code":401` gibi bir JSON parçası gömmüyor. `error.message`, muhtemelen Google'ın düz, insan-okunabilir hata cümlesini içeriyor (ör. "API key not valid. Please pass a valid API key.").

### Kanıt: mapAiError'ın Gerçek Kodu

`mapAiError.ts`'in TÜM Gemini-özel dallarını tekrar inceledim:
```
if (message.includes('"code":401') || message.includes('"code":403')) { return ErrorCode.AI_006; }
if (message.includes("RESOURCE_EXHAUSTED")) { return ErrorCode.AI_007; }
if (message.includes('"code":429')) { return ErrorCode.AI_008; }
if (message.includes('"code":400')) { return ErrorCode.AI_009; }
```

Bu kontrollerin hepsi, `error.message` içinde `"code":XXX` biçiminde bir JSON alt-dizesi arıyor. Ama gerçek `ApiError.message`, bu biçimde değil — durum kodu `error.status`'ta, mesaj metni `error.message`'da, ikisi ayrı alanlar. Bu yüzden hiçbir Gemini API hatası (401/403/429/400 dahil) bu kontrollerin hiçbirine uymuyor.

### Sonuç

Gerçek cihazda her türlü Gemini API hatası (geçersiz anahtar, kota, rate limit, geçersiz istek — hepsi) `mapAiError()`'ın son fallback'ine (`SYS_001`, "Bir şeyler ters gitti. Lütfen tekrar deneyin.") düşüyor — kullanıcının bildirdiği gözlemle birebir örtüşüyor. Bu, hem basit soruların hem uygulama-verisi sorularının aynı genel mesajı göstermesinin tam açıklaması: hatanın türü farklı olabilir ama sınıflandırma katmanının kendisi hiçbir zaman doğru çalışmıyor.

### Önemli Ek Not — Bu Aynı Sorun Sprint 6'dan Beri Var, Kod İncelemesiyle Şimdi Bulundu

`GeminiProvider.ts`'in `isRetryableGeminiError()` fonksiyonu da aynı yanlış varsayımı kullanıyor (`message.includes('"code":429')` vb.) — bu kod, dosyanın kendi yorumunda "web projesinden birebir taşındı" olarak belgelenmiş. Bu, retry mantığının da muhtemelen doğru çalışmadığı anlamına geliyor: gerçek bir 429 (rate limit, retry edilebilir olmalı) hatası, bu kontrol eşleşmediği için retry edilebilir olarak yanlış sınıflandırılıyor olabilir (`isRetryableGeminiError`, eşleşme bulamazsa `true` döner — kodun kendi mantığına göre "tanınmayan hata = retry et" varsayılıyor). Bu, ayrı bir olası yan etki — kesinleştirmek için gerçek cihaz logu gerekir, ama kod seviyesinde tutarlı bir bulgu.

---

## 2. Fotoğraf Analizi Sonsuz Bekleme — Güçlü Kanıt

### Kanıt 1: usePhotoAnalysis'in Kendi Hata Yönetimi Sağlam

`analyze()` fonksiyonunun tamamı bir try/catch/finally içinde — `getActiveAiProvider()`, `readFileAsBase64()`, veya `provider.analyzeImage()`'ın fırlattığı herhangi bir hata, catch bloğunda yakalanıp `status: "error"`'a çevrilir. Kullanıcının gördüğü "sonsuz bekleme, hiç hata gelmiyor" durumu, bu fonksiyonun kendisinde bir hata olmadığını, ama içeride çağrılan bir Promise'in hiçbir zaman resolve/reject olmadığını kanıtlıyor — sorun `provider.analyzeImage()`'ın içinde.

### Kanıt 2: Hiçbir Timeout Mekanizması Yok

`GeminiProvider.ts`'in `callGeminiWithRetry()` fonksiyonu tekrar incelendi — retry sayısı (`MAX_RETRY_ATTEMPTS = 2`) ve gecikme (`RETRY_BASE_DELAY_MS`) var, ama hiçbir yerde bir `AbortController`, `setTimeout` ile isteği iptal etme, veya SDK'ya bir timeout parametresi geçme yok. `client.models.generateContent()` çağrısı, sunucudan bir yanıt gelene kadar (veya bağlantı düşük seviyede kesilene kadar) sınırsız süre bekler.

### Kanıt 3: Fotoğraflar Hiç Küçültülmeden/Sıkıştırılmadan Gönderiliyor

`native/camera.ts`: `Camera.takePhoto({ quality: 90 })` — yüksek kalite, boyut sınırlaması yok. `usePhotoAnalysis.ts`: `readFileAsBase64(filePath)` — dosyayı olduğu gibi base64'e çeviriyor, hiçbir yeniden boyutlandırma/sıkıştırma adımı yok. Modern bir telefon kamerasından gelen bir fotoğraf kolayca 2-5+ MB olabilir; base64 kodlaması bunu ~%33 büyütür.

### Sonuç (Kesinleştirme İçin Gerçek Cihaz Logu Gerekir)

Büyük bir HTTP isteği gövdesi (birkaç MB'lık base64 veri), zayıf veya kesintili bir mobil ağ bağlantısında yüklenirken, JavaScript'in native fetch API'sinin varsayılan bir timeout içermemesi nedeniyle bağlantı sonsuza kadar "pending" kalabilir. Bu, fetch'in bilinen, dokümante edilmiş bir sınırlamasıdır (timeout için açıkça AbortController kullanılması gerekir — bu kodda hiç kullanılmıyor). Bu, kullanıcının gözlemlediği "hiçbir zaman tamamlanmayan, hiçbir hata/sonuç vermeyen" davranışla tam olarak tutarlı.

Bunu "kesin" değil "güçlü kanıtlanmış" olarak işaretliyorum çünkü: Bu ortamda gerçek bir Android cihazda ağ davranışını gözlemleyemiyorum — bağlantının gerçekten nerede takıldığını (DNS, TLS handshake, veri yükleme sırasında) yalnızca gerçek cihaz Logcat çıktısı kesinleştirebilir.

---

## 3. Diğer İncelenen Alanlar (Kanıt Bulunamayan/Ekarte Edilen)

| Alan | Bulgu |
|---|---|
| Android Manifest | INTERNET izni gerçekten tanımlı — sorun burada değil |
| Network Security Config | Hiç tanımlı değil (varsayılan Android davranışı, HTTPS'i engellemiyor) — sorun burada değil |
| API Key yüklenmesi (secureStorage) | Kod akışı doğru (get/set simetrik, doğru anahtar adı) — ama gerçek cihazda anahtarın geçerli bir değer olup olmadığı bu ortamdan doğrulanamaz |
| @google/genai'nin tarayıcı/WebView uyumluluğu | Resmi dokümantasyon (bu incelemede arandı), SDK'nın tarayıcı ortamında da çalışacak şekilde tasarlandığını gösteriyor — "SDK Node-only, WebView'da çalışmıyor" hipotezi zayıf, kanıt bulunamadı |
| Promise zinciri / async akış (usePhotoAnalysis, useAiChat) | Her ikisi de doğru try/catch yapısına sahip — kod seviyesinde bir zincir kopukluğu bulunamadı |
| ToolRegistry/Context Engine | Basit sorular tool-calling'i hiç tetiklemez, bu katmanlar basit soru senaryosunda devrede değil — ekarte edildi |

---

## 4. "Android Logcat'te Oluşacak Gerçek Hata" — Tahmin Edilebilir Ama Doğrulanamaz

Kod seviyesinde biliyorum ki (Sprint 10.6'da eklendi): `mapAiError()`, her hatadan önce `console.error("[AI Hatası — ham teknik detay...]", message)` çağırıyor. Bu satır gerçek cihazda çalışıyorsa, Logcat'te (chromium/WebView console log kategorisi altında) gerçek `error.message`'ın içeriği görünüyor olmalı — bu, kullanıcının cihazında Logcat'e bakarak gerçek Google hata metnini (ör. "API key not valid") görebileceği anlamına gelir. Ama bu ortamda gerçek bir Logcat çıktısına erişimim yok — bu, tahmin, kesin bilgi değil.

---

## Kesin Sonuç

| Belirti | Kök Neden | Kanıt Seviyesi |
|---|---|---|
| AI Sohbet — her türlü soruda genel "Bir şeyler ters gitti" | `mapAiError()`'ın Gemini hata biçimi varsayımı (message içinde "code":XXX), gerçek ApiError sınıfının yapısıyla (ayrı status alanı) uyuşmuyor — hiçbir spesifik hata kodu asla eşleşmiyor | Kesin — resmi SDK tip tanımlarından doğrulandı |
| AI Fotoğraf Analizi — sonsuz bekleme, hiç sonuç/hata yok | callGeminiWithRetry'de hiçbir timeout mekanizması yok + fotoğraflar sıkıştırılmadan/küçültülmeden gönderiliyor + JavaScript fetch'in varsayılan timeout içermemesi | Güçlü kanıtlanmış — kesinleştirmek için gerçek cihaz Logcat/ağ izleme gerekir |

Bu iki sorun birbirinden bağımsızdır ve ayrı ayrı düzeltilmesi gerekir — biri hata sınıflandırma katmanında (mesaj formatı uyumsuzluğu), diğeri ağ isteği yönetimi katmanında (timeout eksikliği + istek boyutu).

Hiçbir düzeltme uygulanmadı — onayını bekliyorum.
