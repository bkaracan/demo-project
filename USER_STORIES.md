# KULLANICI HİKAYELERİ VE BDD KABUL KRİTERLERİ (USER STORIES)

**Proje:** demo-project — Bireysel Zaman & Görev Yönetim Platformu (To-Do App)  
**Tarih:** 19 Eylül 2026  
**Sürüm:** v1.0.0  
**SDLC Fazı:** Aşama 02 (Gereksinim Analizi) — Adım A4 (User Stories & BDD Kabul Kriterleri)  
**Metodoloji:** Behavior-Driven Development (BDD) & INVEST Prensipleri  
**Girdi Belgesi:** [SRS.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/SRS.md)  
**Statü:** **ONAYLANDI (Quality Gate Geçildi)**  

---

## 1. GİRİŞ VE METODOLOJİ

Bu doküman, `SRS.md` belgesinde yer alan tüm **Must** (Zorunlu) öncelikli fonksiyonel gereksinimlerin geliştirici ve QA otomasyon mühendisleri tarafından doğrudan test senaryolarına (`*.feature`, Jest, Playwright) dönüştürülebileceği BDD formatındaki kullanıcı hikayelerini ve **Given-When-Then (Gherkin)** kabul kriterlerini içerir.

Tüm hikayeler **INVEST** (Independent, Negotiable, Valuable, Estimable, Small, Testable) prensiplerine uygun olarak dilimlenmiştir. Her hikaye en az bir başarı (Happy Path) ve bir hata/sınır (Edge Case / Alternative Path) senaryosu içerir.

---

## 2. KİMLİK DOĞRULAMA VE GÜVENLİK HİKAYELERİ (US-AUTH)

### [US-001] Güvenli Kullanıcı Kaydı
- **İlgili SRS Maddesi:** `FR-AUTH-001`
- **Hikaye:**  
  *Bir **Ziyaretçi** olarak,*  
  *E-posta adresim ve güçlü bir parola belirleyerek sisteme kaydolmak istiyorum,*  
  *Böylece kendime ait kişisel görev ve zaman yönetimi alanımı başlatabileyim.*

#### Kabul Kriterleri (Gherkin Formatı):
```gherkin
Scenario: Başarılı Yeni Kullanıcı Kaydı (Happy Path)
  Given sistemde kayıtlı olmayan geçerli bir "burak@example.com" e-posta adresim varken
  And en az 8 karakterli, büyük harf, rakam ve özel karakter içeren "P@ssw0rd2026" parolası girdiğimde
  When "Kayıt Ol" butonuna tıkladığımda
  Then sistem HTTP 201 Created durum kodu dönmelidir
  And parola veritabanına Argon2id ile hash'lenmiş olarak kaydedilmelidir
  And yanıt gövdesinde kullanıcı kimliği (ID) ve onay mesajı yer almalıdır.

Scenario: Mükerrer E-posta ile Kayıt Denemesi (Hata Durumu)
  Given "burak@example.com" e-posta adresi veritabanında zaten kayıtlıyken
  When bu e-posta adresiyle tekrar kayıt olma isteği gönderdiğimde
  Then sistem HTTP 409 Conflict durum kodu dönmelidir
  And "Bu e-posta adresi zaten kullanımda" hata mesajı gösterilmelidir
  And mükerrer yeni bir kullanıcı oluşturulmamalıdır.
```

---

### [US-002] Kullanıcı Oturumu Açma ve Token Üretimi
- **İlgili SRS Maddesi:** `FR-AUTH-002`
- **Hikaye:**  
  *Bir **Kayıtlı Kullanıcı** olarak,*  
  *E-posta ve parolamla sisteme giriş yapmak istiyorum,*  
  *Böylece yetkilendirilmiş API servislerine ve kişisel görev panoma erişebileyim.*

