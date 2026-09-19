# SHIFT-LEFT RİSK KÜTÜĞÜ VE MATRİSİ (RISK REGISTER)

**Proje:** demo-project — Bireysel Zaman & Görev Yönetim Platformu (To-Do App)  
**Tarih:** 19 Eylül 2026  
**Sürüm:** v1.0.0  
**SDLC Fazı:** Aşama 01 (Planlama) — Adım A4 (Shift-Left Risk Değerlendirmesi & Matris)  
**Girdi Belgeleri:** [PROJECT_CHARTER.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/PROJECT_CHARTER.md), [FEASIBILITY_REPORT.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/FEASIBILITY_REPORT.md), [FINOPS_BUDGET.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/FINOPS_BUDGET.md)  
**Rol:** Kıdemli Risk Denetçisi (Risk Controller)  

---

## 1. YÖNETİCİ ÖZETİ VE RİSK YÖNETİM METODOLOJİSİ

Bu çalışma; SDLC Aşama 01 kapsamında olası teknik, kurumsal, dışsal ve yasal riskleri projenin en başında (**Shift-Left** prensibiyle) tespit etmek, derecelendirmek ve somut azaltma stratejileri geliştirmek üzere hazırlanmıştır.

Değerlendirme standardı:
- **Olasılık (O):** 1 (Çok Düşük) ile 5 (Çok Yüksek) arasında
- **Etki (E):** 1 (Önemsiz) ile 5 (Felaket / Yıkıcı) arasında
- **Risk Skoru:** $Skor = Olasılık \times Etki$ (1 - 25 Skalası)

### Risk Bölgeleri Sınıflandırması
- 🟢 **Düşük Risk (1 - 6):** Kabul edilebilir, standart izleme.
- 🟡 **Orta Risk (7 - 14):** Aktif izleme, önleyici kontroller.
- 🔴 **Kritik / Yüksek Risk (15 - 25):** Acil eylem, zorunlu Tetikleyici Koşul (Fallback / Contingency Action) planı şartı.

---

## 2. 5×5 OLASILIK VE ETKİ RİSK MATRİSİ

```
ETKİ (Impact)
  5 │               │  SR-02 (10)   │  HR-01 (15)   │  SR-01 (15)   │               │
  4 │               │  TR-02 (8)    │  ER-01 (12)   │  TR-01 (16)   │               │
    │               │  ER-02 (8)    │               │               │               │
  3 │               │  HR-02 (6)    │  TR-03 (9)    │  SR-03 (12)   │               │
  2 │               │               │               │               │               │
  1 │               │               │               │               │               │
    └───────────────┴───────────────┴───────────────┴───────────────┴───────────────┘
            1               2               3               4               5
                                  OLASILIK (Likelihood)

    [Lejant: 🔴 Kırmızı (Skor >= 15)  |  🟡 Sarı (Skor 7-14)  |  🟢 Yeşil (Skor 1-6)]
```

---

## 3. KAPSAMLI RİSK KÜTÜĞÜ (RISK REGISTER)

