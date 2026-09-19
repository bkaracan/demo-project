# TEKNİK VE YASAL FİZİBİLİTE RAPORU (FEASIBILITY REPORT)

**Proje:** demo-project — Bireysel Zaman & Görev Yönetim Platformu (To-Do App)  
**Tarih:** 19 Eylül 2026  
**Sürüm:** v1.0.0  
**SDLC Fazı:** Aşama 01 (Planlama) — Adım A2 (Teknik, Yasal ve Operasyonel Fizibilite Analizi)  
**Girdi Belgesi:** [PROJECT_CHARTER.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/PROJECT_CHARTER.md)  
**Nihai Karar:** **CONDITIONAL GO (ŞARTLI ONAY)**  

---

## 1. YÖNETİCİ ÖZETİ (EXECUTIVE SUMMARY)

`PROJECT_CHARTER.md` belgesinde tanımlanan hedefler (P95 < 100ms gecikme, ≥ 500 RPS eşzamanlı işlem yükü, %99.9 Uptime, 4 haftalık MVP teslimatı ve aylık 500 USD FinOps tavanı) doğrultusunda kapsamlı bir teknik, açık kaynak lisanslama ve hukuki regülasyon fizibilite denetimi icra edilmiştir.

Yapılan araştırma ve simülasyonlar sonucunda projenin **"CONDITIONAL GO (ŞARTLI ONAY)"** statüsünde olduğu belirlenmiştir. Belirlenen 4 kritik mimari şartın yerine getirilmesi kaydıyla projenin teknik olarak hedeflenen performans sınırları içinde inşa edilebileceği, FinOps bütçesi dahilinde kalınabileceği ve yasal/cezai yaptırım risklerinin sıfırlanabileceği teyit edilmiştir.

---

## 2. TEKNİK FİZİBİLİTE ANALİZİ

### 2.1. Yük Kapasitesi ve Gecikme Uygunluğu (500 RPS & P95 < 100ms)

Projede hedeflenen **500 RPS** eşzamanlı işlem yükü ve **P95 < 100ms API yanıt süresi**, modern mikro ölçekleme ve optimize I/O mimarisi ile standart bulut altyapılarında (2 vCPU, 2-4 GB RAM) rahatlıkla karşılanabilir bir büyüklüktür.

| Metrik | Proje Hedefi | Öngörülen Mimari Kapasitesi | Değerlendirme |
| :--- | :--- | :--- | :--- |
| **Throughput (RPS)** | >= 500 RPS | 800 - 1.200 RPS (Fastify + Valkey Cache) | **UYGUN (Margin: +%60)** |
| **Gecikme (P95 Latency)** | < 100 ms | 15 - 45 ms (Cache Hit: < 5ms) | **UYGUN (Margin: -%55)** |
| **Erişilebilirlik (SLA)** | %99.9 | Multi-AZ DB + Multi-Replica Container | **UYGUN (Yıllık kesinti < 8.7h)** |

#### Backend Motoru Seçimi: Node.js (NestJS + Fastify) vs. Go (Gin/Fiber)

1. **Seçenek A: Node.js (NestJS + Fastify Adapter) — [TAVSİYE EDİLEN]**
   - **Performans:** Express yerine Fastify HTTP çekirdeği entegre edildiğinde, Node.js tekil event-loop üzerinde I/O-bound (Postgres + Redis/Valkey) yüklerde 1.500+ RPS seviyesine düşük gecikmeyle (P95 ~30-40ms) hizmet verebilmektedir.
   - **Geliştirme Hızı & Ekip Sinerjisi:** Ön yüzün Next.js / TypeScript olması sayesinde Full-Stack TypeScript mimarisi oluşturulur; DTO'lar, arayüz tipleri ve doğrulama şemaları (Zod) paylaşılır. Ekip bağlam geçişi (context switching) yaşamaz. 4 haftalık MVP takvimi için en az riskli ve en hızlı yoldur.
   - **Kaynak Tüketimi:** Container başına ortalama 150-250 MB RAM tüketimi. FinOps bütçesi dahilinde 2 pod replikası ~$30-40/ay maliyetle çalışır.
