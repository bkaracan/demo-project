# YAZILIM GEREKSİNİM ŞARTNAMESİ (SOFTWARE REQUIREMENTS SPECIFICATION - SRS)

**Proje:** demo-project — Bireysel Zaman & Görev Yönetim Platformu (To-Do App)  
**Standart:** IEEE 830-1998 Standart Formatı  
**Tarih:** 19 Eylül 2026  
**Sürüm:** v1.0.0  
**SDLC Fazı:** Aşama 02 (Gereksinim Analizi) — Adım A2 & A3 (Fonksiyonel & NFR Gereksinimleri)  
**Girdi Belgeleri:** [PROJECT_CHARTER.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/PROJECT_CHARTER.md), [DOMAIN_EVENTS.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/DOMAIN_EVENTS.md)  
**Statü:** **ONAYLANDI (Quality Gate Geçildi)**  

---

## 1. GİRİŞ (INTRODUCTION)

### 1.1. Belgenin Amacı
Bu Yazılım Gereksinim Şartnamesi (SRS), `demo-project` bünyesinde geliştirilecek olan modern bireysel zaman ve görev yönetim sisteminin (To-Do App) fonksiyonel gereksinimlerini, iş kurallarını, aktör yetkilerini ve istisna yönetim protokollerini IEEE 830 standardına tam uyumlu olarak tanımlar.

### 1.2. Ürün Kapsamı
Platform; bireysel son kullanıcıların günlük, haftalık ve aylık zaman programları oluşturmasını, görevlerini önceliklendirip etiketlemesini ve gecikmesiz hatırlatıcılarla üretkenliklerini optimize etmesini sağlayan yüksek performanslı bir Web / PWA çözümüdür.

### 1.3. Tanımlar ve Kısaltmalar
- **SRS:** Software Requirements Specification (Yazılım Gereksinim Şartnamesi)
- **MoSCoW:** Must have (Zorunlu), Should have (Önemli), Could have (İsteğe Bağlı), Won't have (Kapsam Dışı)
- **JWT:** JSON Web Token (RFC 7519)
- **IDOR:** Insecure Direct Object Reference (Yetkisiz Nesne Erişimi)
- **TTL:** Time-To-Live (Önbellek Yaşam Süresi)
- **SLO / SLI:** Service Level Objective / Indicator

---

## 2. GENEL SİSTEM TANIMI (OVERALL DESCRIPTION)

### 2.1. Ürün Perspektifi ve Bounded Contexts
Sistem 4 temel Bounded Context üzerinden hizmet verir:
1. **Identity & Profile:** Kimlik, güvenlik, veri dışa aktarma ve KVKK/GDPR imhası.
2. **Task Lifecycle:** Görev oluşturma, güncelleme, önceliklendirme, etiketleme ve tamamlama.
3. **Schedule & Calendar:** Günlük/haftalık/aylık zaman çizelgeleme ve takvim görünümü.
4. **Notification:** BullMQ asenkron kuyruğu ve harici e-posta/push iletimi.

### 2.2. Kullanıcı Rolleri ve Aktörler
- **Bireysel Kullanıcı (ACT-01):** Kendi görevlerini yöneten doğrulanmış son kullanıcı.
- **Zamanlayıcı / Worker (ACT-02):** Hatırlatıcıları ve veri imha periyotlarını işleten sistem aktörü.
- **Bildirim Gateway (ACT-03):** E-posta/Push bildirim iletimini sağlayan dış sistem.
- **Sistem Yöneticisi (ACT-04):** Sistem sağlığını ve denetim loglarını izleyen teknik yönetici.

---

## 3. FONKSİYONEL GEREKSİNİMLER (FUNCTIONAL REQUIREMENTS)

### 3.1. Kimlik Doğrulama ve Hesap Güvenliği (FR-AUTH)