| Risk ID | Kategori | Risk Tanımı | Olasılık (1-5) | Etki (1-5) | Risk Skoru (O×E) | Azaltma Stratejisi (Mitigation Plan) | Tetikleyici Koşul ve Acil Eylem Planı (Fallback Trigger) | Sorumlu Ajan / Rol |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- | :--- | :--- |
| **TR-01** | **Teknik** | **500+ RPS Pik Yükte PostgreSQL Bağlantı Havuzunun Tükenmesi ve Kilitlenme (Deadlock)** | 4 | 4 | **16 (🔴 KRİTİK)** | Mimari tasarıma PgBouncer entegre edilecek; pool boyutu 20-30 ile sınırlanacak. Valkey önbellekleme oranı > %60 tutulacak. İndeksleme `(user_id, status, due_date)` olarak optimize edilecek. | **TETİKLEYİCİ:** DB CPU kullanımı > %80 veya aktif bağlantı sayısı havuzun %90'ına ulaşırsa.<br>**ACİL EYLEM:** Otomatik Read-Replica ayağa kaldırma + Valkey TTL sürelerini geçici olarak 600 sn'ye çıkarma ve write-heavy olmayan endpoint'lerde agresif rate-limiting tetikleme. | `database-architect`, `core-developer` |
| **TR-02** | **Teknik** | **Fastify / NestJS Katmanında Bellek Sızıntısı veya Event Loop Tıkanması** | 2 | 4 | **8 (🟡 ORTA)** | P3 öncesi K6 ile 1-2 günlük Spike POC yapılacak. Profilleme için Clinic.js kullanılacak. Ağır hesaplamalar (rapor dışa aktarma) worker thread'e taşınacak. | **TETİKLEYİCİ:** Event-loop lag > 50ms veya Node heap bellek kullanımı container sınırının %75'ine ulaşırsa.<br>**EYLEM:** Pod otomatik yeniden başlatma (Kubernetes Liveness Probe restart) ve bellek dump analizi. | `system-architect`, `core-developer` |
| **TR-03** | **Teknik** | **BullMQ Gecikmeli Kuyruğunda (Delayed Jobs) Valkey/Redis Bellek Aşımı** | 3 | 3 | **9 (🟡 ORTA)** | Tamamlanan ve başarısız işler için otomatik temizleme (clean/removeOnComplete) politikası uygulanacak. MaxMemory `volatile-lru` yapılandırılacak. | **TETİKLEYİCİ:** Bekleyen iş sayısı > 20.000 veya bellek kullanımı > %80 olursa.<br>**EYLEM:** Dead-Letter Queue (DLQ) tetiklenerek eski kuyruklar arşiv diskine boşaltılacak. | `backend-lead`, `core-developer` |
| **HR-01** | **İnsan & Süreç** | **4 Haftalık Hızlı Takvimde Anahtar Geliştiricinin Ayrılması veya Kritik Yetkinlik Kaybı** | 3 | 5 | **15 (🔴 KRİTİK)** | Tekil kişi bağımlılığı (Bus Factor) sıfırlanacak. Kod tabanı Antigravity CLI otonom ajanları (`core-developer`, `code-reviewer`) tarafından standart mimaride yazılacak, Living Docs (`docs/`) sürekli güncel tutulacak. | **TETİKLEYİCİ:** Sprint hızı (Burndown velocity) planlananın %35 altına düşerse veya 48 saat görev ilerlemesi durursa.<br>**ACİL EYLEM:** Antigravity CLI çift subagent moduna (`pair-programming-agent`) geçirilerek otonom kod üretimi ve test tamamlama hızı 2 katına çıkarılacak. | `project-manager`, `orchestrator-agent` |
| **HR-02** | **İnsan & Süreç** | **Ekipte Drizzle ORM ve Full-Stack TypeScript Deneyim Eksikliği** | 2 | 3 | **6 (🟢 DÜŞÜK)** | Ortak DTO ve Zod şemaları monorepo yapısında standardize edilecek. Katı ESLint/TypeScript kuralları ve CI/CD pre-commit hook'ları (Husky) zorunlu tutulacak. | **TETİKLEYİCİ:** PR gözden geçirme döngüsü > 24 saati aşarsa.<br>**EYLEM:** `code-reviewer` subagent PR'lara anlık otomatik refactoring şablonları önerecek. | `code-reviewer`, `tech-lead` |
| **ER-01** | **Dış Bağımlılık** | **Harici E-posta / Bildirim Sağlayıcısının (Amazon SES / SendGrid) Çökmesi veya Kota Aşımı** | 3 | 4 | **12 (🟡 ORTA)** | Circuit Breaker deseni uygulanacak. Çoklu sağlayıcı mimarisi kurulacak: Birincil AWS SES, İkincil Mailjet/Brevo. | **TETİKLEYİCİ:** Dış sağlayıcı hata oranı (HTTP 5xx) > %5 veya yanıt gecikmesi > 3 saniye olursa.<br>**ACİL EYLEM:** Circuit Breaker açılarak bildirim trafiği anında 2. yedek SMTP sağlayıcısına yönlendirilecek; kullanıcıya SMS/In-App fallback uyarısı verilecek. | `api-designer`, `backend-developer` |
| **ER-02** | **Dış Bağımlılık** | **Bulut Veri Merkezi Bölgesel Kesintisi (AWS/GCP Frankfurt Outage)** | 2 | 4 | **8 (🟡 ORTA)** | PostgreSQL RDS Multi-AZ (farklı kullanılabilirlik alanlarında eşzamanlı kopya) ve Cloud Run çoklu bölge dağıtımı kullanılacak. | **TETİKLEYİCİ:** Global healthcheck endpoint yanıt vermezse (> 60 sn).<br>**EYLEM:** DNS failover (Route 53 latency routing) ikincil kullanılabilirlik bölgesine otomatik yönlendirme yapacak. | `cloud-deployer`, `sre-incident-responder` |
| **SR-01** | **Güvenlik & Uyum**| **Kullanıcı Görev Verilerinin Yetkisiz Erişimi (IDOR / Broken Object-Level Authorization)** | 3 | 5 | **15 (🔴 KRİTİK)** | Tüm SQL sorgularında zorunlu parametrik `WHERE user_id = :authenticated_user` filtresi Drizzle ORM katmanında soyutlanacak. NestJS Guard ile JWT subject doğrulaması zorunlu tutulacak. | **TETİKLEYİCİ:** CI/CD güvenlik taramasında (SAST) veya sızma testinde parametre manipülasyonu tespit edilirse.<br>**ACİL EYLEM:** Pipeline derhal durdurulacak (Build Break); söz konusu endpoint API Gateway seviyesinde 403 Forbidden ile geçici olarak dondurulacak. | `security-architect`, `qa-automation-engineer` |
| **SR-02** | **Güvenlik & Uyum**| **KVKK Md. 7 / GDPR Art. 17 "Unutulma Hakkı" İhlali ve Ağır Regülasyon Cezaları** | 2 | 5 | **10 (🟡 ORTA)** | 30 günlük soft-delete grace period sonrasında çalışan otomatik hard-delete worker geliştirilecek. Kriptografik imha (Crypto-shredding: kullanıcıya özel şifreleme anahtarının silinmesi) uygulanacak. | **TETİKLEYİCİ:** Silinmiş bir kullanıcının kişisel verisi 35. günde veritabanında veya aktif yedeklerde kalmışsa.<br>**EYLEM:** Otomatik DPO (Veri Koruma Görevlisi) uyarısı tetiklenecek ve acil purging job çalıştırılacak. | `compliance-officer`, `database-architect` |
| **SR-03** | **Güvenlik & Uyum**| **Brute-Force Kimlik Doğrulama ve Bot Credential Stuffing Saldırıları** | 4 | 3 | **12 (🟡 ORTA)** | Parolalar bellek-zorluklu **Argon2id** ile hash'lenecek. IP ve kullanıcı adı bazında Valkey üzerinde katı Rate Limiting (dakikada max 5 başarısız deneme) uygulanacak. | **TETİKLEYİCİ:** Bir IP veya kullanıcı için 1 dakikada > 5 başarısız login.<br>**EYLEM:** İlgili IP 15 dakika boyunca geçici bloklanacak (HTTP 429), kullanıcıya "Olağan dışı giriş denemesi" e-posta güvenlik bildirimi gönderilecek. | `security-architect`, `sre-incident-responder` |

