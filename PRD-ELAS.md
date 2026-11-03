# PRD — ELAS (LFC Assistant)

**Product Requirements Document**
Versi 1.0 · 3 November 2026 · Pemilik: Founder (LFC)
Status: DRAFT untuk review · Target deploy: server Linux lokal
Dasar: pola GENESA AI Company (terbukti Gate 0–4, 50+ task nyata) — disederhanakan
untuk development multi-project, **tanpa** agent marketing & legal.

---

## 1. Ringkasan eksekutif

**ELAS** (*LFC Assistant*) adalah tim AI internal yang membantu Founder dalam
proses development beberapa project sekaligus: menulis fitur, test, dokumentasi,
refactor, review kode, dan menjaga kualitas — dengan biaya mendekati nol,
keamanan berlapis, dan jejak audit penuh.

ELAS adalah **salinan pola GENESA AI Company** yang disesuaikan:
- ✅ Stack yang sama (Paperclip + OpenCode + Docker runner + model ganda)
- ✅ Pelajaran 24 temuan operasional (F10–F34) langsung dibakar ke desain
- ➕ **Multi-project** dari hari pertama (beberapa repo = beberapa project)
- ➖ **Tanpa** Growth/Marketing, Social, Legal — dan tanpa overhead kampanye
- 🖥️ Deploy di **server Linux lokal** (bukan WSL) — lebih bersih & stabil

---

## 2. Tujuan & non-tujuan

### Tujuan
1. Founder memberi task development dalam bahasa biasa → hasil berupa commit
   siap review yang **sudah lulus test** (CI gate) — bukan sekadar saran.
2. Dukungan **banyak project/repo** secara bersamaan dengan isolasi kerja nyata.
3. Review kualitas independen (QA) untuk setiap perubahan berarti.
4. Biaya rendah & terukur: default model gratis/langganan, semua tercatat.
5. Keamanan: agent **tidak mungkin** menyentuh secret/deploy/merge tanpa manusia.

### Non-tujuan (dilarang / out of scope)
- Agent marketing, konten, social, legal (diminta tidak diperlukan).
- Deploy otomatis ke produksi, push/merge otomatis ke branch proteksi.
- Menggantikan keputusan teknis manusia (arsitektur, rilis, keamanan data).
- Menjalankan tanpa pengawasan pada project berisiko tinggi (auth/pembayaran).

---

## 3. Pengguna & model operasi

- **Pengguna**: Founder (satu-satunya pengguna; kemanusiaan = otoritas akhir).
- **Pola kerja (hybrid — terbukti di Gate 3)**:
  - Task kecil/eksplorasi → jalur cepat (OpenCode langsung + CI gate).
  - Task berdampak/butuh jaminan → jalur Paperclip penuh (task → Developer →
    CI → QA review → **Founder approve → merge manual**).
- Manusia selalu: **satu-satunya merger, deployer, dan pemegang secret**.

---

## 4. Arsitektur sistem

```
Founder ( manusia )
   │  bahasa biasa / board UI / CLI
   ▼
┌──────────────────────────────────────────────────────────┐
│ Paperclip (control plane)                                 │
│  · task/issue per project · lifecycle agent · policy R1   │
│  · budget & cost accounting · secret manager · audit log  │
└───────────────┬──────────────────────────────────────────┘
                │ adapter opencode_local
                ▼
┌──────────────────────────────────────────────────────────┐
│ OpenCode (runtime agent) — permission per profile (V1)    │
│  Coordinator · Developer (xN paralel) · QA · DevOps/Sec   │
└───────────────┬──────────────────────────────────────────┘
                │ sudo (satu perintah) untuk test
                ▼
┌──────────────────────────────────────────────────────────┐
│ Test Runner (Docker, per project)                          │
│  container --internal · tanpa internet/secret/socket      │
│  + DB test disposable per project                          │
└──────────────────────────────────────────────────────────┘
   Model: OpenRouter :free (utama) → ClinePass (fallback berbayar)
   Workspace: git repo per project + worktree per lajur Developer
```

**Prinsip pemisahan** (dari GENESA): Paperclip = tata kelola, OpenCode =
eksekusi, Docker = pagar, model = tenaga. Jangan mencampur peran ini.

---

## 5. Agent & role contract