| Gereksinim ID | Öncelik (MoSCoW) | Gereksinim Tanımı (Sistem Kuralı) | İstisna ve Hata Yönetimi (Edge-Cases) |
| :---: | :---: | :--- | :--- |
| **FR-AUTH-001** | **Must** | Sistem, **ACT-01 (Kullanıcı)** e-posta ve parola ile kayıt olma isteği gönderdiğinde; e-postanın RFC 5322 formatında olduğunu, parolanın en az 8 karakter uzunluğunda olup en az bir büyük harf, bir rakam ve bir özel karakter içerdiğini doğrulamalı ve parolayı doğrudan **Argon2id** (`m=65536, t=3, p=4`) ile hash'leyerek yeni kullanıcı kaydı oluşturmalıdır. | **EX-AUTH-001A:** E-posta veritabanında zaten kayıtlıysa sistem `HTTP 409 Conflict` ve "Bu e-posta adresi zaten kullanımda" hatası dönmelidir.<br>**EX-AUTH-001B:** Parola karmaşıklık kurallarına uyulmadığında sistem `HTTP 422 Unprocessable Entity` ve eksik kural listesini dönmelidir. |
| **FR-AUTH-002** | **Must** | Sistem, **ACT-01 (Kullanıcı)** geçerli kimlik bilgileriyle oturum açma isteği gönderdiğinde; kullanıcının kimliğini doğrulamalı, 15 dakika geçerlilik süresine sahip bir JWT Access Token ile 7 gün geçerli bir Refresh Token üretmeli ve bunları `HttpOnly`, `Secure`, `SameSite=Strict` bayraklarıyla çerezde saklamalıdır. | **EX-AUTH-002A:** E-posta veya parola hatalı olduğunda sistem `HTTP 401 Unauthorized` ve genel "Geçersiz kimlik bilgileri" mesajı dönmelidir (kullanıcı varlığı ifşa edilmez). |
| **FR-AUTH-003** | **Must** | Sistem, **ACT-01 (Kullanıcı)** veya harici bir istemciden gelen oturum açma denemelerinde son 60 saniye içinde aynı IP adresinden 5 başarısız deneme yapıldığını tespit ettiğinde; ilgili IP adresinden gelen tüm oturum açma isteklerini 15 dakika boyunca engellemelidir. | **EX-AUTH-003A:** Engelli IP'den istek geldiğinde sistem `HTTP 429 Too Many Requests` ve kalan bekleme süresini içeren `Retry-After` başlığı döndürmelidir. |
| **FR-AUTH-004** | **Must** | Sistem, **ACT-01 (Kullanıcı)** oturumu kapatma isteği gönderdiğinde; istemcideki kimlik doğrulama çerezlerini sıfırlamalı ve mevcut Refresh Token kaydını Valkey önbelleğindeki kara listeye (blacklist) ekleyerek anında geçersiz kılmalıdır. | **EX-AUTH-004A:** Geçersiz veya süresi dolmuş token ile oturum kapatma çağrıldığında sistem `HTTP 401 Unauthorized` dönmelidir. |
| **FR-AUTH-005** | **Should** | Sistem, **ACT-01 (Kullanıcı)** şifre sıfırlama talebinde bulunduğunda; kullanıcının e-posta adresine 30 dakika geçerli tek kullanımlık kriptografik bir bağlantı (token) göndermeli ve yeni şifre belirlendiğinde kullanıcının tüm aktif oturumlarını geçersiz kılmalıdır. | **EX-AUTH-005A:** 30 dakikayı aşmış veya önceden kullanılmış token ile şifre yenileme denendiğinde `HTTP 400 Bad Request` dönmelidir. |

---

### 3.2. Görev ve Yaşam Döngüsü Yönetimi (FR-TASK)