#### Kabul Kriterleri:
```gherkin
Scenario: Geçerli Kimlik Bilgileri ile Başarılı Giriş (Happy Path)
  Given kayıtlı ve aktif bir hesabım varken
  When geçerli e-posta ve doğru parolamı girip "Giriş Yap" butonuna bastığımda
  Then sistem HTTP 200 OK durum kodu dönmelidir
  And 15 dakika geçerli JWT Access Token ile 7 gün geçerli Refresh Token üretilmelidir
  And token'lar HttpOnly, Secure ve SameSite=Strict çerezlerinde istemciye teslim edilmelidir
  And kullanıcı günlük görev panosuna yönlendirilmelidir.

Scenario: Hatalı Parola ile Giriş Denemesi (Hata Durumu)
  Given kayıtlı bir kullanıcı hesabım varken
  When geçerli e-posta adresimin yanına hatalı bir parola girip giriş yapmayı denediğimde
  Then sistem HTTP 401 Unauthorized durum kodu dönmelidir
  And ekranda genel "Geçersiz e-posta veya parola" mesajı gösterilmelidir
  And kullanıcının sistemde var olup olmadığı ifşa edilmemelidir.
```

---

### [US-003] Brute-Force Koruması ve IP Hız Sınırlaması (Rate Limiting)
- **İlgili SRS Maddesi:** `FR-AUTH-003`
- **Hikaye:**  
  *Bir **Sistem Yöneticisi** olarak,*  
  *Aynı IP'den gelen ardışık başarısız oturum açma denemelerinin engellenmesini istiyorum,*  
  *Böylece kullanıcı hesapları kaba kuvvet (brute-force) saldırılarına karşı korunsun.*

#### Kabul Kriterleri:
```gherkin
Scenario: 1 Dakikada 5 Başarısız Denemede IP Engelleme (Güvenlik Kuralı)
  Given herhangi bir IP adresinden son 60 saniyede 5 kez hatalı parola ile login isteği yapılmışken
  When aynı IP'den 6. kez oturum açma isteği gönderildiğinde
  Then sistem kimlik kontrolü yapmadan doğrudan HTTP 429 Too Many Requests dönmelidir
  And yanıtta 15 dakikalık bekleme süresini belirten "Retry-After: 900" başlığı bulunmalıdır
  And bu güvenlik olayı sistem audit loglarına kaydedilmelidir.
```

---

### [US-004] Güvenli Oturum Kapatma ve Token İptali
- **İlgili SRS Maddesi:** `FR-AUTH-004`
- **Hikaye:**  
  *Bir **Giriş Yapmış Kullanıcı** olarak,*  
  *Oturumumu güvenle kapatmak istiyorum,*  
  *Böylece benden sonra cihazı kullanan kişiler görevlerime erişemesin.*

#### Kabul Kriterleri:
```gherkin
Scenario: Başarılı Oturum Kapatma (Happy Path)
  Given geçerli bir oturuma sahipken
  When "Çıkış Yap" eylemini gerçekleştirdiğimde
  Then sistem HTTP 200 OK dönmelidir
  And istemcideki kimlik doğrulama çerezleri temizlenmelidir
  And mevcut Refresh Token Valkey kara listesine (blacklist) eklenerek anında geçersiz kılınmalıdır.
```

---

## 3. GÖREV VE YAŞAM DÖNGÜSÜ HİKAYELERİ (US-TASK)

### [US-005] Yeni Görev Oluşturma
- **İlgili SRS Maddesi:** `FR-TASK-001`
- **Hikaye:**  
  *Bir **Kullanıcı** olarak,*  
  *Yeni bir görev başlığı ve isteğe bağlı detaylar girerek görev oluşturmak istiyorum,*  
  *Böylece yapmam gereken işleri kayıt altına alıp unutulmasını önleyebileyim.*

#### Kabul Kriterleri:
```gherkin
Scenario: Geçerli Başlık ile Görev Oluşturma (Happy Path)
  Given oturum açmış aktif bir kullanıcıyken
  When görev başlığına "Q3 Finans Raporunu Hazırla" yazıp kaydettiğimde
  Then sistem HTTP 201 Created kodu dönmelidir
  And görev varsayılan olarak "TODO" durumu ve "MEDIUM" öncelik seviyesiyle kaydedilmelidir
  And görev kullanıcı kimliğimle (user_id) ilişkilendirilmelidir.

Scenario: Boş Görev Başlığı ile Oluşturma Denemesi (Hata Durumu)
  Given oturum açmış bir kullanıcıyken
  When görev başlığını boş bırakıp veya yalnızca boşluk girip kaydetmeyi denediğimde
  Then sistem HTTP 400 Bad Request dönmelidir
  And "Görev başlığı 1 ile 255 karakter arasında olmalıdır" hata mesajı dönmelidir
  And veritabanına yeni bir kayıt eklenmemelidir.
```