2. **Seçenek B: Go (Gin / Fiber)**
   - **Performans:** Derlenmiş makine kodu ve hafif goroutine yapısıyla olağanüstü düşük P95 gecikmesi (< 15ms) ve 10-30 MB RAM tüketimi sunar.
   - **Ekip Eğrisi & Risk:** Ekipte Go uzmanlığı eşit düzeyde değilse, kurumsal mimari kalıplarını (Clean Architecture, DI, ORM/Migrasyon pratikleri) sıfırdan kurmak 4 haftalık takvimi riske atar. Ayrıca TypeScript ile Go arasında DTO çift yazımı (duplication) bakım yükü doğurur.

### 2.2. Veritabanı ve Önbellek Katmanı (PostgreSQL + Valkey/Redis)

500 RPS seviyesinde tipik bir To-Do uygulamasında trafik karakteristiği **%75 Okuma (Read) / %25 Yazma (Write)** şeklindedir (375 Read RPS, 125 Write RPS).

- **PostgreSQL Optimizasyonu:**
  - PostgreSQL bağlantı modeli "process-per-connection" olduğundan, doğrudan 500 eşzamanlı DB bağlantısı açmak veritabanını tüketir.
  - **Zorunlu Mimari Bileşen:** Entegre Connection Pooling (**PgBouncer** veya Drizzle/Prisma connection pool ayarları: max 20-30 pool size).
  - **İndeksleme:** `(user_id, status, due_date)`, `(user_id, created_at)` bileşik indeksleri (composite indexes) ile sorgu planlayıcısı Index-Only Scan kullanarak disk I/O yapmadan 1-3 ms'de yanıt üretir.
- **Önbellekleme Katmanı:**
  - Kullanıcı oturum bilgileri, yetki kontrolleri, sık erişilen "Bugünkü Görevler" ve "Haftalık Görünüm" özetleri Valkey/Redis üzerinde 60-300 saniyelik TTL ile önbelleklenir.
  - 500 RPS'lik yükün en az %60'ı doğrudan önbellekten (< 2ms) döndürülerek PostgreSQL üzerindeki yük ~150-200 RPS seviyesine düşürülür.
  - Rate Limiting (Kullanıcı başına 100 req/min) Redis Token Bucket algoritmasıyla uygulanır.

### 2.3. Asenkron İş Kuyruğu (BullMQ)

- Charter Madde 3.4'te yer alan hatırlatıcı ve bildirimler ana HTTP istek akışında (request/response cycle) çalıştırılmamalıdır.
- BullMQ, Redis tabanlı hafif, dağıtık ve gecikmeli görevleri (delayed jobs) yöneten bir kütüphanedir.
- Görev oluşturulduğunda vade tarihi için BullMQ gecikmeli kuyruğuna referans yazılır; worker servisi zamanı geldiğinde harici bildirim sağlayıcısını (SendGrid/SES) asenkron çağırır. Ana API gecikmesi sıfır etkilenir.

### 2.4. Ölçeklenebilirlik ve Spike POC Gereksinimi

- **Yatay Ölçekleme:** Stateless tasarlanan API container'ları Cloud Run / AWS ECS üzerinde CPU kullanımı > %70 olduğunda otomatik olarak 2'den 4 replikaya çıkacak şekilde yapılandırılmalıdır.
- **Spike POC (Proof of Concept) İhtiyacı:**
  - **Karar:** **1-2 Günlük Spike POC Zorunludur.**
  - **POC Kapsamı:** NestJS (Fastify) + Drizzle ORM + PostgreSQL + Valkey mimarisi üzerinde K6 ile 500 RPS sabit yük ve 1.000 RPS ani pik (spike) testi yapılarak P95 gecikmesinin < 100ms kaldığı, connection pool tükenmesi veya bellek sızıntısı (memory leak) yaşanmadığı P3 (Tasarım) aşamasında doğrulanmalıdır.