| Gereksinim ID | Öncelik (MoSCoW) | Gereksinim Tanımı (Sistem Kuralı) | İstisna ve Hata Yönetimi (Edge-Cases) |
| :---: | :---: | :--- | :--- |
| **FR-TASK-001** | **Must** | Sistem, **ACT-01 (Kullanıcı)** yeni bir görev oluşturma isteği gönderdiğinde; görev başlığının 1 ile 255 karakter aralığında olduğunu doğrulamalı, görevi varsayılan olarak `TODO` durumunda ve `MEDIUM` öncelikle veritabanına kaydetmeli ve benzersiz görev ID'si ile `HTTP 201 Created` yanıtı üretmelidir. | **EX-TASK-001A:** Başlık boş bırakıldığında veya 255 karakteri aştığında sistem `HTTP 400 Bad Request` dönmelidir.<br>**EX-TASK-001B:** Kullanıcının aktif görev sayısı 1.000 sınırını aşarsa sistem `HTTP 422 Unprocessable Entity` ve limit aşım uyarısı dönmelidir. |
| **FR-TASK-002** | **Must** | Sistem, **ACT-01 (Kullanıcı)** bir görevi güncelleme veya silme isteği gönderdiğinde; görevin sahibinin oturum açmış kullanıcı olduğunu (`task.user_id == authenticated_user.id`) doğrulamalı ve eşleşme sağlandığında güncellemeyi/silmeyi işletmelidir. | **EX-TASK-002A:** Görev başka bir kullanıcıya aitse sistem `HTTP 403 Forbidden` veya varlığı gizlemek için `HTTP 404 Not Found` dönmelidir (IDOR Koruması). |
| **FR-TASK-003** | **Must** | Sistem, **ACT-01 (Kullanıcı)** bir görevin durumunu `IN_PROGRESS` veya `COMPLETED` olarak güncellediğinde; mevcut durumun `ARCHIVED` olmadığını doğrulamalı, geçerli duruma geçişi işletmeli ve durum `COMPLETED` olduğunda `completed_at = CURRENT_TIMESTAMP` damgasını vurmalıdır. | **EX-TASK-003A:** Görev zaten `ARCHIVED` durumundaysa sistem `HTTP 400 Bad Request` ve "Arşivlenmiş görev güncellenemez, önce arşivden çıkarılmalıdır" hatası dönmelidir. |
| **FR-TASK-004** | **Must** | Sistem, **ACT-01 (Kullanıcı)** bir göreve öncelik atadığında; öncelik değerinin `LOW`, `MEDIUM`, `HIGH` veya `URGENT` değerlerinden biri olduğunu doğrulamalı ve görevin öncelik alanını güncellemelidir. | **EX-TASK-004A:** Tanımsız bir öncelik değeri gönderildiğinde sistem `HTTP 400 Bad Request` dönmelidir. |
| **FR-TASK-005** | **Must** | Sistem, **ACT-01 (Kullanıcı)** bir göreve etiket ekleme isteği gönderdiğinde; etiket adının 1 ile 30 karakter arasında alfasayısal olduğunu, göreve iliştirilmiş etiket sayısının 10'u aşmadığını doğrulamalı ve etiketi göreve bağlamalıdır. | **EX-TASK-005A:** Aynı etiket mükerrer eklenmek istendiğinde sistem işlemi yoksaymalı (idempotent) veya `HTTP 409 Conflict` dönmelidir.<br>**EX-TASK-005B:** Etiket sayısı 10'u aştığında sistem `HTTP 422 Unprocessable Entity` dönmelidir. |
| **FR-TASK-006** | **Should** | Sistem, **ACT-01 (Kullanıcı)** `COMPLETED` durumundaki bir görevi arşivlemek istediğinde; görevin durumunu `ARCHIVED` olarak işaretlemeli ve aktif günlük/haftalık listelerden gizleyerek arşiv deposuna taşımalıdır. | **EX-TASK-006A:** Görev henüz tamamlanmamışsa (`TODO` veya `IN_PROGRESS`) sistem `HTTP 400 Bad Request` ve "Yalnızca tamamlanmış görevler arşivlenebilir" uyarısı dönmelidir. |
| **FR-TASK-007** | **Must** | Sistem, **ACT-01 (Kullanıcı)** bir görevi sildiğinde; veritabanında mantıksal silme (`is_deleted = true, deleted_at = NOW()`) uygulamalı ve bu göreve bağlı bekleyen tüm BullMQ hatırlatıcı işlerini kuyruktan derhal temizlemelidir. | **EX-TASK-007A:** Zaten silinmiş bir görev tekrar silinmek istendiğinde sistem `HTTP 404 Not Found` dönmelidir. |

---

### 3.3. Zaman Planlama ve Takvim Yönetimi (FR-SCHED)