| Agent | Jumlah | Tugas | Batas keras |
|-------|--------|-------|-------------|
| **ELAS Coordinator** | 1 | Menerima task Founder, memecah & memilih project/agent, merangkum status | Tidak mengerjakan sendiri; tidak membuat agent; tidak memutuskan risiko |
| **Developer** | 2–3 (paralel) | Implementasi fitur/test/dokumen/refactor per project, commit lokal, jalankan runner | Tidak push/merge/deploy; tidak sentuh path terproteksi; tidak ubah config keamanan |
| **QA** | 1 | Review independen: diff vs acceptance, jalankan runner, laporan defect S1–S4 + verdict | Read-only; tidak memperbaiki sendiri; tidak menyetujui pekerjaannya sendiri |
| **DevOps/Security** | 1 (on-demand) | Review keamanan infra/kode, post-mortem insiden, rekomendasi hardening | Read-only; tanpa credential produksi; tanpa deploy |

Semua agent: `canCreateAgents=false`, `canCreateSkills=false`,
`canAssignTasks=false` (kecuali Coordinator `canAssignTasks=true`),
heartbeat **off** kecuali bila kelak diputuskan lain.

**Profile permission OpenCode (V1 — WAJIB bentuk `agent` tunggal!)**:
- `elas-developer` — edit/write hanya di project aktif; bash: `git add/commit/
  switch -c task/*`, `sudo /opt/elas/run-tests.sh *`; **deny** `--amend`,
  `--no-verify`, `;`, `&&`, `|`, `>`, `<`, `$(`, backtick (urutan TERAKHIR).
- `elas-qa` — read-only + `sudo /opt/elas/run-tests.sh *` + git read-only.
- `elas-readonly` — read-only penuh (Coordinator/DevOps).

---

## 6. Desain multi-project (inti pembeda)

Aturan emas (F33 dari GENESA): **workspace mengikuti PROJECT** — isolasi paralel
hanya terjamin bila tiap lajur kerja punya PROJECT/worktree sendiri.

```
/srv/elas/
├── projects/
│   ├── project-alpha/        ← git repo alpha  (project Paperclip "Alpha")
│   ├── project-beta/         ← git repo beta   (project Paperclip "Beta")
│   └── ...
├── worktrees/                ← lajur paralel per project (bila perlu)
├── runner/                   ← wrapper runner + config per project
└── backups/
```

- Setiap repo terdaftar sebagai **project** di Paperclip dengan execution
  workspace menunjuk ke foldernya (`POST /api/projects/:id/workspaces`).
- **Parallel**: bila satu project butuh 2+ Developer bersamaan, buat git
  worktree + project kerja sementara (pola terbukti: 10 task paralel nol
  tabrakan). `adapterConfig.cwd` TIDAK cukup — ingat F25.
- **Branch**: task = `task/<id>` lokal; `main`/`master` = hanya manusia
  (pre-commit hook menolak commit di main/master).

---

## 7. Workflow task (lifecycle resmi)

1. **Founder memberi task** (bahasa biasa / board) → Coordinator menyusun
   task template: objective, scope, **acceptance criteria terukur**, risiko R0–R3.
2. **Dispatch**: assignee = Developer (atau Developer paralel untuk volume).
3. **Eksekusi**: Developer mengerjakan di branch `task/*`, commit lokal,
   menjalankan runner (CI gate).
4. **Submit → intercept policy R1** → **QA review** (independen) →
   **Founder approve** → **merge manual**.
5. **Catatan**: metrik per task (run count, wall-clock, review minutes, cost,
   outcome) — log operasional.

Kunci sukses GENESA (100% first-pass): **acceptance criteria objektif** +
kewajiban melaporkan bukti (commit hash, exit code runner).

---

## 8. Test runner & CI gate (per project)

- Wrapper root-owned: `/opt/elas/run-tests.sh` — verbs: `test <glob>`,
  `typecheck`, `lint`, `verify`, `reset-db`, `selftest`. Argumen verb tetap
  **menolak argumen tambahan** (exit 2).
- Docker: image per project (dependensi baked, tag = hash lockfile), jaringan
  `--internal` (tanpa egress), `--cpus/--memory` per verb, env **hanya**
  `DATABASE_URL`+`JWT_SECRET` sintetis — **tanpa** key provider apa pun.
