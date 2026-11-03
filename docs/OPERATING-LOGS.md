# ELAS — Operating Logs & KPI

Versi 1.0 · 3 Nov 2026 · Pemilik: Founder · Turunan `PRD-ELAS.md` §12 & §14
Isi untuk setiap task pilot (M3) + ringkasan berkala.

---

## 1. Log task

| Task ID | Project | Risk | Route (Paperclip / direct) | Model | Run count / review round | Wall-clock + review mnt | Cost | Quality (first pass? temuan QA) | Outcome | Alasan |
|---------|---------|------|----------------------------|-------|--------------------------|------------------------|------|--------------------------------|---------|--------|
| Pilot-01 | lfc | R0 | Paperclip | nemotron-3.5-super-vl-free | 1 run | ~25 min (model slow) + 0 review mnt | $0 (free) | first pass, runner RC=0 | ✅ Selesai (issue done) | Agent menambah test method, commit e7177a9, runner verify 65 tests pass |
| Pilot-02 | lfc | R0 | Paperclip | nemotron-3.5-super-vl-free | 1 run | ~5 min + 0 review mnt | $0 (free) | first pass, runner RC=0 | ✅ Selesai (issue done) | Agent menambah test method, commit 41de129, runner verify 65 tests pass |

Catatan pengisian:
- **Review minutes** = waktu aktif manusia (membaca + memutuskan) per task —
  ini metrik pembanding utama vs cara manual.
- **Cost** = dari cost event (`paperclipai cost summary`); model free = $0
  (tetap tercatat!).
- **Quality**: first pass? berapa putaran review? defect S1–S4 oleh QA?
- **Route**: `Paperclip` (chain penuh) vs `direct` (OpenCode langsung + CI) —
  pasangan perbandingan untuk evaluasi mode hybrid.

## 2. Rekap KPI (isi setelah ≥15 task)

| Metrik | Target | Aktual | Catatan |
|--------|--------|--------|---------|
| Accepted first pass | ≥ 60% | | |
| Accepted ≤ 2 putaran | ≥ 80% | | |
| Unauthorized action | **0** (mutlak) | | |
| Critical regression | **0** (mutlak) | | |
| Review minutes/task | < cara manual | | Bandingkan dengan baseline manual Anda |
| Cost per accepted task | tercatat 100% | | |
| Traceability | 100% | | |

## 3. Log insiden

| ID | Tanggal | Aktor | Level | Ringkasan | Status |
|----|---------|-------|-------|-----------|--------|
| | | | S1–S4 | | |

## 4. Change events (model/config/policy/budget)

| Tanggal | Perubahan | Alasan | Cohort evaluasi baru? |
|---------|-----------|--------|-----------------------|
| | | | |

## 5. Keputusan milestone

| Milestone | Tanggal | Keputusan | Dasar |
|-----------|---------|-----------|-------|
| M0–M4 | | | |