| Gereksinim ID | Öncelik (MoSCoW) | Gereksinim Tanımı (Sistem Kuralı) | İstisna ve Hata Yönetimi (Edge-Cases) |
| :---: | :---: | :--- | :--- |
| **FR-SCHED-001** | **Must** | Sistem, **ACT-01 (Kullanıcı)** bir göreve vade tarihi (`due_date`) atadığında; belirtilen zaman damgasının mevcut sistem zamanından ileri bir tarih olduğunu (`due_date >= CURRENT_TIMESTAMP`) doğrulamalı ve görevi kullanıcının takvim panosuna işlemelidir. | **EX-SCHED-001A:** Geçmiş bir tarih seçildiğinde sistem `HTTP 400 Bad Request` ve "Vade tarihi geçmiş bir zaman olamaz" hatası dönmelidir. |
| **FR-SCHED-002** | **Must** | Sistem, **ACT-01 (Kullanıcı)** takvim panosunu sorguladığında; görünüm moduna göre (`daily`, `weekly`, `monthly`) kullanıcının ilgili zaman aralığındaki tüm aktif görevlerini önbellekten (Valkey) veya DB indeksinden P95 < 100ms sürede getirmelidir. | **EX-SCHED-002A:** Geçersiz tarih aralığı veya desteklenmeyen görünüm parametresi girildiğinde sistem `HTTP 400 Bad Request` dönmelidir. |
| **FR-SCHED-003** | **Should** | Sistem, **ACT-01 (Kullanıcı)** bir görevin vade tarihini değiştirdiğinde (yeniden planlama); eski tarihe ait BullMQ hatırlatma işini iptal etmeli, yeni tarihe göre gecikmeli işi yeniden kuyruğa eklemelidir. | **EX-SCHED-003A:** Yeni tarih geçmiş bir zaman ise güncelleme reddedilmeli ve eski tarih korunmalıdır. |

---

### 3.4. Hatırlatıcı ve Asenkron Bildirimler (FR-NOTIF)

| Gereksinim ID | Öncelik (MoSCoW) | Gereksinim Tanımı (Sistem Kuralı) | İstisna ve Hata Yönetimi (Edge-Cases) |
| :---: | :---: | :--- | :--- |
| **FR-NOTIF-001** | **Must** | Sistem, **ACT-01 (Kullanıcı)** bir görev için hatırlatıcı kurduğunda; hatırlatıcı zamanının görevin vade tarihinden önce veya vade anında olduğunu doğrulamalı ve BullMQ gecikmeli kuyruğuna hedef milisaniye farkı ile iş kaydetmelidir. | **EX-NOTIF-001A:** Hatırlatıcı zamanı vade tarihinden sonraki bir zaman olarak girilirse sistem `HTTP 400 Bad Request` dönmelidir. |
| **FR-NOTIF-002** | **Must** | Sistem, **ACT-02 (Zamanlayıcı)** hatırlatma zamanı gelen bir kuyruk işini işlediğinde; görevin silinmediğini ve henüz tamamlanmadığını (`status != COMPLETED`) doğrulamalı ve kullanıcının tercih ettiği kanaldan (E-posta / Web Push) asenkron bildirim fırlatmalıdır. | **EX-NOTIF-002A:** Görev kullanıcı tarafından önceden tamamlanmış veya silinmişse worker bildirimi iletmeden işi sessizce başarıyla sonlandırmalıdır (Discard). |
| **FR-NOTIF-003** | **Must** | Sistem, **ACT-03 (Bildirim Gateway)** e-posta veya push gönderiminde hata döndüğünde (HTTP 5xx / Network Timeout); üstel geri çekilme (1 dakika, 5 dakika, 15 dakika) ile işi en fazla 3 kez yeniden denemeli, 3. başarısızlık sonrasında işi Dead-Letter Queue'ya (DLQ) taşımalıdır. | **EX-NOTIF-003A:** Kullanıcının e-posta adresi kalıcı olarak reddedilirse (Hard Bounce); sistem e-posta gönderimini devre dışı bırakmalı ve arayüzde uyarı göstermelidir. |
| **FR-NOTIF-004** | **Could** | Sistem, **ACT-01 (Kullanıcı)** bildirim tercihlerini güncellediğinde; kullanıcının günlük özet e-postası (Daily Digest) alıp almayacağını ve sessiz saatler (Quiet Hours) aralığını profiline kaydetmelidir. | **EX-NOTIF-004A:** Başlangıç saati bitiş saatiyle aynı girildiğinde `HTTP 400 Bad Request` dönmelidir. |