---

### [US-006] Görev Güncelleme ve Yetkisiz Erişim (IDOR) Koruması
- **İlgili SRS Maddesi:** `FR-TASK-002`
- **Hikaye:**  
  *Bir **Kullanıcı** olarak,*  
  *Sadece kendi oluşturduğum görevlerin detaylarını düzenlemek ve başkalarının görevlerime erişmesini engellemek istiyorum,*  
  *Böylece kişisel görevlerimin gizliliği ve veri bütünlüğü korunsun.*

#### Kabul Kriterleri:
```gherkin
Scenario: Kendi Görevini Başarıyla Güncelleme (Happy Path)
  Given bana ait (ID: task-101) mevcut bir görev varken
  When görev açıklamasını "Revize edilmiş finans özeti eklendi" olarak güncellediğimde
  Then sistem HTTP 200 OK dönmeli ve güncellenmiş görev verisini teslim etmelidir.

Scenario: Başka Bir Kullanıcının Görevine Müdahale Denemesi (IDOR Engeli)
  Given "user-B" kullanıcısına ait (ID: task-999) bir görev varken
  When ben ("user-A") olarak "PUT /api/v1/tasks/task-999" isteği gönderdiğimde
  Then sistem görevin varlığını ifşa etmemek için HTTP 404 Not Found dönmelidir
  And söz konusu görev üzerinde hiçbir değişiklik yapılmamalıdır.
```

---

### [US-007] Görev Durumunu Güncelleme ve Tamamlama
- **İlgili SRS Maddesi:** `FR-TASK-003`
- **Hikaye:**  
  *Bir **Kullanıcı** olarak,*  
  *Tamamladığım bir görevi "Tamamlandı" olarak işaretlemek istiyorum,*  
  *Böylece üretkenlik ilerlememi görebileyim ve gereksiz bildirimleri durdurabileyim.*

#### Kabul Kriterleri:
```gherkin
Scenario: Görevi Başarıyla Tamamlama (Happy Path)
  Given "TODO" durumunda bana ait bir görev varken
  When durumunu "COMPLETED" olarak güncellediğimde
  Then sistem HTTP 200 OK dönmelidir
  And görevin durumunu "COMPLETED" yapmalı ve "completed_at" alanına anlık zaman damgasını mühürlemelidir
  And bu göreve bağlı bekleyen tüm hatırlatıcı bildirimler otomatik iptal edilmelidir.

Scenario: Arşivlenmiş Görevi Güncelleme Denemesi (Hata Durumu)
  Given "ARCHIVED" durumunda bulunan bir görevim varken
  When durumunu "IN_PROGRESS" yapmaya çalıştığımda
  Then sistem HTTP 400 Bad Request dönmelidir
  And "Arşivlenmiş görev güncellenemez, önce arşivden çıkarılmalıdır" uyarısı gösterilmelidir.
```

---

### [US-008] Göreve Öncelik Seviyesi ve Etiket Ekleme
- **İlgili SRS Maddeleri:** `FR-TASK-004`, `FR-TASK-005`
- **Hikaye:**  
  *Bir **Kullanıcı** olarak,*  
  *Görevlerime öncelik seviyesi (Urgent/High/Medium/Low) ve etiketler atamak istiyorum,*  
  *Böylece kritik işlerimi kolayca filtreleyip önceliklendirebileyim.*

#### Kabul Kriterleri:
```gherkin
Scenario: Göreve Geçerli Öncelik ve Etiket Ekleme (Happy Path)
  Given aktif bir görevim varken
  When önceliğini "URGENT" yapıp "finans" ve "rapor" etiketlerini iliştirdiğimde
  Then sistem HTTP 200 OK dönmelidir
  And görev önceliği güncellenmeli ve etiketler görevle ilişkilendirilmelidir.

Scenario: Maksimum Etiket Sınırının Aşılması (Sınır Durumu)
  Given üzerinde zaten 10 adet etiket bulunan bir görevim varken
  When 11. etiketi eklemeye çalıştığımda
  Then sistem HTTP 422 Unprocessable Entity dönmelidir
  And "Bir göreve en fazla 10 etiket iliştirilebilir" hata mesajı gösterilmelidir.
```

---

