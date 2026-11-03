# ELAS — Dokumentasi Sistem

Panduan indeks dokumen turunan PRD-ELAS. Status: **Batch 1 selesai** (Batch 2
= runbook instalasi, NEG test, tasks milestone, template, model registry).

| Dokumen | Isi | Status |
|---------|-----|--------|
| `../PRD-ELAS.md` | **PRD induk** — tujuan, arsitektur, keputusan Founder | DRAFT review |
| `CHARTER.md` | Piagam operasional: prinsip, struktur folder, alur keputusan | ✅ v1.0 |
| `SECURITY-POLICY.md` | Aturan keamanan wajib (secret, permission, insiden) | ✅ v1.0 |
| `roles/coordinator.md` | Kontrak role ELAS Coordinator | ✅ v1.0 |
| `roles/developer.md` | Kontrak role Developer | ✅ v1.0 |
| `roles/qa.md` | Kontrak role QA | ✅ v1.0 |
| `roles/devops-security.md` | Kontrak role DevOps/Security Advisor | ✅ v1.0 |
| `templates/opencode-global.json` | Config OpenCode V1 (profile + provider + deny) | ✅ v1.0 |
| `RUNBOOK-SETUP.md` | Instalasi server Linux end-to-end (systemd, runner, hooks, backup+restore test, smoke M0) | ✅ v1.0 |
| `NEG-TEST-RUNBOOK.md` | 19 negative test syarat M1 (100% PASS = pagar mutlak) | ✅ v1.0 |
| `tasks/TASKS-MILESTONE.md` | Checklist M0–M4 + kriteria lulus | ✅ v1.0 |
| `templates/task-template.yaml` | Template task wajib (acceptance terukur) | ✅ v1.0 |
| `MODEL-REGISTRY.md` | Alokasi model per agent, fallback, cost accounting | ✅ v1.0 |
| `OPERATING-LOGS.md` | Log task + KPI + insiden + change events | ✅ v1.0 |

Prinsip penulisan: setiap dokumen **dapat dijalankan langsung** (perintah nyata,
keputusan tercatat) — bukan deskripsi umum. Seluruh pelajaran GENESA (F10–F34)
sudah menjadi aturan, bukan catatan kaki.