---

### 3.5. Kullanıcı Profili, Veri Taşınabilirliği ve Unutulma Hakkı (FR-PROF)

| Gereksinim ID | Öncelik (MoSCoW) | Gereksinim Tanımı (Sistem Kuralı) | İstisna ve Hata Yönetimi (Edge-Cases) |
| :---: | :---: | :--- | :--- |
| **FR-PROF-001** | **Must** | Sistem, **ACT-01 (Kullanıcı)** verilerini dışa aktarma (GDPR Art. 20 Data Portability) isteği gönderdiğinde; kullanıcının tüm profil, görev, kategori ve etiket geçmişini içeren standart JSON veya CSV dosyasını P95 < 2 saniyede üreterek tekil indirme bağlantısı sağlamalıdır. | **EX-PROF-001A:** Kullanıcı son 24 saat içinde 3'ten fazla dışa aktarma talebinde bulunursa sistem `HTTP 429 Too Many Requests` ve kota aşım mesajı dönmelidir. |
| **FR-PROF-002** | **Must** | Sistem, **ACT-01 (Kullanıcı)** hesabını silme (KVKK Md. 7 / GDPR Art. 17 Unutulma Hakkı) talebinde bulunduğunda; kullanıcının parolasını teyit ettikten sonra hesabı anında devre dışı bırakmalı (`status = PENDING_DELETION`), oturumlarını sonlandırmalı ve 30 günlük yasal cayma süresi başlatmalıdır. | **EX-PROF-002A:** Parola doğrulaması başarısız olursa sistem `HTTP 401 Unauthorized` dönmeli ve hesap silme sürecini başlatmamalıdır. |
| **FR-PROF-003** | **Must** | Sistem, **ACT-02 (Zamanlayıcı)** günlük imha worker'ı çalıştığında; `PENDING_DELETION` durumunda 30 gününü (720 saat) dolduran kullanıcıların kişisel verilerini, görevlerini ve ilişkili loglarını veritabanından kalıcı olarak silmeli (Hard Delete / Crypto-shredding) ve işlem denetim kaydını anonim olarak mühürlemelidir. | **EX-PROF-003A:** Henüz 30 günü dolmamış kullanıcılar için silme işlemi atlanmalı ve cayma süresi beklenmelidir. |

---

### 3.6. Kapsam Dışı Bırakılan Fonksiyonlar (Won't Have - FR-OOS)

Charter ve Fizibilite kararları doğrultusunda aşağıdaki gereksinimler ilk sürümde (MVP) **KESİNLİKLE UYGULANMAYACAKTIR**:

| Gereksinim ID | Öncelik | Kapsam Dışı Başlığı | Açıklama ve Gerekçe |
| :---: | :---: | :--- | :--- |
| **FR-OOS-001** | **Won't** | Takım ve Çok Kullanıcılı Ortak Çalışma Alanı | Proje bireysel üretkenliğe odaklıdır; kullanıcılar arası görev atama veya paylaşımlı pano yapılamaz. |
| **FR-OOS-002** | **Won't** | Ödeme Ağ Geçidi ve Abonelik Modülü | Stripe/Iyzico faturalandırma ve ücretli paketler kapsam dışıdır; PCI-DSS muafiyetini korur. |
| **FR-OOS-003** | **Won't** | Yapay Zeka ile Otopilot Görev Çizelgeleme | Otonom takvim yerleşimi veya yapay zeka algoritması bu sürümde yer almayacaktır. |
| **FR-OOS-004** | **Won't** | 2 Yönlü Harici Takvim Canlı Senkronizasyonu | Google Calendar / Outlook canlı çift yönlü senkronizasyonu geliştirilmeyecektir. |
| **FR-OOS-005** | **Won't** | Yerel Masaüstü Binary Uygulamaları | Electron/Native derleme binary'leri yapılmayacak; standart duyarlı Web/PWA geliştirilecektir. |

---

## 4. FONKSİYONEL OLMAYAN GEREKSİNİMLER (NON-FUNCTIONAL REQUIREMENTS - NFR) & SAYISAL SLO