### [US-009] Görevi Silme ve Kaynak Temizliği
- **İlgili SRS Maddesi:** `FR-TASK-007`
- **Hikaye:**  
  *Bir **Kullanıcı** olarak,*  
  *Artık gerekmeyen bir görevi silmek istiyorum,*  
  *Böylece görev listem temiz kalsın ve silinen görev için bildirim almayayım.*

#### Kabul Kriterleri:
```gherkin
Scenario: Görevi Başarıyla Silme (Happy Path)
  Given bana ait aktif bir görev varken
  When silme komutu verdiğimde
  Then sistem HTTP 204 No Content dönmelidir
  And görev veritabanında mantıksal olarak silinmelidir (is_deleted = true)
  And BullMQ kuyruğundaki ilişkili bekleyen hatırlatıcı işi derhal silinmelidir.
```

---

## 4. ZAMAN PLANLAMA VE BİLDİRİM HİKAYELERİ (US-SCHED & US-NOTIF)

### [US-010] Göreve Vade Tarihi Atama ve Takvim Görünümü
- **İlgili SRS Maddeleri:** `FR-SCHED-001`, `FR-SCHED-002`
- **Hikaye:**  
  *Bir **Kullanıcı** olarak,*  
  *Görevlerime teslim tarihi atamak ve günlük/haftalık/aylık takvim görünümünde incelemek istiyorum,*  
  *Böylece zamanımı etkin planlayıp son teslim tarihlerini kaçırmayayım.*

#### Kabul Kriterleri:
```gherkin
Scenario: Gelecek Tarihe Vade Atama ve Takvimde Listeleme (Happy Path)
  Given aktif bir görevim varken
  When vade tarihini "2026-10-05 15:00" olarak seçtiğimde
  Then sistem HTTP 200 OK ile tarihi kaydetmelidir
  And "Haftalık Görünüm" takvim panosunu açtığımda görev 5 Ekim gününde P95 < 100ms sürede listelenmelidir.

Scenario: Geçmiş Bir Tarihe Vade Atama Denemesi (Hata Durumu)
  Given aktif bir görevim varken
  When vade tarihini geçmiş bir zaman seçtiğimde (due_date < NOW())
  Then sistem HTTP 400 Bad Request dönmelidir
  And "Vade tarihi geçmiş bir zaman olamaz" uyarısı gösterilmelidir.
```

---

### [US-011] Hatırlatıcı Kurma ve Asenkron Bildirim İletimi
- **İlgili SRS Maddeleri:** `FR-NOTIF-001`, `FR-NOTIF-002`, `FR-NOTIF-003`
- **Hikaye:**  
  *Bir **Kullanıcı** olarak,*  
  *Görevimin tesliminden önce bildirim (E-posta veya Push) almak istiyorum,*  
  *Böylece zamanında harekete geçebileyim.*

#### Kabul Kriterleri:
```gherkin
Scenario: Hatırlatıcı Zamanı Geldiğinde Asenkron Bildirim İletimi (Happy Path)
  Given vade tarihi olan ve henüz tamamlanmamış bir görevim için 1 saat öncesine hatırlatıcı kurulmuşken
  When hatırlatma zamanı geldiğinde (ACT-02 Worker tetiklendiğinde)
  Then BullMQ worker görevin tamamlanmadığını teyit etmelidir
  And Amazon SES veya Web Push sağlayıcısına asenkron bildirim fırlatmalıdır
  And dış sağlayıcıdan 200 OK alındığında bildirim "Delivered" olarak işaretlenmelidir.

Scenario: Dış Bildirim Servisi Arızasında Üstel Geri Çekilme (Hata ve Kurtarma)
  Given hatırlatıcı işi çalıştırıldığında dış bildirim ağ geçidi HTTP 503 Service Unavailable dönerse
  When sistem hatayı algıladığında
  Then işi sonlandırmayıp 1 dk, 5 dk ve 15 dk aralıklarla en fazla 3 kez yeniden denemelidir (Retry)
  And 3 denemenin sonunda hala hata alınıyorsa işi Dead-Letter Queue'ya (DLQ) taşımalı ve hata günlüğüne yazmalıdır.
```

---

## 5. KULLANICI PROFİLİ VE MEVZUAT UYUM HİKAYELERİ (US-PROF)