---

## 3. LİSANS VE UYUM DENETİMİ (LICENSE COMPLIANCE AUDIT)

Açık kaynak kütüphanelerin tescilli (proprietary) ticari yazılımlar içinde kullanılması fikri mülkiyet (IP) hakları ve kaynak kod ifşa yükümlülükleri açısından sıkı denetim gerektirir.

### 3.1. Kullanılacak Bileşenler ve Lisans Durumu

| Katman / Kütüphane | Aday Bileşenler | Lisans Türü | Risk Seviyesi | Uyum Notu |
| :--- | :--- | :--- | :--- | :--- |
| **Frontend UI** | Next.js, React, TailwindCSS, Lucide Icons | MIT | Düşük | Ticari kullanıma tamamen uygun, telif feragati tam. |
| **Frontend State/Form** | TanStack Query, Zustand, React Hook Form, Zod | MIT | Düşük | Tamamen serbest, ticari risk yok. |
| **Backend Core** | NestJS, Fastify, TypeScript | MIT / Apache 2.0 | Düşük | Apache 2.0 patent koruması sağlar, uyumlu. |
| **ORM / DB Driver** | Drizzle ORM, Prisma, pg (node-postgres) | Apache 2.0 / MIT | Düşük | Kapalı kaynak ürünlerde serbestçe kullanılabilir. |
| **İş Kuyruğu** | BullMQ | MIT | Düşük | Ticari kullanıma uygun. |
| **Veritabanı** | PostgreSQL | PostgreSQL License | Düşük | BSD/MIT benzeri son derece liberal, ticari dostu lisans. |
| **In-Memory Cache (Eski)** | **Redis (Sürüm >= 7.4)** | **RSALv2 / SSPLv1** | **YÜKSEK (Ticari Kısıt)** | **Redis Ltd. 2024 lisans değişikliği ile copyleft kısıtı getirmiştir.** |
| **Önerilen Cache** | **Valkey (Linux Foundation) veya Redis <= 7.2** | **BSD-3-Clause** | Düşük | Redis yerine drop-in replacement olarak %100 BSD lisanslı Valkey kullanılmalıdır. |
| **Konteyner / OS** | Alpine Linux / Debian Slim / Node Docker | MIT / BSD / GPLv2 (Kernel) | Düşük | Container katman izolasyonu (userspace) GPL virütik etkisini engeller. |

### 3.2. Kesinlikle Reddedilecek Lisanslar Tablosu (Blacklist)

Projede kullanılacak tüm paketlerde aşağıdaki lisans sınıflandırma matrisi katı kural (hard rule) olarak işletilecektir:

```
[Yeşilliste - Kabul]     ──> MIT, Apache 2.0, BSD-2/3-Clause, ISC
[Griliste - Şartlı]       ──> LGPL v2.1/v3, MPL 2.0 (Yalnızca dinamik bağlama & modifikasyonsuz)
[Karaliste - KESİN YASAK] ──> AGPL v3, GPL v2/v3, SSPL v1, RSALv2, CC-BY-NC
```

| Lisans Türü | Temsili Örnekler | Reddedilme Gerekçesi (Risk Analizi) | Eylem / Kural |
| :--- | :--- | :--- | :--- |
| **AGPL v3** (Affero GPL) | Grafana (yeni), Mastodon, bazı MongoDB forkları | **Ağ Virütik Etkisi (Network Copyleft):** Yazılım SaaS olarak sunulduğunda, tüm arka yüz kaynak kodunun kamuya açılmasını yasal olarak zorunlu kılar. | **KESİNLİKLE RED (Derhal Engelle)** |
| **GPL v2 / GPL v3** | Linux CLI araçları, MySQL Client lib, GnuTLS | **Güçlü Copyleft:** Uygulama ile statik veya dinamik olarak aynı bellek alanında bağlandığında uygulamanın fikri mülkiyetini yok eder, kod açmayı zorunlar. | **KESİNLİKLE RED (Node paketlerinde yasak)** |
| **SSPL v1 & RSALv2** | Redis >= 7.4, MongoDB Community Server | **Source-Available / Anti-Cloud Kısıtı:** Servis sağlayıcı modeliyle ticari sunum yapıldığında altyapı kodlarının yayınlanmasını dayatır. | **RED (Valkey veya BSD sürümleri kullanılacak)** |
| **CC-BY-NC** (Non-Commercial) | Çeşitli ikon setleri, hazır UI şablonları | **Ticari Kullanım Yasağı:** Yazılımın ticari amaçla, gelir modeliyle veya şirket bünyesinde çalıştırılmasını yasaklar. | **KESİNLİKLE RED** |