- Sudoers: `NOPASSWD: /opt/elas/run-tests.sh *` — **satu-satunya** perintah sudo.
- `selftest` reusable (uji keamanan a–f: env tanpa key, egress terblokir,
  file host tak ter-mount, docker.sock tak terlihat, uid non-root, hanya
  /workspace) — jalankan berkala (mingguan).
- Runner menjalankan test di **workspace yang benar** (per project). Karena
  runner bersifat per-project, konfigurasikan `runner.conf` per project atau
  verb `test --project <id>` (putusan implementasi).

---

## 9. Keamanan & permission (FINDINGS wajib ditanam)

Dari 24 temuan GENESA — ini **bukan saran**, ini syarat:

| # | Aturan (sumber temuan) |
|---|------------------------|
| 1 | Config OpenCode pakai key **`agent` (tunggal)** + pisahkan `write`/`patch`/`edit` (F10/F11) |
| 2 | Secret hanya di secret manager (Paperclip) + env run; update nilai = **`POST /api/secrets/:id/rotate`** (F28 — PATCH hanya metadata!) |
| 3 | File env lokal WAJIB format `VAR=nilai` — **bukan nilai mentah** (INCIDENT-001) |
| 4 | Model provider kustom **wajib terdaftar** di blok `models` config (F34-saudara) |
| 5 | Task tidak boleh menunjuk file di luar workspace (F30) → desain task benar |
| 6 | Larang `--no-verify`, `--amend`, karakter `;`/`&&`/`|`/redirect (F32/F11) |
| 7 | Pre-commit hook: tolak commit main/master + secret scan + tolak staging `.env`/`node_modules` |
| 8 | **Jangan pernah cancel run** (F16 — zombie akuntansi) |
| 9 | Model self-report tidak reliable (P-E) → percaya log/verifikasi |
| 10 | Klaim status lingkungan wajib berbukti perintah+exit code (P-A) — tulis di role file |

---

## 10. Model & budget strategy

| Prioritas | Sumber | Untuk | Biaya |
|-----------|--------|-------|-------|
| 1 | OpenRouter `:free` (nemotron/inkling dll.) | Developer, QA, Coordinator volume | **$0** |
| 2 | ClinePass (`cline-pass/*`) | Analisis berat, fallback saat OpenRouter habis | $9,99/bulan flat |
| 3 | OpenRouter paid | Hanya bila kredit tersedia | per token |

- Kebijakan fallback (F31): saat 402/habis → PATCH model agent ke `cline-pass/*`.
- Budget policy per agent: **`hardStopEnabled:false`** + `unpricedUsagePolicy:'allow'`
  (F22 — jangan lupa: keputusan hard-off harus diterapkan ke SEMUA scope!) dengan
  `warnPercent` 80 sebagai pengingat.
- Cost event wajib per run (`provider`, `model`, `costCents`, `costStatus`) —
  rekonsiliasi berkala dengan dashboard provider (G3-04 pola).

---

## 11. Infrastruktur: server Linux lokal

| Komponen | Pilihan | Catatan |
|----------|---------|---------|
| OS | Ubuntu Server 24.04 LTS | — |
| Paperclip | **systemd system unit** `elas-paperclip.service` (`Restart=always`) | Bukan user-lingering — server asli lebih bersih dari WSL (F21 tak berlaku) |
| OpenCode | terpasang resmi, versi pin (mis. 1.19.x) | Jangan auto-update diam-diam |
| Docker | Docker Engine + image runner per project | Tanpa socket ke agent |
| Backup | systemd timer harian (DB Paperclip + `/srv/elas/projects`) → lokasi terpisah | Uji restore saat instalasi (G0-03 GENESA) |
| Jaringan | Server lokal; UI board hanya LAN/VPN | `server.bind=lan` + `allowed-hostname` |
| Waktu | NTP aktif (F22 GENESA: jam skew merusak) | `timedatectl` |
| Anti-lupa | Tidak perlu keepalive (bukan WSL) | — |

**Checklist instalasi (ringkas)**: hardening user non-root untuk agent →
pasang Paperclip + klaim board → deploy config OpenCode (template di §5) →
buat image runner per project → sudoers + hook → selftest hijau → backup +
uji restore → smoke 1 task end-to-end.

---

## 12. Observability & audit

