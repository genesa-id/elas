# ELAS — Milestone & Task List (M0–M4)

Versi 1.0 · 3 Nov 2026 · Pemilik: Founder · Turunan `PRD-ELAS.md` §13
Status: ☐ = belum · ⏳ = sedang · ✅ = lulus (dengan bukti tercatat)

---

## M0 — Fondasi (estimasi 1 hari)

- [ ] Server siap + struktur `/srv/elas` + user agent (RUNBOOK §1–2)
- [ ] Paperclip jalan via systemd unit + board claimed (RUNBOOK §3)
- [ ] Secret `OPENROUTER_API_KEY` + `CLINE_API_KEY` di secret manager (§4)
- [ ] OpenCode terpasang (versi pin) + config V1 ter-deploy (§5)
- [ ] Runner: wrapper + sudoers + image per project + selftest hijau (§6)
- [ ] Smoke task R0 end-to-end (task → Developer → runner → QA → merge) — §10

**Lulus M0 bila**: smoke task selesai dengan bukti commit + runner + log.

## M1 — Dev + QA aman (estimasi 1 hari)

- [ ] Pre-commit hook terpasang + teruji (RUNBOOK §7)
- [ ] Agent 4 role dibuat `ready` + permissions terkunci (§8)
- [ ] **NEG batch 19/19 PASS** (NEG-TEST-RUNBOOK.md) — *syarat keras*
- [ ] *(Disarankan)* injected defect test: QA harus menangkap ≥4/5 defect
      suntikan — pola GENESA terbukti mengukur kualitas QA nyata

**Lulus M1 bila**: NEG 100% + nol unauthorized action.

## M2 — Multi-project (estimasi 1 hari)

- [ ] 2–3 project terdaftar + workspace per project (RUNBOOK §8.1)
- [ ] 2 lajur paralel Developer (worktree per project — pola F33)
- [ ] Uji paralel: 2 task **file sama** di lajur berbeda → **tanpa tabrakan**
- [ ] Runner per project terverifikasi (test project A ≠ project B)

**Lulus M2 bila**: isolasi paralel terbukti (cross-marker 0).

## M3 — Pilot terukur (estimasi 2–3 hari kerja)

- [ ] ≥15 task nyata R0–R1 (bug/fitur/test/dokumen/refactor) via chain penuh
- [ ] ≥5 task kontrol via OpenCode langsung (pembanding)
- [ ] Log `OPERATING-LOGS.md` lengkap per task (run count, wall-clock,
      review minutes, cost, quality, outcome)
- [ ] KPI: first-pass ≥60% · unauthorized 0 · regresi kritis 0 ·
      review minutes < cara manual · cost/task tercatat

**Lulus M3 bila**: KPI terpenuhi dari data aktual.

## M4 — Operasional (estimasi setengah hari)

- [ ] Backup + **uji restore** (RUNBOOK §9)
- [ ] Runbook troubleshooting final (pola gejala→solusi, adaptasi PANDUAN GENESA)
- [ ] Review temuan pilot: daftar perbaikan (permission, role file, dsb.)
- [ ] **Keputusan Founder**: lanjut operasional / sederhanakan — tertulis

---

## Catatan pelaksanaan

- Setiap milestone yang selesai → rangkum bukti ke file log/keputusan
  (nanti `GATE-EVIDENCE.md` bila diperlukan formalitas).
- Jangan melompat: M2 butuh M1 (NEG hijau) — ini pagar mutlak.
- Milestone boleh dipercepat dengan bantuan saya (operator) — Founder tetap
  memutuskan kelulusan tiap milestone.