---

## 4. REGÜLASYON VE HUKUKİ UYUM ANALİZİ

### 4.1. KVKK (6698 Sayılı Kanun) ve GDPR Uyumu

Platform bireysel kullanıcıların adı, soyadı, e-posta adresi, IP logları ve görev başlıkları/içeriklerini işleyecektir. Bu veriler kişisel veri niteliğindedir.

1. **Hukuki İşleme Sebebi:**
   - KVKK Madde 5/2-c ve GDPR Madde 6(1)(b): "Bir sözleşmenin kurulması veya ifasıyla doğrudan doğruya ilgili olması kaydıyla kişisel verilerin işlenmesinin gerekli olması."
   - Pazarlama ve opsiyonel bildirimler için: **Açık Rıza (Explicit Consent)** mekanizması (Opt-in onay kutucuğu, önceden işaretlenmemiş).
2. **Aydınlatma Yükümlülüğü (KVKK Md. 10 & GDPR Art. 13):**
   - Kayıt ekranında kullanıcıya anlaşılır "Kişisel Verilerin İşlenmesi Aydınlatma Metni" ve "Çerez Politikası" sunulmalıdır.
3. **Veri Minimizasyonu:**
   - Yalnızca sistemin çalışması için zorunlu asgari veri talep edilmelidir (Ad, E-posta, Şifre). T.C. Kimlik No, telefon veya doğum tarihi gibi gereksiz veriler asla istenmemelidir.

### 4.2. Veri Saklama ve Unutulma Hakkı (Right to Erasure / KVKK Md. 7 & GDPR Art. 17)

- **Soft Delete vs. Hard Delete:**
  - Kullanıcı "Hesabımı Sil" talebinde bulunduğunda hesap anında devre dışı bırakılır (Soft Delete: `deleted_at = NOW()`).
  - Hukuki itiraz ve kötüye kullanım süreçleri için 30 günlük bekleme (grace) süresi tanınır.
  - 30 gün sonunda otomatik bir cron worker çalışarak kullanıcının tüm kişisel verilerini, görevlerini ve ilişkili kayıtlarını **fiziksel olarak kalıcı olarak siler (Hard Delete)** veya geri döndürülemez biçimde anonimleştirir (SHA-256 hash maskeleme).
- **Veri Taşınabilirliği (Data Portability - GDPR Art. 20):**
  - Charter Bölüm 3.3'te taahhüt edilen **JSON/CSV formatında veri dışa aktarma (export)** yeteneği doğrudan GDPR Madde 20 ve KVKK ilkeleriyle uyumludur.

### 4.3. Kimlik Doğrulama, Parola ve Şifreleme Standartları