### [US-012] Kişisel Verileri Dışa Aktarma (Data Portability)
- **İlgili SRS Maddesi:** `FR-PROF-001`
- **Hikaye:**  
  *Bir **Kullanıcı** olarak,*  
  *Sistemdeki tüm görev, kategori ve etiket geçmişimi tek bir dosyada indirmek istiyorum,*  
  *Böylece GDPR Madde 20 gereği veri taşınabilirliği hakkımı kullanabileyim.*

#### Kabul Kriterleri:
```gherkin
Scenario: Tüm Görev Verilerini JSON Formatında İndirme (Happy Path)
  Given hesabımda 150 adet görev ve etiket geçmişim varken
  When "Verilerimi Dışa Aktar" butonuna tıklayıp JSON formatını seçtiğimde
  Then sistem P95 < 2 saniye içinde arşiv dosyasını üretmelidir
  And HTTP 200 OK ile standart formatlı JSON indirme akışı başlatmalıdır.

Scenario: 24 Saatte 3'ten Fazla Dışa Aktarma Denemesi (Kota Sınırı)
  Given son 24 saat içinde 3 kez başarılı veri indirme işlemi yapmışken
  When 4. kez dışa aktarma talebi gönderdiğimde
  Then sistem HTTP 429 Too Many Requests dönmelidir
  And "Günlük veri indirme limitine ulaştınız, lütfen yarın tekrar deneyin" mesajı gösterilmelidir.
```

---

### [US-013] Hesap Silme ve 30 Günlük Unutulma Hakkı (Right to Erasure)
- **İlgili SRS Maddeleri:** `FR-PROF-002`, `FR-PROF-003`
- **Hikaye:**  
  *Bir **Kullanıcı** olarak,*  
  *Hesabımın ve tüm kişisel verilerimin sistemden kalıcı olarak silinmesini talep etmek istiyorum,*  
  *Böylece KVKK Md. 7 ve GDPR Art. 17 uyarınca unutulma hakkımı kullanabileyim.*

#### Kabul Kriterleri:
```gherkin
Scenario: Parola Onayı ile Hesap Silme Talebi Başlatma (Happy Path)
  Given aktif bir hesabım varken
  When hesap ayarlarından doğru parolamı girerek "Hesabımı Sil" onayını verdiğimde
  Then sistem HTTP 200 OK dönmelidir
  And hesabımı "PENDING_DELETION" durumuna alıp anında oturumumu kapatmalıdır
  And 30 günlük yasal cayma süresi (grace period) başlatılmalıdır.

Scenario: 30 Gün Sonunda Fiziksel Kalıcı İmha (Otonom Zamanlayıcı Akışı)
  Given kullanıcının hesap silme talebinden bu yana tam 30 gün (720 saat) geçmişken
  When ACT-02 Gece İmha Worker'ı çalıştığında
  Then kullanıcının tüm kişisel verilerini, görevlerini ve ilişkili kayıtlarını fiziksel olarak silmelidir (Hard Delete)
  And kullanıcının kriptografik şifreleme anahtarlarını imha etmelidir (Crypto-shredding).
```

---

## 6. DOĞRULAMA VE KALİTE KAPISI (QUALITY GATE SIGN-OFF)

- [x] **Tüm "Must" Gereksinimleri Kapsandı:** `FR-AUTH` (4), `FR-TASK` (5), `FR-SCHED` (2), `FR-NOTIF` (3), `FR-PROF` (3) olmak üzere toplam **17 temel gereksinim** 13 detaylı BDD hikayesine dönüştürüldü.
- [x] **En Az 2 Senaryo Kuralı:** Her kullanıcı hikayesinde en az 1 Happy Path ve en az 1 Edge-Case / Hata senaryosu Gherkin formatında kodlandı.
- [x] **INVEST Standartları:** Hikayelerin bağımsızlığı, değer üretmesi ve test edilebilirliği sağlandı.
- [x] **TDD & BDD Hazırlığı:** Senaryolar Jest ve Playwright acceptance testlerine birebir aktarılabilir niteliktedir.

**Agile Koç & İş Analisti:** Agile Coach  
**Statü:** **ONAYLANDI (P2 Aşama 02 Adım A4 Tamamlandı — Adım A5 Abuse Cases & RTM Matrisine Geçişe Hazır)**