- **Log per task**: OPERATING-LOGS (task id, risiko, route, run count,
  wall-clock, review minutes, cost, quality, outcome) — minimal 1×/minggu
  rekap.
- **Audit trail**: semua run + komentar + commit + keputusan tercatat
  (Paperclip sudah menyediakan).
- **Health check** (30 detik, kapan pun):
  `agent list` · `cost summary` · `run-tests.sh selftest` · `git log` per project.
- **Insiden**: template incident report; klasifikasi S1–S4; S1 = eskalasi ke
  Founder segera.

---

## 13. Milestone implementasi (gate ringkas)

| Milestone | Isi | Kriteria lulus |
|-----------|-----|----------------|
| **M0 — Fondasi** | Server, Paperclip, OpenCode, secret, model fallback | Smoke task R0 selesai via chain penuh; selftest hijau |
| **M1 — Dev + QA aman** | Permission V1, runner per project, pre-commit hook, policy R1 | **NEG batch** (minimal: edit path terproteksi ditolak, `git push` ditolak, `.env` commit ditolak, printenv tanpa secret, subagent ditolak, `--amend`/`--no-verify` ditolak) **100% PASS** |
| **M2 — Multi-project** | 2+ project terdaftar, worktree paralel, isolasi teruji | 2 task paralel file-sama **tanpa tabrakan** (pola F33) |
| **M3 — Pilot terukur** | ≥15 task nyata R0–R1 + kelompok kontrol 5 task direct | KPI: first-pass ≥60%, unauthorized 0, review minutes tercatat, cost/task tercatat |
| **M4 — Operasional** | Hardening final, runbook, keputusan lanjut | Dokumentasi lengkap; Founder memutuskan status akhir |

Catatan: injected-defect test (QA ≥4/5) **sangat disarankan** di M1 (terbukti
menilai kualitas QA nyata di GENESA — hasil 5/5).

---

## 14. Metrik sukses

| Metrik | Target |
|--------|--------|
| Accepted first-pass | ≥ 60% (GENESA: 100%) |
| Unauthorized action | **0** (mutlak) |
| Critical regression | **0** (mutlak) |
| Review minutes / task | jauh di bawah cara manual (GENESA: 1–2 mnt vs 24 mnt) |
| Cost / accepted task | tercatat 100% (target $0–kecil) |
| Traceability | 100% task punya bukti run + keputusan |

---

## 15. Risiko & mitigasi (pelajaran GENESA)

| Risiko | Mitigasi |
|--------|----------|
| Model "mengarang" status/hasil | Kewajiban bukti perintah+exit code (role file) + verifikasi QA |
| Zombie akuntansi dari run gagal | Jangan cancel run; jalur repair `accounting/retry` |
| Kebocoran secret | Secret manager + selftest + neg test; larangan keras di role file |
| Tabrakan kerja paralel | Wajib pola project-per-worktree (F33) |
| Ketergantungan provider | Fallback ganda (free + ClinePass); MODEL-REGISTRY hidup |
| Scope creep agent | Permission deny berlapis + protected paths + review manusia |
| Biaya tak terkendali | Budget warn 80% + cap provider + rekonsiliasi berkala |

---

## 16. Lampiran

### A. Template task (terbukti 100% first-pass di GENESA)
```
Tugas R0/R1. (1) <aksi spesifik + file tujuan>. (2) <batasan: jangan ubah X>.
(3) Verifikasi: sudo /opt/elas/run-tests.sh test '<glob>' / typecheck.
(4) Commit: git add <file> && git commit -m '<pesan>'.
Laporkan: branch + hash + hasil verifikasi.
```

### B. Keputusan yang diminta pada Founder saat review PRD ini
1. Daftar project pertama yang didukung (2–3 repo awal).
2. Pilihan model: langsung pakai pola free + ClinePass? (disarankan)
3. Batas risiko: R0–R1 saja, atau izinkan R2 dengan gate khusus?
4. Lokasi server & skema backup.

### C. Referensi pola
Salin bagian terkait dari GENESA AI Company: `templates/opencode-global.json`
(sesuaikan profile), `NEG-TEST-RUNBOOK.md`, `repository-baseline.md` §7
(runner), `MODEL-REGISTRY.md`, `PANDUAN-PENGGUNAAN.md` (adaptasi untuk LFC).