Sistem kalitesi, ölçeklenebilirliği, güvenliği ve dayanıklılığı aşağıdaki 5 temel eksende sayısal, doğrulanabilir ve test edilebilir eşiklerle belirlenmiştir:

### 4.1. Performans ve Yanıt Süresi (Latency & Throughput)

| NFR ID | Metrik / Nitelik | Sayısal Hedef Eşik (SLO) | İşlem Koşulu ve Kapsam | Doğrulama & Test Yöntemi |
| :---: | :--- | :--- | :--- | :--- |
| **NFR-PERF-001** | API P95 Yanıt Süresi | **P95 < 150 ms** (Normal yükte < 100 ms) | 1.000 eşzamanlı sanal kullanıcı (VU) altında tüm `GET /api/v1/tasks` ve `GET /api/v1/schedule` sorguları | K6 yük testi (`k6 run --vus 1000 --duration 10m`) ve Prometheus telemetrisi ile doğrulanacaktır. |
| **NFR-PERF-002** | API P99 Yanıt Süresi | **P99 < 400 ms** | 1.000 eşzamanlı istek altında en yavaş %1'lik dilimdeki karmaşık filtreleme sorguları | K6 stres testi ve APM OpenTelemetry izleri (distributed tracing) ile doğrulanacaktır. |
| **NFR-PERF-003** | Eşzamanlı İşlem Yükü (Throughput) | **>= 500 RPS** (Pik: 1.000 RPS) | Sistemde hata oranı (HTTP 5xx) < %0.1 olacak şekilde sürekli yük altında | K6 sabit throughput senaryosu (`constant-arrival-rate: 500`) ile doğrulanacaktır. |
| **NFR-PERF-004** | İlk İçerikli Boyama (FCP) & LCP | **FCP < 1.2 sn, LCP < 2.5 sn** | 4G mobil ağ bağlantısında PWA / Web arayüz ilk yükleme performansı | Google Lighthouse CI ve Web Vitals test otomasyonu ile doğrulanacaktır. |

### 4.2. Erişilebilirlik ve Güvenilirlik (Availability & SLA)

| NFR ID | Metrik / Nitelik | Sayısal Hedef Eşik (SLO) | İşlem Koşulu ve Kapsam | Doğrulama & Test Yöntemi |
| :---: | :--- | :--- | :--- | :--- |
| **NFR-AVAIL-001** | Sistem Uptime (Hizmet Süresi) | **%99.9 Uptime** | Ayda en fazla **43.8 dakika** plansız kesinti toleransı (7/24 bazında) | Harici Uptime Robot / Datadog sentetik monitörleri ile 60 saniyede bir doğrulanacaktır. |
| **NFR-AVAIL-002** | Hata Bütçesi (Error Budget) | **Aylık max %0.1 Hata Oranı** | Toplam HTTP istekleri içinde 5xx yanıtlarının oranı <= %0.1 | Grafana SLO Panosu ve Alertmanager kuralı ile otomatik alarm tetiklenecektir. |

### 4.3. Güvenlik, Kriptografi ve Uyum (Security & Privacy)

| NFR ID | Metrik / Nitelik | Sayısal Hedef Eşik (SLO) | İşlem Koşulu ve Kapsam | Doğrulama & Test Yöntemi |
| :---: | :--- | :--- | :--- | :--- |
| **NFR-SEC-001** | İletimde Şifreleme (In-Transit) | **TLS 1.3 Zorunlu** (Min. TLS 1.2) | Tüm HTTP trafiği HTTPS'e yönlendirilmeli, HSTS `max-age=31536000` aktif olmalıdır. | SSL Labs API testi ile A+ skoru ve `curl -v` SSL el sıkışma doğrulaması yapılacaktır. |
| **NFR-SEC-002** | Durağan Veri Şifreleme (At-Rest) | **AES-256 (KMS Entegre)** | PostgreSQL veri tabloları, WAL logları, Redis/Valkey dump'ları ve S3 yedekleri | Terraform IaC güvenlik denetimi (`checkov` / `tfsec`) ile doğrulanacaktır. |
| **NFR-SEC-003** | Parola ve Kimlik Güvenliği | **Argon2id Kriptografik Karma** | Parametreler: `m=65536` (64MB), `t=3` (3 iterasyon), `p=4` (4 paralellik). | OWASP ASVS Seviye 2 doğrulama testleri ve birim testler ile doğrulanacaktır. |
| **NFR-SEC-004** | Web Güvenlik Standartları | **OWASP ASVS Seviye 2 Uyumu** | XSS, CSRF, SQL Injection, IDOR, SSRF zafiyetlerine karşı sıfır tolerans | OWASP ZAP DAST tarayıcısı ve SonarQube SAST taraması ile CI hattında doğrulanacaktır. |

