# ELAS — Piagam Operasional (CHARTER)

Versi 1.0 · 3 November 2026 · Pemilik: Founder (LFC)
Posisi: turunan operasional `PRD-ELAS.md` §3–§4

---

## 1. Apa ELAS ini (satu kalimat)

Tim AI internal berbasis Paperclip + OpenCode yang membantu Founder mengembangkan
beberapa project dengan hasil berupa **commit siap merge yang sudah lulus test** —
murah, aman berlapis, dan teraudit penuh.

## 2. Prinsip operasional (tidak dapat ditawar)

1. **Manusia = otoritas akhir.** Satu-satunya merger, deployer, pemegang
   keputusan risiko. Agent tidak pernah push/merge/deploy.
2. **Bukti mengalahkan klaim.** Setiap hasil wajib disertai commit hash,
   exit code runner, atau keluaran terverifikasi. Klaim tanpa bukti = ditolak.
3. **Deny by default.** Permission agent bersifat daftar-izinkan; semua sisanya
   ditolak — termasuk pola injeksi (`;`, `&&`, `>`, `--no-verify`, dsb.).
4. **Secret tidak pernah berpindah** keluar secret manager + env run.
5. **Verifikasi sebelum percaya.** Test = keputusan CI (runner), review = QA
   independen, keputusan = Founder.
6. **Semua keputusan tercatat.** Setiap perubahan model/config/policy/budget
   menjadi catatan change-event.

## 3. Struktur folder (server Linux)

```
/srv/elas/
├── projects/            # repo per project (execution workspace per project)
│   ├── <project-a>/     # git repo — branch task/* lokal; main = manusia
│   └── <project-b>/
├── worktrees/           # lajur paralel per project (pola project-per-worktree)
├── runner/              # /opt/elas/run-tests.sh + runner.conf + image tag
├── backups/             # timer harian: DB Paperclip + projects
└── docs/ ← salinan repo dokumentasi ini
```

Dokumen sumber (repo `elas` di Git): struktur `docs/` ini.

## 4. Alur keputusan (siapa memutuskan apa)

| Keputusan | Pemutus | Peran agent |
|-----------|---------|-------------|
| Risiko R0/R1 (dokumen/test/refactor kecil) | Developer + CI gate | Eksekusi penuh |
| Risiko R2/R3 (auth, data, deploy, arsitektur) | **Founder** (diskusi sebelum eksekusi) | Hanya analisis/rekomendasi |
| Kualitas suatu perubahan | QA (verdict) → Founder (approve) | QA = review independen |
| Merge ke main | **Founder** | Tidak ada |
| Model/config/permission/budget | **Founder** (tercatat sebagai change event) | Tidak ada |
| Insiden S1 (keamanan/secret) | Founder — eskalasi segera | DevOps/Security = post-mortem |

## 5. Arus kerja standar (ringkas)

```
Founder (bahasa biasa) → Coordinator (task template) → Developer (branch task/*)
  → runner CI gate → QA review → Founder approve → Founder merge
```

Task kecil/eksplorasi boleh memakai jalur cepat (OpenCode langsung + CI gate) —
keputusan **mode hybrid** (dasar: hasil pilot GENESA Gate 3).

## 6. Anggaran & model (ringkas)

- Default: model **gratis** (OpenRouter `:free`); fallback: **ClinePass**
  (`cline-pass/*`, langganan flat $9,99/bulan).
- Budget policy per agent: `hardStopEnabled:false` + `warnPercent:80`
  (**terapkan ke SEMUA scope** — pelajaran F22), `unpricedUsagePolicy:'allow'`.
- Setiap run tercatat sebagai cost event; rekonsiliasi berkala.

## 7. Kesehatan sistem (cek 30 detik, kapan pun)

```bash
systemctl status elas-paperclip            # server hidup
paperclipai agent list -C <companyId>      # semua agent idle/ready
sudo /opt/elas/run-tests.sh selftest       # keamanan runner hijau
paperclipai cost summary -C <companyId>    # biaya terkendali
```

## 8. Larangan mutlak

- Menjalankan task R2+ tanpa diskusi Founder.
- Menambah sudo/rules/permission di luar jalur change-event resmi.
- Membatalkan (cancel) run yang sedang berjalan — biarkan timeout bekerja.
- Menyimpan secret di file selain lokasi resmi (format `VAR=nilai`, mode 600).
- Menjalankan agent tanpa laporan bukti (commit hash / exit code runner).

## 9. Riwayat versi

| Versi | Tanggal | Perubahan | Sign-off |
|-------|---------|-----------|----------|
| 1.0 | 3 Nov 2026 | Baseline (turunan PRD-ELAS Batch 1) | ☐ Founder |
