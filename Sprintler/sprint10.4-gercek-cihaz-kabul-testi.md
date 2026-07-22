# Sprint 10.4 — Gerçek Cihaz Kabul Test Dokümanı

**Tarih:** 2026-07-21 · **Kural:** Bu belge SADECE bir test dokümanıdır. Hiçbir kod yazılmadı, hiçbir dosya değiştirilmedi, hiçbir commit oluşturulmadı.

**Nasıl kullanılır:** Bu dokümandaki her senaryoyu, APK'yı telefona kurduktan sonra sırayla uygulayın. Her adımda GERÇEKTEN gördüğünüz ekran metnini, bu belgede yazan metinle karşılaştırın — arayüz Türkçe olduğu için tüm buton/alan adları gerçek Türkçe metinlerle yazıldı.

---

## Genel Ön Hazırlık (Tüm Senaryolar İçin Ortak)

1. Uygulamayı açın, kilit ekranını geçin.
2. Ana ekranda ("Parseller") en az 1 parsel ve o parselde en az 1 ağaç bulunduğundan emin olun (yoksa önce bir parsel + ağaç oluşturun).
3. Bir parseli açın (Parsel Formu'na girin — parsel adına dokunun).

---

## Senaryo 1 — Sulama

**Ön Koşul:** Genel Ön Hazırlık tamamlanmış, bir Parsel Formu ekranındasınız (en az 1 ağaçlı bir parsel).

**Adımlar:**
1. Parsel Formu'nda "Toplu İşlemler" butonuna dokunun.
2. Açılan ekranda "Sulama" butonuna dokunun.
3. Açılan formda, "Tür" alanının "Sulama" olarak seçili gelip gelmediğini kontrol edin.
4. "Sulama Süresi" başlığı altında "Başlangıç Saati" ve "Bitiş Saati" alanlarının göründüğünü kontrol edin.
5. Başlangıç Saati'ne 06:15, Bitiş Saati'ne 08:05 girin.
6. "Tarih" ve "Saat" alanlarının (formun üstünde, geriye dönük tarih/saat için) şu anki tarih/saatle dolu geldiğini kontrol edin.
7. "Tüm Ağaçlara Uygula" seçili bırakıp "Uygula" butonuna dokunun.
8. Çıkan onay penceresinde "Devam" / "Tamam" seçeneğine dokunun.
9. Sonuç ekranında "N kayıt oluşturuldu. 0 hata." mesajını kontrol edin.
10. "Geri" butonuna basıp Parsel Formu'na dönün, sonra "Bakımı Görüntüle" butonuna dokunun (Liste ekranı).
11. Listede yeni oluşturulan "Sulama" kaydına bakın — kart üzerinde saat bilgisinin (06:15–08:05 gibi) göründüğünü kontrol edin.
12. Karta dokunup Detay/Düzenleme ekranını açın.
13. "Başlangıç Saati" ve "Bitiş Saati" alanlarının 06:15/08:05 değerleriyle dolu geldiğini kontrol edin.
14. Bir saati değiştirip "Kaydet"e dokunun, listeye dönüp değişikliğin yansıdığını kontrol edin.

**Beklenen Sonuç:** Form "Sulama" seçili açılır, saat alanları görünür, kayıt oluşturulur, listede saat bilgisi görünür, detay ekranında saatler doğru yüklenir ve düzenlenebilir.

**Başarısızlık Durumu:** Form farklı bir türle (örn. varsayılan olarak başka bir şey) açılıyorsa, saat alanları hiç görünmüyorsa, listede/detayda saat bilgisi hiç yoksa veya düzenleme kaydedilmiyorsa — bu senaryo BAŞARISIZ sayılır.

---

## Senaryo 2 — İlaçlama

**Ön Koşul:** Senaryo 1 ile aynı.

**Adımlar:**
1. Parsel Formu'nda "Toplu İşlemler" → "İlaçlama" butonuna dokunun.
2. "Tür" alanının "İlaçlama" olarak seçili geldiğini kontrol edin.
3. Sulama Süresi alanlarının (Başlangıç/Bitiş Saati) HİÇ GÖRÜNMEDİĞİNİ kontrol edin.
4. "Tarih"/"Saat" alanlarının göründüğünü ve düzenlenebildiğini kontrol edin (dünün tarihini girip test edin).
5. "Uygula"ya dokunup onaylayın, sonuç ekranını kontrol edin.
6. Liste ekranına gidin — yeni "İlaçlama" kaydını bulun, kartta hiçbir saat bilgisinin görünmediğini kontrol edin.
7. Karta dokunup Detay ekranını açın — Başlangıç/Bitiş Saati alanlarının bu ekranda da hiç görünmediğini kontrol edin.

**Beklenen Sonuç:** Form "İlaçlama" seçili açılır, saat alanları (Sulama'ya özel) hiçbir yerde görünmez, geriye dönük tarih doğru kaydedilir.

**Başarısızlık Durumu:** Form "Sulama" (veya başka bir tür) ile açılıyorsa, saat alanları yanlışlıkla görünüyorsa — BAŞARISIZ.

---

## Senaryo 3 — Gübreleme

**Ön Koşul:** Senaryo 1 ile aynı.

**Adımlar:** Senaryo 2 ile birebir aynı adımlar, sadece 1. adımda "Gübreleme" butonuna dokunun ve tüm kontrollerde "Gübreleme" türünü arayın.

**Beklenen Sonuç:** Form "Gübreleme" seçili açılır, saat alanları hiçbir yerde görünmez.

**Başarısızlık Durumu:** Senaryo 2 ile aynı kriterler.

---

## Senaryo 4 — Budama

**Ön Koşul:** Senaryo 1 ile aynı.

**Adımlar:** Senaryo 2 ile birebir aynı adımlar, sadece 1. adımda "Budama" butonuna dokunun ve tüm kontrollerde "Budama" türünü arayın.

**Beklenen Sonuç:** Form "Budama" seçili açılır, saat alanları hiçbir yerde görünmez.

**Başarısızlık Durumu:** Senaryo 2 ile aynı kriterler.

---

## Senaryo 5 — Biçme

**Ön Koşul:** Senaryo 1 ile aynı.

**Adımlar:**
1. Parsel Formu'nda "Toplu İşlemler" → "Biçme" butonuna dokunun.
2. "Tür" alanının "Biçme" olarak seçili geldiğini kontrol edin (Toplu İşlemler formunun kendi dropdown'ında "Biçme" etiketi görünür).
3. Saat alanlarının görünmediğini kontrol edin.
4. "Uygula"ya dokunup onaylayın, sonucu kontrol edin.
5. Liste ekranına gidin — yeni oluşan kaydı bulun.

ÖZEL NOT — BEKLENEN, BİLİNEN BİR DURUM (BAŞARISIZLIK DEĞİL): Liste ve Detay ekranlarında bu kayıt "Biçme" değil, "Diğer" olarak görünecektir. Bu, bu sprintte bilinçli olarak düzeltilmemiş, ayrı bir sprintte ele alınacak bilinen bir durumdur — Toplu İşlemler formunun kendisinde "Biçme" doğru görünmesi yeterlidir, Liste/Detay'da "Diğer" görünmesi bu testte başarısızlık sayılmamalıdır.

**Beklenen Sonuç:** Toplu İşlemler formu "Biçme" seçili açılır, saat alanları görünmez, kayıt oluşturulur. Liste/Detay'da "Diğer" olarak görünmesi normaldir.

**Başarısızlık Durumu:** Toplu İşlemler formunun KENDİSİNDE "Biçme" yerine başka bir tür seçiliyse, veya kayıt hiç oluşmuyorsa — BAŞARISIZ. Liste/Detay'da "Diğer" yazması BAŞARISIZLIK DEĞİLDİR.

---

## Gerçek Cihazda Özellikle Kontrol Edilmesi Gereken Noktalar

| Kontrol Noktası | Nasıl Test Edilir |
|---|---|
| Dropdown doğru açılıyor mu? | Her 5 senaryoda, menüden hangi butona basıldıysa formdaki "Tür" alanının GERÇEKTEN o türle geldiğini gözle kontrol edin — bu, Sprint 10.4'ün düzelttiği ana sorundu. |
| Saat alanı gerçekten gizleniyor mu? | İlaçlama/Gübreleme/Budama/Biçme formlarını açtığınızda, "Başlangıç Saati"/"Bitiş Saati" yazılarının EKRANDA HİÇ olmadığından emin olun (kaydırarak da kontrol edin — gizli bir alan aşağıda kalmış olabilir). |
| Geri tuşu doğru çalışıyor mu? | Toplu İşlemler menüsündeyken telefonun donanım/gezinme geri tuşuna basın — bir önceki ekrana (Parsel Formu) dönmeli. Form doldururken geri tuşuna basın — menüye dönmeli, uygulama ÇÖKMEMELİ. |
| Form kapanınca veri korunuyor mu? | Bir formu yarı doldurup "Geri"ye basıp tekrar aynı türe girin — form BOŞ gelmelidir (bu, beklenen davranıştır, veri kalıcı olarak saklanmaz; "korunuyor" ifadesi burada "uygulama çökmüyor, form temiz açılıyor" anlamındadır). |
| SQLite'a gerçekten kaydediliyor mu? | Kayıt oluşturduktan sonra uygulamayı tamamen kapatıp (arka plandan da kaldırıp) yeniden açın, aynı parselin Bakım Listesi'ne gidin — kayıt hâlâ orada olmalı. |
| Liste güncelleniyor mu? | Toplu İşlem sonrası "Bakımı Görüntüle" ekranına gittiğinizde, listeyi yenilemeden (uygulamayı kapatmadan) yeni kaydın orada olduğunu görmelisiniz. |
| Birden fazla ağaç seçiliyken doğru sayıda kayıt oluşuyor mu? | "Ağaç Seçerek Uygula" moduna geçip 2-3 ağaç işaretleyin, "N kayıt oluşturuldu" mesajındaki sayının işaretlediğiniz ağaç sayısıyla eşleştiğini kontrol edin. |
| Saat alanına geçersiz bir değer girilebiliyor mu? | Saat alanına dokunduğunuzda cihazın kendi saat seçici arayüzünün (native time picker) açıldığını, serbest metin girilemediğini kontrol edin. |

---

## Kabul Kriterleri

Sprint 10.4'ün tamamlandı sayılabilmesi için aşağıdaki maddelerin tamamı başarılı olmalıdır:

1. Senaryo 1-5'in tamamında, menüden tıklanan tür ile formda açılan tür birebir eşleşmelidir (bu, düzeltilen ana sorundu — tek bir istisna bile kabul edilemez).
2. Sadece Sulama'da Başlangıç/Bitiş Saati alanları görünmelidir; diğer 4 türde (İlaçlama/Gübreleme/Budama/Biçme) bu alanlar hiçbir zaman görünmemelidir.
3. Sulama kaydı oluşturulduktan sonra, Liste ekranında saat bilgisi görünmelidir.
4. Sulama kaydının Detay ekranında saatler doğru yüklenmeli ve değiştirilip kaydedilebilmelidir.
5. Diğer 4 türde, Liste ve Detay ekranlarında saat alanı hiç görünmemelidir (boş bir alan olarak bile değil — tamamen yok olmalıdır).
6. Hiçbir senaryoda uygulama çökmemelidir — özellikle geri tuşu, form iptali, ve arka plana alıp geri getirme senaryolarında.
7. Uygulama tamamen kapatılıp yeniden açıldığında, oluşturulan kayıtlar kaybolmamalıdır (SQLite'a gerçekten yazıldığının kanıtı).
8. "Biçme" senaryosunda Liste/Detay'da "Diğer" görünmesi kabul edilebilir bir durumdur — bu madde kabul kriterlerini BAŞARISIZ KILMAZ, sadece bilinen bir sınır olarak not düşülmelidir.

**Not:** Bu kabul testi, gerçek cihazda sizin tarafınızdan uygulanacaktır — geliştirme ortamında Android SDK/gerçek cihaz erişimi olmadığı için bu testler otomatik olarak çalıştırılamamıştır (bu, Sprint 10.4 ve Sprint 10.5 teknik raporlarında da dürüstçe belirtilmişti).
