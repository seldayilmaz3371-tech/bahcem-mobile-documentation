# Git Yapısı Analizi ve Kurtarma Planı

**Tarih:** 2026-07-21 · **Kural:** Bu belge SADECE salt-okunur (read-only) git komutlarıyla toplanan gerçek veriye dayanır. Hiçbir kod yazılmadı, hiçbir dosya değiştirilmedi, hiçbir commit/merge/rebase/push/force-push çalıştırılmadı. Kullanılan komutlar: `git status`, `git remote -v`, `git branch -vv -a`, `cat .git/config`, `git show-ref`, `git rev-list --count`, `git log --graph` — bunların hiçbiri repo durumunu değiştirmez (bir dosyayı okumakla aynı kategoridedir).

---

## KRİTİK BULGU — Bu Analizi Temelden Etkiliyor

Bu çalışma ortamındaki (Claude'un işlem yaptığı sandbox) git deposunda HİÇBİR remote tanımlı değil.

Gerçek kanıt:
- `git remote -v` → boş çıktı (hiçbir remote yok).
- `.git/config` → sadece `[core]` ve `[user]` bölümleri var, hiçbir `[remote "origin"]` bölümü yok.
- `.git/refs/remotes/` klasörü → hiç mevcut değil (bu klasör, bir remote'tan en az bir kez fetch yapıldığında oluşur — hiç oluşmamış).
- `git show-ref` → sadece `refs/heads/master` var, `refs/remotes/origin/master` diye bir referans hiç yok.
- `git branch -vv` → hiçbir tracking bilgisi (`[origin/master]` gibi) göstermiyor.

### Bunun Anlamı

Sizin bahsettiğiniz "remote fetch çalışıyor, remote push reddediliyor, origin/master mevcut, GitHub Sprint 10.3'te" durumu, bu sandbox'taki repo ile ilgili olamaz — çünkü burada hiçbir remote hiçbir zaman tanımlanmamış. Sizin bahsettiğiniz git problemi, kendi bilgisayarınızdaki ayrı bir git deposuyla ilgili olmalı — muhtemelen bu sandbox'ta üretilen ZIP dosyalarını kendi makinenize indirip, kendi git deponuza (GitHub'a bağlı olan) aktardığınız, ayrı bir kopya.

Bu, dürüstçe belirtilmesi gereken bir sınırdır: Ben, sizin kendi makinenizdeki git deposunun gerçek `origin/master` durumunu, tracking yapılandırmasını, veya fork-point'ini buradan göremem ve doğrulayamam. Aşağıdaki analiz, (1) bu sandbox'taki commit geçmişinin gerçek yapısını, ve (2) sizin kendi ortamınızda çalıştırmanız gereken genel bir teşhis ve senkronizasyon yöntemini sunuyor — spesifik commit hash'leri üzerinden değil, çünkü sizin `origin/master`'ınızın gerçek durumunu bilmiyorum.

---

## 1. Repository Analizi (Bu Sandbox'ta Gözlemlenen)

| Özellik | Değer |
|---|---|
| Toplam commit sayısı | 133 |
| Branch sayısı | 1 (`master`) |
| Merge commit sayısı | 0 — tamamen doğrusal (linear) bir geçmiş |
| Remote sayısı | 0 |
| Working tree durumu | Temiz (`nothing to commit, working tree clean`) |
| HEAD | `6c07939` (Sprint 10.4 Düzeltme Paketi'nin son commit'i) |

## 2. Branch Analizi

Bu sandbox'ta sadece `master` dalı var — hiçbir başka yerel veya uzak dal referansı yok. Bu, basit bir yapı: tek bir doğrusal geçmiş, dallanma/birleştirme (merge) hiç yaşanmamış.

## 3. Tracking Analizi

Tracking hiç kurulmamış. `git branch -vv`'nin çıktısı, normalde bir dal `origin/master`'ı takip ediyorsa `master 6c07939 [origin/master] ...` şeklinde görünür — burada sadece `master 6c07939 ...` var, köşeli parantez içinde hiçbir referans yok. Bu, `git push`/`git pull`'un hangi remote'a karşı çalışacağının hiç tanımlanmadığı anlamına gelir.

## 4. Merge-Base Analizi

Uygulanamaz — bu sandbox'ta karşılaştırılacak ikinci bir referans (`origin/master`) yok. Merge-base, iki dal arasındaki en yakın ortak atayı bulur; burada sadece tek bir dal olduğu için bu analiz burada anlamsız. Sizin kendi ortamınızda çalıştırmanız gereken komut: `git merge-base master origin/master` (Bölüm "Kurtarma Planı"nda detaylandırıldı).

## 5. Fork-Point Analizi

Aynı gerekçeyle burada uygulanamaz. Fork-point analizi, yerel dalın uzak daldan ne zaman ayrıldığını bulur — bu, sizin makinenizde `git merge-base --fork-point master origin/master` ile bulunmalı.

## 6. Commit DAG Analizi (Bu Sandbox'ın Gerçek Geçmişi)

Bu sandbox'taki DAG tamamen doğrusal — 133 commit, tek bir zincir, hiç dallanma yok. Son commit'ler (gerçek `git log --graph` çıktısı):

```
* 6c07939 docs(sprint-10.4-duzeltme): Teknik Rapor, BUILD_INFO.md, CHANGELOG.md ve module-status.md guncellendi
* dd66e67 fix(sprint-10.4-duzeltme): initialMaintenanceType entegrasyonu + Sulama saati gorunumu (Liste+Detay)
* 7b39907 docs(sprint-10.5): Teknik Rapor, BUILD_INFO.md ve module-status.md guncellendi
* ef12114 feat(sprint-10.5): Fotograf Analizi Navigasyon Entegrasyonu
* 49f0773 docs(sprint-10.4): Teknik Rapor, BUILD_INFO.md ve module-status.md guncellendi
* 84bd8e2 feat(sprint-10.4): Toplu Islemlerde Geriye Donuk Tarih/Saat + Sulama Baslangic/Bitis Saati
* e9134ee feat(sprint-10.4): Toplu Gozlem/Bakimda Geriye Donuk Tarih-Saat + Sulama Baslangic-Bitis Suresi
* ed388d9 docs(sprint-10.3): Teknik Rapor, BUILD_INFO.md ve module-status.md guncellendi
  ... (Sprint 10.1-10.3, 9.1-9.2 ve öncesi devam ediyor)
```

Zincirin en başı (proje kökü) gerçek ilk 3 commit:
```
00b05fb feat(altyapi): şifreli SQLite bağlantı katmanı ve repository deseni
2e1990e feat: Capacitor entegrasyonu ve bağımlılık kararları
9b7917f chore: proje iskeleti (Vite + React 19 + TypeScript)
```

## 7. Local ve Remote Commit Ağacı Karşılaştırması

Bu sandbox'ta yapılamıyor (Bölüm 1'deki kritik bulgu gereği — karşılaştırılacak bir remote referansı yok). Bu, sizin kendi ortamınızda yapılması gereken bir adım.

## 8. Ayrışmanın Olası Nedeni (Genel Değerlendirme — Sizin Ortamınıza Özel Detaylar Olmadan)

Sizin bildirdiğiniz senaryoya (GitHub Sprint 10.3'te, yerel Sprint 10.4/10.5 içeriyor, push reddediliyor) dayanarak, genel olarak bu tür bir ayrışmanın en yaygın 3 nedeni:

1. GitHub'daki `origin/master`, sizin yerel `master`'ınızdan bağımsız yeni commit(ler) almış olabilir (ör. GitHub üzerinden doğrudan bir değişiklik, başka bir makineden yapılan bir push, veya bir Pull Request merge'ü) — bu durumda yerel ve uzak dallar birbirinden ayrı (diverged) olur, `git push` bunu reddeder ("non-fast-forward" hatası) çünkü Git, uzaktaki commit'lerin üzerine yerel geçmişi kayıtsızca yazmanıza izin vermez.
2. Yerel `master`, `origin/master`'ı hiç takip etmiyor olabilir (`git branch -vv`'de tracking referansı yoksa) — bu durumda `git push` komutu, hangi uzak dala push edileceğini bile bilmeyebilir, bu da farklı bir hata (ilişkilendirilmemiş dal) üretir.
3. Yerel repo, `origin/master`'ın gerisinde kalmış olabilir (basit bir "fast-forward gerekiyor" durumu) — bu en basit, en az riskli senaryo, ama sizin "GitHub'da 10.3, yerelde 10.4/10.5 var" ifadeniz, bunun tam tersini (yerel daha ileride) işaret ediyor — bu da 1. senaryoyu (gerçek bir diverge) daha olası kılıyor.

Kesin nedeni, sizin kendi ortamınızda çalıştıracağınız teşhis komutlarının çıktısı belirleyecek (Kurtarma Planı'nın 1. adımı).

---

## Kurtarma Planı (Hiçbir Komut Henüz Çalıştırılmadı)

Bu plan, sizin kendi git ortamınızda, sırayla uygulanmak üzere tasarlandı. Her adım, bir öncekinin çıktısına göre yorumlanmalı — kör kör ilerlemeyin, her adımın çıktısını okuyun.

### Adım 0 — Güvenlik Ağı (Hiçbir Şeyi Kaybetmemek İçin)

```
git branch yedek-sprint10.5-oncesi
```
Neden gerekli: Bu, mevcut `master`'ınızın tam bir kopyasını, ayrı bir dal adı altında oluşturur — hiçbir commit'i silmez, sadece "eğer bir şey ters giderse buraya dönebilirim" güvencesi sağlar. Bu adım, sıfır risk taşır (sadece yeni bir referans oluşturur, hiçbir şeyi değiştirmez).

### Adım 1 — Gerçek Durumu Görmek (Teşhis)

```
git fetch origin
git status
git branch -vv
git log --oneline --graph --all -30
```
Neden gerekli: `git fetch` (push DEĞİL), GitHub'daki gerçek `origin/master` durumunu yerel bir referansa (`refs/remotes/origin/master`) indirir — hiçbir yerel dalınızı değiştirmez, sadece "uzakta ne var" bilgisini günceller. `git status` ve `git branch -vv`, bu fetch sonrası artık "kaç commit ileride/geride olduğunuzu" gösterecek. `git log --graph --all`, yerel `master` ile `origin/master`'ın gerçekten aynı çizgide mi yoksa ayrışmış mı olduğunu görsel olarak gösterir.

### Adım 2 — Merge-Base ve Fork-Point'i Bulmak

```
git merge-base master origin/master
git merge-base --fork-point master origin/master
```
Neden gerekli: İlk komut, iki dalın ortak atasını (nereden ayrıldıklarını) bulur. Eğer bu, `origin/master`'ın kendisiyle aynıysa, yerel dalınız sadece "ileride" demektir (basit durum). Eğer farklıysa, gerçek bir ayrışma var demektir — GitHub'da yerel geçmişinizde olmayan commit'ler var.

### Adım 3 — Karar Noktası (Çıktıya Göre 2 Yol)

Senaryo A — `origin/master`, sizin `master`'ınızın bir atası ise (basit "ileride" durumu):
```
git push origin master
```
Neden gerekli/güvenli: Bu durumda push, bir "fast-forward" işlemidir — GitHub'daki geçmişin üzerine sadece yeni commit'leriniz eklenir, hiçbir şey üzerine yazılmaz, hiçbir commit kaybolmaz. Bu, en olası ve en güvenli senaryodur (sizin "yerel ileride" ifadenizle tutarlı).

Senaryo B — Gerçek bir ayrışma varsa (GitHub'da yerelde olmayan commit'ler varsa):
```
git log origin/master..master --oneline    # Sadece yerelde olan commitler
git log master..origin/master --oneline    # Sadece GitHub'da olan commitler
```
Neden gerekli: Bu iki komut, tam olarak hangi commit'lerin nerede olduğunu gösterir — hiçbir şeyi değiştirmez, sadece karşılaştırma yapar. Bu çıktıyı görmeden Senaryo B için bir sonraki adıma geçilmemeli.

Eğer `master..origin/master` boşsa (GitHub'da yerelde olmayan hiçbir commit yoksa) ama push yine de reddediliyorsa, bu muhtemelen tracking eksikliği demektir (Bölüm 3'teki bulgu) — bu durumda:
```
git branch --set-upstream-to=origin/master master
git push
```
Neden gerekli: `--set-upstream-to`, yerel `master`'ı `origin/master`'ı takip edecek şekilde sadece yapılandırır — hiçbir commit'i değiştirmez. Bundan sonra `git push` (parametresiz) çalışacaktır; bu, sizin "her sprint sonunda sadece `git push` yeterli olsun" hedefinizi karşılar.

Eğer `master..origin/master` doluysa (GitHub'da gerçekten yerelde olmayan commit'ler varsa) — bu, gerçek bir insan kararı gerektirir, otomatik bir komut önerilmemeli:
- Bu commit'ler önemliyse: `git merge origin/master` (sizin yasakladığınız bir işlem — bu yüzden bu adım için sizin açık onayınız gerekir, ben bunu öneremem/çalıştıramam).
- Bu commit'ler yanlışlıkla oluşmuşsa (ör. GitHub üzerinden yanlışlıkla bir değişiklik yapıldıysa): bunun kaldırılması ayrı, dikkatli bir karar gerektirir — force push burada kesinlikle önerilmez, çünkü GitHub'daki commit'leri kalıcı olarak siler.

### Adım 4 — Sürekli Senkronizasyon İçin (Hedefinizin Karşılanması)

Adım 3'ün (Senaryo A veya B'nin çözümü) ardından, tracking doğru kurulmuş olacak. Bundan sonra her sprint sonunda:
```
git push
```
tek başına yeterli olacaktır — çünkü `master` artık `origin/master`'ı takip ediyor olacak, Git otomatik olarak doğru remote/dala push edecek.

---

## Özet — Neden Bu Sırayla?

1. Önce yedek (Adım 0) — hiçbir riski olmayan bir güvenlik ağı.
2. Sonra sadece okuma (Adım 1-2, `fetch`+`log`+`merge-base`) — hiçbir şeyi değiştirmeden gerçek durumu öğrenmek.
3. Sonra, çıktıya göre ya basit bir push (Senaryo A) ya da dikkatli bir inceleme + sizin onayınızla ilerleyecek bir karar noktası (Senaryo B).
4. Hiçbir adımda `--force`, `git reset --hard` origin'e karşı, veya commit kaybına yol açabilecek bir komut önerilmedi — sizin "hiçbir commit kaybetmeden" hedefinizle tam uyumlu.

Bu plandaki hiçbir komut henüz çalıştırılmadı. Adım 1'in çıktısını paylaşırsanız, Adım 3'ün hangi senaryosunun geçerli olduğunu birlikte netleştirip bir sonraki adıma karar verebiliriz.
