# PROJE BAŞLATMA BELGESİ (PROJECT CHARTER)
**Proje Adı:** demo-project — Bireysel Zaman & Görev Yönetim Platformu (To-Do App)  
**Tarih:** 19 Eylül 2026  
**Sürüm:** v1.0.0-draft  
**Faz:** SDLC Aşama 01 (Planlama) — Adım A1 (Kapsam & Mülakat)  
**Statü:** Onaylandı / Kilitlendi (Quality Gate Geçildi)  

---

## 1. Proje Özeti ve Varlık Sebebi (Problem Tanımı)
Modern bilgi çalışanları, serbest çalışanlar ve öğrenciler; çoklu kanallardan gelen görevleri takip etmekte, günlük/haftalık/aylık planlamalarını organize etmekte ve önceliklendirmede ciddi verimlilik kayıpları yaşamaktadır. Dağınık notlar, karmaşık veya gereksiz özelliklerle şişirilmiş kurumsal araçlar bireysel odaklanmayı engellemektedir.

**Çözüm:** Kullanıcının zaman yönetimini optimize eden, günlük, haftalık ve aylık görünümde esnek programlar oluşturmasını sağlayan, düşük gecikmeli, güvenilir ve bireysel odaklı modern bir To-Do platformu.

---

## 2. SMART Başarı Hedefleri ve KPI'lar
Projenin başarısı aşağıdaki somut, ölçülebilir (SMART) kriterler ve operasyonel metrikler ile takip edilecektir:

| Kategori | SMART Metrik | Hedef Değer | Ölçüm Yöntemi |
| :--- | :--- | :--- | :--- |
| **Performans (Latency)** | P95 API Yanıt Süresi | < 100 ms | APM & Prometheus/Grafana telemetrisi |
| **Erişilebilirlik (SLA)** | Sistem Uptime | %99.9 | Harici Healthcheck ve Uptime monitörü |
| **İşlem Yükü (Throughput)** | Eşzamanlı İstek Kapasitesi | >= 500 RPS | K6 / Locust yük testleri |
| **Kullanıcı Sadakati (Retention)** | 30 Günlük Tutundurma (D30) | > %35 | Ürün analitik olayları (Product Analytics) |
| **Üretkenlik Etkisi** | Görev Tamamlama Oranı | > %70 | Tamamlanan görev / oluşturulan görev oranı |

---

## 3. Sistem Kapsamı (In-Scope)
MVP sürümü için sisteme dahil edilecek temel fonksiyonel yetenekler:
1. **Zaman & Program Yönetimi:** Günlük, haftalık ve aylık görünümlü interaktif takvim ve görev panosu.
2. **Görev & Yaşam Döngüsü:** Görev oluşturma, durum güncelleme (Yapılacak, Devam Eden, Tamamlandı, Arşiv), öncelik seviyeleri (Acil, Yüksek, Orta, Düşük) ve etiketleme (Tagging).
3. **Bireysel Kimlik ve Profil:** Güvenli oturum açma, parola yönetimi, profil ayarları ve veri dışa aktarma (JSON/CSV).
4. **Hatırlatıcı ve Bildirim Altyapısı:** Belirlenen tarihlerde zamanında tetiklenen e-posta veya tarayıcı içi push bildirim uyarıları.
5. **Duyarlı Web / PWA Deneyimi:** Masaüstü, tablet ve mobilde yüksek performanslı, akıcı responsive kullanıcı arayüzü.

---

## 4. Kapsam Dışı Sınırlar (Out of Scope - Kesin Sınırlar)
İlk sürümde (MVP) kapsam kaymasını (scope creep) ve gereksiz maliyet/zaman artışını önlemek adına aşağıdaki 5 madde **KESİNLİKLE** bu sürümün kapsamı dışındadır:

1. **Takım / Şirket İçi Çok Kullanıcılı Ortak Pano İşbirliği:** Proje tamamen bireysel kullanıcı üretkenliğine odaklıdır; organizasyonel çalışma alanları, takım panoları veya kullanıcılar arası görev atama yapılmayacaktır.
2. **Ödeme Ağ Geçidi ve Abonelik Faturalandırması:** Stripe, Iyzico veya benzeri ödeme entegrasyonları, faturalandırma ve ücretli üyelik seviyeleri ilk sürümde yer almayacaktır.
3. **Yapay Zeka ile Otopilot Görev Çizelgeleme:** Otomatik takvim yerleşimi veya otonom yapay zeka zaman planlama motoru bu sürümde geliştirilmeyecektir.
4. **Harici Takvimlerle Çift Yönlü Canlı Senkronizasyon:** Google Calendar, Microsoft Outlook veya Apple Calendar ile çift yönlü 2-way senkronizasyon kapsam dışıdır (yalnızca iCal dışa aktarma ileride değerlendirilebilir).
5. **Yerel Masaüstü Binary Uygulamaları:** macOS, Windows veya Linux için özel derlenmiş Electron/Native binary uygulamaları yapılmayacaktır; modern responsive Web ve PWA standartları kullanılacaktır.

