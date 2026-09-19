# GEREKSİNİM İZLENEBİLİRLİK MATRİSİ (REQUIREMENTS TRACEABILITY MATRIX - RTM)

**Proje:** demo-project — Bireysel Zaman & Görev Yönetim Platformu (To-Do App)  
**Tarih:** 19 Eylül 2026  
**Sürüm:** v1.0.0  
**SDLC Fazı:** Aşama 02 (Gereksinim Analizi) — Adım A5 (Abuse Cases & RTM İzlenebilirlik)  
**Girdi Belgeleri:** [SRS.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/SRS.md), [USER_STORIES.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/USER_STORIES.md), [ABUSE_CASES.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/ABUSE_CASES.md)  
**Rol:** QA Mühendisi & Sistem Analisti  
**Statü:** **ONAYLANDI (Quality Gate Geçildi — Aşama 03 Tasarım'a Geçiş Onaylandı)**  

---

## 1. İZLENEBİLİRLİK VE KALİTE GÜVENCESİ METODOLOJİSİ

Bu matris; projenin tüm fonksiyonel (`FR-*`), fonksiyonel olmayan (`NFR-*`) ve kapsam dışı (`FR-OOS-*`) gereksinimlerinin sistem mimarisi (`C4`, `Contracts`), kodlama bileşenleri ve otomatik test seviyeleri (Birim, Entegrasyon, E2E, Performans, Güvenlik) arasındaki çift yönlü izlenebilirlik (bi-directional traceability) köprüsünü kurar.

### Kural ve Kalite İlkesi
- **Sıfır Yetim (Orphan) İlkesi:** Hiçbir gereksinim testsiz veya tasarımsız bırakılamaz.
- **Test Kapsamı Standartları:** Her işlevsel kural en az bir otomatik test türüyle doğrulanmak zorundadır.

---

## 2. TAM İZLENEBİLİRLİK MATRİSİ (RTM TABLOSU)

| Gereksinim ID | Gereksinim Tanımı ve Kapsam Özeti | İlgili BDD / Abuse Case | Tasarım & Mimari Bileşeni | Atanan Test Türü (Unit/Int/E2E/Sec/Perf) | İzlenebilirlik Durumu |
| :---: | :--- | :---: | :--- | :--- | :---: |
| **FR-AUTH-001** | Argon2id ile güvenli kullanıcı kaydı ve parola kuralları | `US-001` | `AuthService`, `Argon2Hasher`, `UserEntity` | **Unit Test** (Password complexity), **Integration Test** (DB duplicate check) | **MAPPED** |
| **FR-AUTH-002** | JWT Access (15dk) / Refresh (7g) token üretimi & Secure Cookie | `US-002` | `JwtService`, `TokenController`, `AuthGuard` | **Integration Test** (Token generation), **E2E Test** (Login flow) | **MAPPED** |
| **FR-AUTH-003** | Brute-force koruması: 1 dk'da 5 başarısız girişte 15 dk IP engeli | `US-003`, `AC-001` | `RateLimitMiddleware`, `ValkeyTokenBucket` | **Integration Test** (Rate limit counts), **Security Test** (ZAP login attack) | **MAPPED** |
| **FR-AUTH-004** | Güvenli çıkış ve Refresh Token'ın Valkey kara listesine alınması | `US-004`, `AC-003` | `TokenBlacklistService`, `ValkeyCache` | **Integration Test** (Blacklist verification), **Security Test** (Replay attack) | **MAPPED** |
| **FR-AUTH-005** | Tek kullanımlık 30 dk süreli token ile şifre sıfırlama | `US-001` (Alt) | `PasswordResetService`, `MailerQueue` | **Unit Test** (Token expiry check), **Integration Test** (Reset flow) | **MAPPED** |
| **FR-TASK-001** | Başlık doğrulamalı görev oluşturma (TODO, Medium, Max 1.000) | `US-005`, `AC-004` | `TaskService`, `DrizzleSchema.tasks`, `ZodSchema` | **Unit Test** (Validation schema), **Integration Test** (Quota limit check) | **MAPPED** |
| **FR-TASK-002** | Görev güncelleme ve IDOR yetkisiz erişim koruması (`WHERE user_id`) | `US-006`, `AC-002` | `TaskRepository`, `TenantIsolationGuard` | **Security Test** (BOLA/IDOR attempt), **Integration Test** (Ownership check) | **MAPPED** |
| **FR-TASK-003** | Görev durumu güncelleme, `completed_at` damgası ve arşiv kontrolü | `US-007` | `TaskStateMachine`, `TaskService` | **Unit Test** (State transition rules), **Integration Test** (Status persistence) | **MAPPED** |
| **FR-TASK-004** | Öncelik seviyesi atama (`LOW`, `MEDIUM`, `HIGH`, `URGENT`) | `US-008` | `PriorityEnum`, `TaskValidator` | **Unit Test** (Enum validation), **Integration Test** (DB query) | **MAPPED** |
| **FR-TASK-005** | Göreve etiket iliştirme (1-30 karakter, görev başına max 10 adet) | `US-008` | `TagService`, `DrizzleSchema.task_tags` | **Unit Test** (Tag constraints), **Integration Test** (Tag relation limit) | **MAPPED** |
| **FR-TASK-006** | Tamamlanmış görevlerin arşivlenmesi ve aktif listeden gizlenmesi | `US-007` (Alt) | `TaskService.archive()`, `TaskRepository` | **Unit Test** (Archive guard condition), **Integration Test** (Query filtering) | **MAPPED** |
| **FR-TASK-007** | Görev silme (`is_deleted=true`) ve BullMQ bekleyen iş temizliği | `US-009` | `TaskService.delete()`, `BullQueueManager` | **Integration Test** (Soft delete check), **Integration Test** (Queue cleanup) | **MAPPED** |
| **FR-SCHED-001**| Göreve gelecek zamanlı vade tarihi (`due_date >= NOW()`) atama | `US-010`, `AC-006` | `ScheduleService`, `DateValidator` | **Unit Test** (Past date reject rule), **Integration Test** (Calendar mapping) | **MAPPED** |
| **FR-SCHED-002**| Günlük, haftalık, aylık takvim görünümü sorguları (P95 < 100ms) | `US-010` | `CalendarController`, `ValkeyScheduleCache` | **Performance Test** (K6 500 RPS query), **E2E Test** (Calendar UI view) | **MAPPED** |
| **FR-SCHED-003**| Görev yeniden planlandığında eski hatırlatıcı işinin iptal edilmesi | `US-010` (Alt) | `ScheduleService`, `BullQueueManager` | **Integration Test** (Reschedule queue replace verification) | **MAPPED** |
| **FR-NOTIF-001**| Vade öncesi hatırlatıcı kurma ve BullMQ gecikmeli kuyruk kaydı | `US-011` | `NotificationService`, `BullDelayedQueue` | **Unit Test** (Offset calculation), **Integration Test** (Delayed job payload) | **MAPPED** |
| **FR-NOTIF-002**| Hatırlatma anında görevin kontrolü ve e-posta/push fırlatılması | `US-011`, `AC-006` | `NotificationWorker`, `SESGateway`, `PushGateway` | **Integration Test** (Completed task discard), **Integration Test** (SES client) | **MAPPED** |
| **FR-NOTIF-003**| Bildirim hatasında 3 kez üstel geri çekilme (1dk, 5dk, 15dk) ve DLQ | `US-011` | `NotificationWorker.retryStrategy()`, `DeadLetterQueue` | **Integration Test** (Backoff delays), **Integration Test** (DLQ routing) | **MAPPED** |
| **FR-NOTIF-004**| Günlük özet ve sessiz saatler (Quiet Hours) tercihlerinin işletilmesi | — | `UserProfileService`, `NotificationFilter` | **Unit Test** (Quiet hours filter check), **Integration Test** (Preference update) | **MAPPED** |
| **FR-PROF-001** | GDPR Art. 20 verileri JSON/CSV dışa aktarma (24 saatte max 3 kez) | `US-012`, `AC-004` | `ExportService`, `ArchiveStreamer` | **Integration Test** (Export schema), **Integration Test** (Rate limit quota) | **MAPPED** |
| **FR-PROF-002** | KVKK Md. 7 hesap silme talebi, oturum iptali ve 30 gün cayma | `US-013`, `AC-005` | `AccountLifecycleService`, `AuthGuard` | **Integration Test** (Soft delete & session purge), **Security Test** (Login block) | **MAPPED** |
| **FR-PROF-003** | 30 gün sonunda otonom hard-delete ve kriptografik anahtar imhası | `US-013`, `AC-005` | `PurgeWorker`, `CryptoShredderService` | **Integration Test** (Hard delete verification), **Unit Test** (KMS key shred) | **MAPPED** |
| **FR-OOS-001** | Takım ve çok kullanıcılı ortak pano (Kapsam Dışı) | — | — (Mimariye dahil edilmedi) | **Scope Verification** (PR architecture review) | **EXCLUDED** |
| **FR-OOS-002** | Ödeme ağ geçidi ve abonelik modülü (Kapsam Dışı) | — | — (Mimariye dahil edilmedi) | **Scope Verification** (PCI-DSS compliance check) | **EXCLUDED** |
| **FR-OOS-003** | Yapay zeka ile otopilot görev çizelgeleme (Kapsam Dışı) | — | — (Mimariye dahil edilmedi) | **Scope Verification** (Feature gate check) | **EXCLUDED** |
| **FR-OOS-004** | 2 yönlü harici takvim canlı senkronizasyonu (Kapsam Dışı) | — | — (Mimariye dahil edilmedi) | **Scope Verification** (Scope audit) | **EXCLUDED** |
| **FR-OOS-005** | Yerel masaüstü binary uygulamaları (Kapsam Dışı) | — | — (Yalnızca responsive Web/PWA) | **Scope Verification** (Build artifacts check) | **EXCLUDED** |

---

## 3. FONKSİYONEL OLMAYAN GEREKSİNİMLER (NFR) İZLENEBİLİRLİĞİ

| NFR ID | NFR Tanımı | Sayısal Hedef Eşik (SLO) | Doğrulama & Test Aracı | Test Aşaması | İzlenebilirlik Durumu |
| :---: | :--- | :--- | :--- | :---: | :---: |
| **NFR-PERF-001** | API P95 Yanıt Süresi | **P95 < 150 ms** (1.000 VU altında) | **K6 Yük Testi** (`k6 run --vus 1000`) | CI / Staging | **MAPPED** |
| **NFR-PERF-002** | API P99 Yanıt Süresi | **P99 < 400 ms** (1.000 VU altında) | **K6 Stres Testi** + OpenTelemetry APM | CI / Staging | **MAPPED** |
| **NFR-PERF-003** | Eşzamanlı İşlem Kapasitesi | **>= 500 RPS** (Pik: 1.000 RPS) | **K6 Sabit Throughput Testi** | CI / Staging | **MAPPED** |
| **NFR-PERF-004** | Web Vitals Önyüz Hızı | **FCP < 1.2s, LCP < 2.5s** | **Google Lighthouse CI** | CI / Pre-deploy | **MAPPED** |
| **NFR-AVAIL-001**| Sistem Erişilebilirliği | **%99.9 Uptime** (Max 43.8 dk/ay) | **Uptime Robot / Synthetic Monitor** | Production | **MAPPED** |
| **NFR-AVAIL-002**| Hata Bütçesi | **Max %0.1 Hata Oranı** (5xx) | **Grafana SLO & Alertmanager** | Production | **MAPPED** |
| **NFR-SEC-001**  | İletimde Şifreleme | **TLS 1.3 Zorunlu, HSTS** | **SSL Labs API & curl test** | Pre-deploy | **MAPPED** |
| **NFR-SEC-002**  | Durağan Veri Şifreleme | **AES-256 (KMS Entegre)** | **Checkov & Tfsec IaC Scan** | CI Pipeline | **MAPPED** |
| **NFR-SEC-003**  | Parola Karma Güvenliği | **Argon2id** (`m=65536, t=3, p=4`) | **Birim Testler (Password Hash Check)** | CI Pipeline | **MAPPED** |
| **NFR-SEC-004**  | Uygulama Güvenlik Açıkları | **OWASP ASVS Seviye 2 (Zero Critical)** | **OWASP ZAP DAST & SonarQube SAST** | CI Pipeline | **MAPPED** |
| **NFR-DR-001**   | Veri Kaybı Toleransı | **RPO < 5 Dakika** | **PostgreSQL RDS PITR Simülasyonu** | Pre-production | **MAPPED** |
| **NFR-DR-002**   | Servis Ayağa Kalkma Süresi | **RTO < 30 Dakika** | **Multi-AZ Otomatik Failover Testi** | Pre-production | **MAPPED** |
| **NFR-ACC-001**  | Dijital Erişilebilirlik | **WCAG 2.1 AA (Kontrast ≥ 4.5:1)** | **axe-core & Pa11y CI** | CI Pipeline | **MAPPED** |
| **NFR-ACC-002**  | Klavye ile Gezinilebilirlik| **%100 Klavye Uyumu (Tab/Enter)** | **Playwright Keyboard E2E Tests** | CI Pipeline | **MAPPED** |

---

## 4. DOĞRULAMA VE KALİTE KAPISI (QUALITY GATE SIGN-OFF)

- [x] **Sıfır Yetim Gereksinim:** Tüm fonksiyonel (`FR-*`) ve fonksiyonel olmayan (`NFR-*`) gereksinimlerin karşısında atanmış en az bir test türü ve tasarım bileşeni bulunmaktadır.
- [x] **3 Boyutlu Köprü:** Gereksinim (SRS) ↔ BDD Kabul Kriteri (User Story / Abuse Case) ↔ Test Doğrulaması eksiksiz kuruldu.
- [x] **Kapsam Dışı Sınırların Denetimi:** 5 kapsam dışı madde (`FR-OOS-*`) matriste "EXCLUDED" olarak açıkça işaretlendi.
- [x] **Faz 03 Tasarım Kapısı:** Tüm gereksinim şartnameleri tamamlandığı için **SDLC Aşama 03 Sistem Mimarisi ve Tasarım (P3)** fazına geçiş resmen onaylandı.

**QA & Güvenlik Baş Mühendisi:** Security Analyst  
**Statü:** **ONAYLANDI (SDLC Aşama 02 Tamamlandı — Aşama 03 Tasarım ve C4 Modellemesine Geçişe Hazır)**
