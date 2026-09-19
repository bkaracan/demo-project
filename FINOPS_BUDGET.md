# BULUT MALİYET VE FİNOPS BÜTÇELEME RAPORU (FINOPS BUDGET)

**Proje:** demo-project — Bireysel Zaman & Görev Yönetim Platformu (To-Do App)  
**Tarih:** 19 Eylül 2026  
**Sürüm:** v1.0.0  
**SDLC Fazı:** Aşama 01 (Planlama) — Adım A3 (FinOps Bulut Maliyet ve Kaynak Bütçelemesi)  
**Temel Girdiler:** [PROJECT_CHARTER.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/PROJECT_CHARTER.md), [FEASIBILITY_REPORT.md](file:///C:/Users/burak/OneDrive/Masaüstü/demo-project/FEASIBILITY_REPORT.md)  
**FinOps Bütçe Tavanı:** 500,00 USD / Ay (6.000,00 USD / Yıl)  
**Hedef İşlem Kapasitesi:** 500 RPS (Pik Yük: 1.000 RPS), P95 < 100ms  

---

## 1. YÖNETİCİ ÖZETİ VE FİNOPS ÇERÇEVESİ

`demo-project` için hazırlanan bu bütçe simülasyonu, Charter'da belirlenen **aylık maksimum $500 tavan bütçe** sınırlarına sıkı sıkıya bağlı kalınarak hazırlanmıştır. Sistem mimarisi; yalın (lean), sunucusuz (serverless-first) veya hafif konteyner (container-based) bileşenlerden kurgulanmış olup, trafik dalgalanmalarına göre otomatik ölçeklenen maliyet-verimli bir altyapıyı hedefler.

Tüm hesaplamalarda **%20 Contingency (Beklenmeyen Gider Marjı)** eklenmiş olup, 12 aylık toplam işletme maliyeti tavan bütçenin oldukça altında (%25 güvenlik marjıyla) optimize edilmiştir.

---

## 2. DETAYLI BULUT ALTYAPI MALİYET HESAPLAMASI (AWS & GCP KARŞILAŞTIRMALI)

Hesaplamalar; gündüz saatlerinde ortalama 100-250 RPS, pik saatlerde 500 RPS ve gece saatlerinde 10-30 RPS yük profili (aylık ~150 milyon HTTP isteği) baz alınarak yapılmıştır.

### 2.1. Bileşen Bazlı Aylık ve Yıllık Maliyet Dağılımı

| Altyapı Bileşeni | AWS Karşılığı & Konfigürasyon | GCP Karşılığı & Konfigürasyon | Tahmini Aylık (USD) | Tahmini Yıllık (USD) |
| :--- | :--- | :--- | :---: | :---: |
| **1. Compute (API & Worker)** | **AWS ECS Fargate:** 2 vCPU, 4 GB RAM (Min: 2 replica, Max: 6 replica) | **Cloud Run:** 2 vCPU, 4 GB RAM (Min: 1 instance, Max: 5 instance, Concurrency: 80) | $85,00 | $1.020,00 |
| **2. Yönetilen Veritabanı** | **RDS PostgreSQL:** `db.t4g.small` (2 vCPU, 2GB RAM), 50 GB gp3 SSD, Otomatik Yedek | **Cloud SQL Postgres:** `db-custom-2-3840` (2 vCPU, 3.75GB RAM), 50 GB SSD | $68,00 | $816,00 |
| **3. In-Memory Cache** | **ElastiCache (Valkey 7.2):** `cache.t4g.micro` (0.5 GB RAM) veya Upstash Serverless | **Memorystore (Redis/Valkey):** Basic 1 GB veya Serverless Valkey | $22,00 | $264,00 |
| **4. Depolama & Statik Varlık** | **Amazon S3 + CloudFront:** 20 GB Depolama, 100 GB Veri Transferi (Egress) | **Cloud Storage + Cloud CDN:** 20 GB Depolama, 100 GB Egress | $12,00 | $144,00 |
| **5. Ağ, Egress & NAT Gateway** | AWS NAT Gateway (t4g EC2 router ile optimize) + DNS (Route 53) | GCP Serverless VPC Egress + Cloud DNS | $25,00 | $300,00 |
| **6. Gözlemlenebilirlik (Telemetri)** | CloudWatch Logs (7 gün saklama) + Prometheus/Grafana Cloud Free Tier | Cloud Logging + Cloud Monitoring (Free tier dahilinde) | $18,00 | $216,00 |
| **7. E-posta / Bildirim Gönderimi** | **Amazon SES:** Ayda 100.000 e-posta hatırlatma bildirimi | **SendGrid / Mailjet:** Essential Tier (100k e-posta) | $10,00 | $120,00 |
| **ARA TOPLAM (Bulut Altyapı)** | — | — | **$240,00** | **$2.880,00** |

---

## 3. ANTIGRAVITY AGENT VE AI API TÜKETİM MALİYETLERİ

SDLC fazları (P1-P7) boyunca görev yapacak 12 uzman subagent'ın model çalıştırma maliyetleri ve MVP üretim sürecindeki token tüketimi aşağıdaki şekilde bütçelenmiştir:

| Faz / Kullanım Alanı | Model Dağılımı | Tahmini Token Hacmi | Aylık Maliyet (USD) | Yıllık Maliyet (USD) |
| :--- | :--- | :--- | :---: | :---: |
| **SDLC Otonom Geliştirme** | Gemini 1.5 Pro / Flash & Claude 3.5 Sonnet (Kodlama, Test, Denetim) | 25 Milyon Token / Ay (Dev & Refactor) | $45,00 | $540,00 |
| **CI/CD Otomatik Kod İnceleme** | Gemini 1.5 Flash (PR bazlı lint, güvenlik ve mimari denetimi) | 10 Milyon Token / Ay | $8,00 | $96,00 |
| **Operasyonel Telemetri Analizi** | Gemini 1.5 Flash (Haftalık hata logu ve anomali özetleme) | 5 Milyon Token / Ay | $5,00 | $60,00 |
| **ARA TOPLAM (AI & Token Gideri)** | — | — | **$58,00** | **$696,00** |

---

## 4. TOPLAM FİNOPS BÜTÇE KONSOLİDASYONU VE CONTINGENCY

Tüm tahminlere, öngörülemeyen trafik sıçramaları, log birikmeleri ve ek sorgu yüklerini absorbe etmek üzere **%20 Contingency (Beklenmeyen Gider Payı)** eklenmiştir.

| Kalem | Aylık Projeksiyon (USD) | Yıllık Projeksiyon (USD) | Charter Tavanı Oranı |
| :--- | :---: | :---: | :---: |
| **Bulut Altyapı Giderleri (Bölüm 2)** | $240,00 | $2.880,00 | %48,0 |
| **AI & Subagent Geliştirme Maliyeti (Bölüm 3)**| $58,00 | $696,00 | %11,6 |
| **Net Tahmini İşletme Gideri** | **$298,00** | **$3.576,00** | %59,6 |
| **%20 Contingency (Beklenmeyen Gider Tamponu)**| **$59,60** | **$715,20** | %11,9 |
| **GENEL TOPLAM (FİNOPS BÜTÇESİ)** | **$357,60** | **$4.291,20** | **%71,5** |
| **CHARTER TAVAN BÜTÇESİ** | **$500,00** | **$6.000,00** | **%100,0** |
| **FİNOPS GÜVENLİK REZERVİ (Kalan Boşluk)** | **+$142,40 / ay** | **+$1.708,80 / yıl** | **+%28,5 Tasarruf** |

> [!TIP]
> Hesaplanan toplam operasyonel maliyet **$357,60 / ay** olup; proje için taahhüt edilen **$500,00 / ay** tavan bütçenin **%28,5 altındadır**. Bu güvenlik marjı, beklenmedik veri büyümesi veya kampanya dönemleri için mükemmel bir koruma sağlar.

---

## 5. TASARRUF ÖNERİLERİ VE MALİYET OPTİMİZASYON STRATEJİSİ

Maliyetlerin kontrolden çıkmasını önlemek için aşağıdaki 5 FinOps taktiği altyapı koduna (Terraform/IaC) işlenecektir:

### 5.1. Otomatik Ölçekleme ve Tavan Sınırları (Autoscaling Hard Limits)
- **Min / Max Instance Sınırı:**
  - API Container'ları için `Min: 1 (gece), Normal: 2, Max: 6 (500-1.000 RPS pik)` olarak sınırlandırılacaktır. 6 instance üzerine çıkılması engellenerek faturanın şişmesi engellenir.
  - Scale-up kuralı: CPU > %70 veya RPS > 150/pod (bekleme: 60 sn).
  - Scale-down kuralı: CPU < %30 (bekleme: 300 sn).
- **Geliştirme / Test Ortamı Kapatma:** Staging ortamı konteyner ve veritabanları çalışma saatleri dışında (20:00 - 08:00 ve hafta sonları) otomatik olarak uyku moduna (scale to 0) alınarak geliştirme maliyetinde **%65 tasarruf** sağlanacaktır.

### 5.2. Taahhütlü İndirimler (Savings Plans & CUD)
- MVP testlerinin ardından mimari sabitlendiğinde (1. ay sonu), 1 yıllık **AWS Compute Savings Plans** veya **GCP Committed Use Discounts (CUD)** devreye alınarak compute maliyetlerinde **%32 indirim** elde edilecektir.

### 5.3. Spot Instance / Preemptible Stratejisi
- Asenkron çalışan BullMQ worker pod'ları ve periyodik cron job'lar (30 günlük veri imha worker'ı) durumsuz (stateless) olduğundan, doğrudan **AWS Fargate Spot** veya **GCP Spot VMs** üzerinde çalıştırılacak; bu sayede worker compute gideri **%70 indirimle** karşılanacaktır.

### 5.4. Veritabanı ve Önbellek Verimliliği
- PostgreSQL bağlantı havuzu PgBouncer ile sınırlanarak DB sunucusunun gereksiz yere büyük bir instance tipine (`db.t4g.xlarge` vb.) yükseltilmesi önlenecektir.
- Valkey önbellekleme oranı > %60 seviyesinde tutularak PostgreSQL I/O maliyeti minimize edilecektir.

### 5.5. Bütçe Alarmları ve FinOps Anomali Koruması
- AWS Budgets / GCP Billing Alerts entegrasyonu:
  - **Eşik 1 (%60 - $300):** FinOps ekibine bilgilendirme e-postası.
  - **Eşik 2 (%85 - $425):** Uyarı alarmı (Slack/Webhook bildirimi).
  - **Eşik 3 (%100 - $500):** Otomatik aksiyon (Non-critical staging sunucularını dondurma, rate limiting seviyesini sıkılaştırma).

---

## 6. DOĞRULAMA VE KALİTE KAPISI (QUALITY GATE SIGN-OFF)

- [x] **Aylık ve Yıllık Projeksiyon:** AWS & GCP bazında 12 aylık gider simülasyonu detaylandırıldı ($357,60/ay — $4.291,20/yıl).
- [x] **Antigravity AI Token Tüketimi:** SDLC ve operasyonel model giderleri bütçelendi ($58,00/ay).
- [x] **Contingency Payı:** %20 beklenmeyen gider tamponu ($59,60/ay) eklendi.
- [x] **Autoscaling Tavan Sınırları:** Min 1, Normal 2, Max 6 pod sınırı ve gece uyku politikası belirlendi.
- [x] **Charter Uyumu:** Toplam bütçe $500/ay tavanının altında kalarak %28,5 rezerv sağlandı.

**FinOps Baş Mimarı:** FinOps Architect  
**Statü:** **ONAYLANDI (Quality Gate Geçildi — A4 Risk Kütüğü Adımına Geçişe Hazır)**
