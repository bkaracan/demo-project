# DEMO-PROJECT: AGENTIC SDLC MASTER CONSTITUTION & RULES
Bu proje, Antigravity CLI (agy) otonom ajanları tarafından SDLC fazlarına (P1-P7) ve A1-A5 adımlarına sıkı bağlılıkla geliştirilmektedir.

## 1. TEMEL AJAN ÇALIŞMA KURALLARI
- Her faz sırasıyla yürütülür: P1 (Planlama) -> P2 (Gereksinim) -> P3 (Tasarım) -> P4 (Geliştirme) -> P5 (Test) -> P6 (Dağıtım) -> P7 (Bakım/İzleme).
- İlgili fazın kalite kapısı (Quality Gate) onaylanmadan bir sonraki faza geçilemez.
- Kod yazılmadan önce kesinlikle birim ve entegrasyon testleri yazılmalıdır (TDD Red-Green-Refactor zorunluluğu).
- Proje köküne API key, parola veya hassas veri (secrets) commit edilemez. GitLeaks kontrolleri atlanamaz.
- Tüm commit mesajları Conventional Commits ('feat:', 'fix:', 'test:', 'refactor:', 'chore:') formatında olmalıdır.

## 2. GÖREVLİ SUBAGENT KADROSU
- finops-risk-analyst (P1)
- requirements-engineer (P2)
- system-architect, security-architect, database-architect, api-designer (P3)
- core-developer, code-reviewer (P4)
- qa-automation-engineer, performance-engineer (P5)
- cloud-deployer (P6)
- sre-incident-responder (P7)