---

## 4. KRİTİK RİSKLER İÇİN GERİ DÖNÜŞ VE ACİL EYLEM PROTOKOLÜ (FALLBACK ACTIONS)

Quality Gate kuralı uyarınca skoru **15 ve üzeri olan tüm riskler (TR-01, HR-01, SR-01)** için özel tetikleme ve acil kurtarma planı onaylanmıştır:

```
[TR-01: Veritabanı Aşırı Yükü]  ──> DB CPU > %80 ──> Otomatik Read-Replica + Valkey Cache 600s + Write Throttling
[HR-01: Anahtar Personel Kaybı] ──> Hız < %35    ──> Antigravity CLI Otonom Subagent Çift Vardiya Modu + Living Docs
[SR-01: IDOR Veri Sızıntısı Açığı]──> SAST/DAST Fail──> Build Break + Endpoint Karantinası + Zorunlu Drizzle Scope Fix
```

---

## 5. DOĞRULAMA VE KALİTE KAPISI (QUALITY GATE SIGN-OFF)

- [x] **4 Temel Kategori Kapsandı:** Teknik (3 risk), İnsan & Süreç (2 risk), Dış Bağımlılıklar (2 risk), Güvenlik & Uyum (3 risk) olmak üzere toplam **10 somut risk** modellendi.
- [x] **5×5 Olasılık ve Etki Matrisi:** Görsel ASCII matris oluşturuldu.
- [x] **Kritik Risklerde Fallback Planı:** Skoru 15 ve üzeri olan tüm riskler (TR-01, HR-01, SR-01) için net tetikleyici eşikler ve geri dönüş (fallback action) protokolü tanımlandı.
- [x] **Sorumlu Ajanlar Belirlendi:** Her risk maddesi için sorumlu subagent rolü atandı.

**Risk Yönetim Direktörü:** Risk Controller  
**Statü:** **ONAYLANDI (Quality Gate Geçildi — A5 WBS ve /plan Fazına Geçişe Hazır)**