---

## 5. Hedef Kitle, Roller ve Dış Sistemler
- **Standart Bireysel Kullanıcı:** Kendi görev ve zaman çizelgelerini oluşturan, etiketleyen ve takip eden ana son kullanıcı aktörü.
- **Sistem Yöneticisi (Admin):** Sistem sağlığını, hata oranlarını, kullanıcı metriklerini ve operasyonel teleometriyi izleyen teknik aktör.
- **Bildirim Servisleri (External System):** Görev vadesi geldiğinde asenkron bildirim kuyruğundan tetiklenen e-posta (SMTP/SES/SendGrid) veya Web Push iletim servisleri.

---

## 6. Teslimat Hedefi ve Zaman Çizelgesi
- **Metodoloji:** 4 Haftalık Hızlı & Çevik MVP Geliştirme Döngüsü
- **Hedef MVP Canlıya Çıkış Tarihi:** 17 Ekim 2026
- **Aşamalar:**
  - Hafta 1 (19-25 Eylül 2026): Planlama & Gereksinim Analizi (P1, P2 tamamlanması)
  - Hafta 2 (26 Eylül - 02 Ekim 2026): Mimari Tasarım, Güvenlik & DB Modelleme (P3 tamamlanması)
  - Hafta 3 (03-09 Ekim 2026): Çekirdek Geliştirme, TDD & CI/CD Pipeline (P4 tamamlanması)
  - Hafta 4 (10-17 Ekim 2026): Uçtan Uca Test, Yük Testleri, Güvenlik Taramaları ve Dağıtım (P5, P6, P7)

---

## 7. FinOps Bulut Bütçe Tavanı
- **Aylık Maksimum Bulut Tavanı:** 500 USD / Ay
- **Kaynak Dağılım Tahmini:**
  - Konteyner Çalıştırma (Cloud Run / AWS ECS): ~$150/ay
  - Yönetilen Veritabanı (PostgreSQL Multi-AZ / RDS): ~$180/ay
  - Önbellekleme Katmanı (Redis ElastiCache/Upstash): ~$40/ay
  - Telemetri, Ağ & E-posta/Bildirim: ~$50/ay
  - Beklenmeyen Gider Tamponu (%16 FinOps Marjı): ~$80/ay

---

## 8. Önerilen Teknoloji Yığını (Teknik Fizibilite Girdisi)
- **Frontend & İstemci:** TypeScript, Next.js / React, TailwindCSS, PWA standartları.
- **Backend / API Katmanı:** Node.js (NestJS / Fastify) veya Go (Gin/Fiber) — Düşük gecikme ve 500 RPS throughput gereksinimi için optimize edilmiş REST/GraphQL mimarisi.
- **Veritabanı & Önbellek:** PostgreSQL (ACID uyumlu ilişkisel model) + Redis (Oturum, oran sınırlama ve sorgu önbelleği).
- **Asenkron Görevler:** BullMQ / Redis veya SQS tabanlı hafif bildirim kuyruğu.
- **CI/CD & Dağıtım:** GitHub Actions, Docker, Kubernetes/Cloud Run, Terraform.

---

## 9. Doğrulama Kapısı ve Onay (Quality Gate Sign-Off)
- [x] **Çözülen Problem Tanımı Netleştirildi:** Bireysel zaman ve görev yönetimi üretkenlik platformu.
- [x] **SMART Metrikler Belirlendi:** P95 < 100ms, %99.9 Uptime, >= 500 RPS, D30 > %35, Tamamlama > %70.
- [x] **Kapsam Dışı 5 Kritik Madde Tanımlandı:** Takım işbirliği, ödeme altyapısı, yapay zeka otopilot, 2-way takvim senkronu, masaüstü binary uygulamaları hariç tutuldu.
- [x] **MVP Teslimat Hedefi & FinOps Bütçesi Sabitlendi:** 17 Ekim 2026, Max 500 USD/Ay.

**Mülakat İcra Sorumlusu:** Antigravity Interrogator & Strategic Director  
**Onaylayan Paydaş:** Proje Sahibi  
**Durum:** **BAŞARILI (A1 Adımı Tamamlandı — A2 Fazına Geçişe Hazır)**