### 4.4. Felaket Kurtarma ve İş Sürekliliği (Disaster Recovery)

| NFR ID | Metrik / Nitelik | Sayısal Hedef Eşik (SLO) | İşlem Koşulu ve Kapsam | Doğrulama & Test Yöntemi |
| :---: | :--- | :--- | :--- | :--- |
| **NFR-DR-001** | Kurtarma Noktası Hedefi (RPO) | **RPO < 5 Dakika** | Fiziksel veri merkezi veya bölge felaketinde kabul edilebilir maksimum veri kaybı süresi | PostgreSQL RDS Point-in-Time Recovery (PITR) ve sürekli WAL arşivleme simülasyonu ile doğrulanacaktır. |
| **NFR-DR-002** | Kurtarma Zamanı Hedefi (RTO) | **RTO < 30 Dakika** | Kesinti anından itibaren sistemin tüm servisleriyle ayağa kaldırılma süresi | 6 aylık periyodik Chaos Engineering ve Multi-AZ otomatik failover tatbikatı ile doğrulanacaktır. |

### 4.5. Erişilebilirlik ve Kullanıcı Deneyimi (Accessibility & UX)

| NFR ID | Metrik / Nitelik | Sayısal Hedef Eşik (SLO) | İşlem Koşulu ve Kapsam | Doğrulama & Test Yöntemi |
| :---: | :--- | :--- | :--- | :--- |
| **NFR-ACC-001** | Dijital Erişilebilirlik Standardı | **WCAG 2.1 AA Seviyesi** | Tüm kullanıcı arayüzlerinde metin ve arka plan kontrast oranı en az **4.5:1** olmalıdır. | `axe-core` otomatik test aracı ve Pa11y CI entegrasyonu ile doğrulanacaktır. |
| **NFR-ACC-002** | Ekran Okuyucu ve Klavye Uyumu | **%100 Klavye ile Gezinilebilirlik** | Tüm görev oluşturma, tamamlama ve tarih seçme akışları sadece `Tab`, `Enter`, `Space` ile yönetilebilmelidir. | Playwright klavye navigasyon senaryoları ve NVDA / VoiceOver ekran okuyucu testleri ile doğrulanacaktır. |

---

## 5. DOĞRULAMA VE KALİTE KAPISI (QUALITY GATE SIGN-OFF)

- [x] **IEEE 830 Standart Formatı:** Bölüm 1 (Giriş), Bölüm 2 (Genel Tanım), Bölüm 3 (Fonksiyonel) ve Bölüm 4 (NFR) eksiksiz yapılandırıldı.
- [x] **Modüler NFR ID Standardı:** `NFR-PERF-XXX`, `NFR-AVAIL-XXX`, `NFR-SEC-XXX`, `NFR-DR-XXX`, `NFR-ACC-XXX` tanımlandı.
- [x] **Sayısal ve Ölçülebilir Eşikler:** Sübjektif sıfatlar kullanılmadı; P95 < 150ms, P99 < 400ms, %99.9 Uptime, RPO < 5dk, RTO < 30dk, WCAG 2.1 AA değerleri bağlandı.
- [x] **Somut Doğrulama Yöntemleri:** Her NFR maddesi için K6, ZAP, SSL Labs, axe-core ve PITR gibi doğrulama araçları belirtildi.
- [x] **Charter & FinOps Uyumu:** Belirlenen SLO kriterlerinin $500/ay bulut altyapı bütçesiyle tam uyumlu olduğu teyit edildi.

**SRE & Sistem Baş Mimarı:** SRE Architect  
**Statü:** **ONAYLANDI (P2 Aşama 02 Adım A3 Tamamlandı — Adım A4 User Stories & BDD Senaryolarına Geçişe Hazır)**