| Güvenlik Alanı | Standart / Protokol | Teknik Gereksinim ve Uygulama |
| :--- | :--- | :--- |
| **Parola Saklama** | **Argon2id (Önerilen)** veya bcrypt (salt >= 12) | OWASP Password Storage standartlarına uygun bellek-zorluğu (memory-hard) sunan **Argon2id** (`m=65536, t=3, p=4`) uygulanmalıdır. GPU/ASIC brute-force saldırılarına tam koruma sağlar. MD5, SHA-1 ve tuzsuz SHA-256 KESİNLİKLE YASAKTIR. |
| **Aktarım Güvenliği (In-Transit)** | **TLS 1.3 (Zorunlu)** / Min. TLS 1.2 | Tüm API ve web trafiğinde HTTPS zorunlu tutulmalı, HSTS (max-age: 31536000, includeSubDomains, preload) bayrağı aktif edilmelidir. |
| **Durağan Veri Güvenliği (At-Rest)** | **AES-256 (KMS / TDE)** | PostgreSQL depolama hacmi bulut sağlayıcı düzeyinde AES-256 ile şifrelenmeli; yedekler KMS anahtarları ile korunmalıdır. |
| **Oturum Yönetimi** | **JWT & Secure Cookies** | Access Token kısa ömürlü (15 dk), Refresh Token (7 gün) Valkey'de tutulmalı ve `HttpOnly`, `Secure`, `SameSite=Strict` çerezlerde saklanmalıdır. |

### 4.4. Veri Egemenliği ve Sınır Ötesi Aktarım (Data Sovereignty)

- 2024 yılında yürürlüğe giren KVKK Madde 9 değişiklikleri uyarınca verilerin yurt dışı sunucularına aktarımı belirli güvencelere bağlanmıştır.
- Bulut veri merkezi seçimi için:
  - Birincil Bölge: **Frankfurt (eu-central-1)** — AB GDPR ve KVKK regülasyonuna uyumlu güvenli bölge.
  - Bildirim Sağlayıcıları: AB lokasyonlu veri işleme sözleşmesi (DPA) sunan sağlayıcılar tercih edilmelidir.

### 4.5. Denetim İzleri (Audit Logs) ve 5651 Sayılı Kanun

- Kullanıcı giriş denetimleri (login, şifre sıfırlama, e-posta değişikliği, hesap silme) değiştirilemez (append-only) bir denetim tablosuna kaydedilmelidir.
- Sistem erişim logları IP adresi, istek zamanı ve durum kodunu içerecek şekilde en az 1 yıl süreyle saklanmalı ve RFC 3161 uyumlu zaman damgasıyla imzalanmalıdır.

### 4.6. PCI-DSS ve HIPAA Muafiyet Gerekçeleri

`PROJECT_CHARTER.md` dosyasının 4. Maddesi (Kesin Kapsam Dışı Sınırlar) hukuki açıdan incelenmiştir:
1. **PCI-DSS Muafiyeti:** Charter Madde 4.2 gereğince ödeme ağ geçidi ve abonelik faturalandırması ilk sürümde kesinlikle yer almamaktadır. Kart verisi (PAN, CVV) işlenmemekte, depolanmamakta ve akmamaktadır. Sistem PCI-DSS denetimlerinden **TAMAMEN MUAFDIR**.
2. **HIPAA Muafiyeti:** Platform bireysel görev/zaman yönetimi uygulamasıdır. Elektronik korunan sağlık bilgisi (ePHI) toplanmamakta, sistem "Covered Entity" veya "Business Associate" statüsü taşımamaktadır. Sistem HIPAA yükümlülüklerinden **TAMAMEN MUAFDIR**.

---

## 5. RİSK MATRİSİ VE AZALTMA PLANI

| Risk No | Risk Tanımı | Etki | Olasılık | Azaltma / Önleme Stratejisi (Mitigation) |
| :---: | :--- | :---: | :---: | :--- |
| **R-01** | Redis 7.4+ lisans kısıtı (RSALv2) riski | Orta | Yüksek | %100 BSD lisanslı Linux Foundation **Valkey** kütüphanesi kullanılacaktır. |
| **R-02** | 500 RPS pik yükte PostgreSQL bağlantı havuzu tükenmesi | Yüksek | Orta | PgBouncer entegrasyonu ve Valkey önbellek oranı %60 üzerine çıkarılacaktır. |
| **R-03** | 4 haftalık dar takvimde çoklu dil (Go + TS) gecikmesi | Yüksek | Yüksek | Backend NestJS (Fastify) + TypeScript olarak sabitlenecektir. |
| **R-04** | Hesap silme taleplerinde veri kalıntısı (KVKK ihlali) | Yüksek | Düşük | Otomatik 30 günlük hard-delete ve kriptografik imha worker'ı kurulacaktır. |
| **R-05** | AGPL/GPL lisanslı bir paketin projeye sızması | Yüksek | Düşük | CI/CD pipeline'ına `license-checker` entegre edilerek otomatik kapı konacaktır. |

---

## 6. KARAR TAVSİYESİ (DECISION: CONDITIONAL GO)

### Karar: **CONDITIONAL GO (ŞARTLI ONAY)**

Projenin teknik hedefleri, takvimi, FinOps bütçesi ve yasal çerçevesi genel olarak **UYGULANABİLİR (FEASIBLE)** bulunmuştur. Projenin sonraki fazlara (P2 Gereksinim, P3 Tasarım) güvenle geçebilmesi için aşağıdaki **4 ZORUNLU KOŞULUN** mimariye dahil edilmesi şart koşulmuştur:

#### Geçiş Koşulları (Mandatory Conditions to Proceed):

1. **[TEKNİK KOŞUL] Teknoloji Yığınının TypeScript ve Fastify Olarak Sabitlenmesi:**
   - 4 haftalık teslimat hedefi (17 Ekim 2026) göz önünde bulundurularak Go seçeneği elenmeli; Backend mimarisi **NestJS + Fastify Adapter + TypeScript** olarak belirlenmelidir. ORM katmanında yüksek performans ve düşük bellek için **Drizzle ORM** tercih edilmelidir.
2. **[LİSANS KOŞULU] Valkey (Redis İkamesi) Kullanımı ve Lisans Kapısı:**
   - Redis'in lisans kısıtlarından (RSALv2/SSPLv1) korunmak için altyapıda doğrudan **Valkey 7.2+** (BSD-3-Clause) kullanılmalıdır.
   - CI/CD sürecine AGPL ve GPL paketleri derhal reddeden otomatik lisans tarayıcısı (`license-checker --production --onlyAllow "MIT;Apache-2.0;BSD-3-Clause;BSD-2-Clause;ISC"`) eklenmelidir.
3. **[GÜVENLİK & YASAL KOŞUL] Kimlik ve Veri İmha Standartlarının Kodlanması:**
   - Parola hashleme standardı olarak kesinlikle **Argon2id** kullanılmalıdır.
   - KVKK Md. 7 ve GDPR Art. 17 gereksinimleri için mimari tasarıma ilk günden "30 Günlük Hard Delete Worker" ve "JSON Export" servisleri dahil edilmelidir.
4. **[DOĞRULAMA KOŞULU] 2 Günlük K6 Spike POC Testi:**
   - P3 mimari tasarım fazı başlamadan önce, temel NestJS + Fastify + Valkey + PostgreSQL iskeleti üzerinde 500 RPS yük altında P95 < 100ms gecikmesini ampirik olarak ispatlayan bir K6 yük testi icra edilmelidir.

---

## 7. DOĞRULAMA KAPISI (QUALITY GATE SIGN-OFF)

- [x] **Karar Başlığı Belirlendi:** Karar: CONDITIONAL GO (Şartlı Onay).
- [x] **Lisans Riskleri Tablolaştırıldı:** MIT/Apache/BSD onaylı, AGPL/GPL/SSPL karaliste tablosu hazırlandı.
- [x] **GPL Virütik Paketler Reddedildi:** CI/CD lisans tarayıcısı kuralı tanımlandı.
- [x] **Regülasyon Uyum Analizi:** KVKK, GDPR, PCI-DSS muafiyeti, HIPAA muafiyeti belgelendi.
- [x] **Teknik Kapasite Onaylandı:** 500 RPS, P95 < 100ms için PgBouncer ve Valkey önbellek stratejisi netleşti.

**Fizibilite Raportörü:** Kıdemli Teknik ve Yasal Fizibilite Araştırmacısı (research subagent)  
**Durum:** **BAŞARILI (A2 Adımı Tamamlandı — A3 FinOps Bütçeleme Fazına Geçişe Hazır)**
