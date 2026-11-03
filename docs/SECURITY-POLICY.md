# ELAS — Security Policy

Versi 1.0 · 3 November 2026 · Pemilik: Founder (LFC)
Status: WAJIB dipatuhi seluruh operasi — pelanggaran = insiden (§5)
Dasar: `PRD-ELAS.md` §9 · sumber: 24 temuan operasional GENESA (F10–F34)

---

## 1. Manajemen secret (paling sering bocor di dunia nyata)

1. Secret (API key, token) **hanya** hidup di: (a) secret manager Paperclip;
   (b) env proses run (di-inject via `secret_ref`); (c) file lokal terbatas
   `mode 600` **dengan format `VAR=nilai`** untuk jalur non-Paperclip.
2. **DILARANG**: nilai secret di chat, komentar issue, dokumen, kode, log,
   argumen perintah, atau output error yang dicetak ulang.
3. Mengubah nilai secret = **`POST /api/secrets/:id/rotate`** (endpoint
   PATCH hanya metadata — nilai tidak berubah; F28).
4. File env lokal **wajib divalidasi formatnya SEBELUM di-source** (nilai
   mentah tanpa `VAR=` akan terekspos ke log oleh shell — INCIDENT-001).
5. Rotasi: setiap kecurigaan kebocoran → rotate segera + catat insiden.
6. Agent tidak memiliki kemampuan membaca nilai secret keluar dari env run-nya.

## 2. Permission agent (deny by default)

1. Config OpenCode memakai skema **V1**: key **`agent` (tunggal)** — bukan
   `agents` (dibuang diam-diam oleh parser; F10).
2. `edit`, `write`, `patch` diatur **terpisah** (F11) — ketiganya wajib
   disetel.
3. Deny karakter injeksi **dipasang paling akhir** agar menang atas allow:
   `;` `&&` `||` `|` `>` `<` `$(` `` ` `` (last-match-wins).
4. Deny eksplisit: `git commit --amend*`, `git commit*--no-verify*`,
   `git commit*-n*`, `git push*`, `git config*`, `git remote*`, `git reset*`.
5. Sudo agent **hanya** `/opt/elas/run-tests.sh *` (NOPASSWD) — semua lain
   ditolak sudoers.
6. Model provider kustom **wajib terdaftar** di blok `models` config (F34
   saudara: `ALLOW_ALL_MODELS` saja tidak cukup).
7. Protected path (default; disesuaikan per project saat onboarding):
   `.github/**`, `opencode.json`, `.opencode/**`, `**/.env*`, `**/*.pem`,
   `**/*.key`, `**/migrations/**`, `**/deploy*/**`, `scripts/**`, `docker/**`.

## 3. Isolasi eksekusi (runner)

1. Test berjalan di container Docker `--internal` (tanpa egress), tanpa
   docker.sock, tanpa secret, uid non-root, hanya `/workspace` yang ter-mount.
2. Env container hanya `DATABASE_URL` + `JWT_SECRET` sintetis (bukan rahasia
   nyata — DB test disposable).
3. `selftest` (a–f) wajib hijau; jalankan **mingguan** dan setiap perubahan
   infra.
4. Egress dilarang untuk test; bila kelak dibutuhkan = izin Founder + allowlist
   eksplisit + review.

## 4. Git & rantai rilis

1. `main`/`master` = **hanya manusia**; pre-commit hook menolak commit di
   main/master + secret scan (tanpa mencetak nilai) + menolak staging
   `.env`/`node_modules`.
2. Hook **root-owned 555** — agent tidak dapat mengubah.
3. Task = branch `task/*` lokal; merge manual oleh Founder.
4. **Jangan pernah cancel run** (F16) — biarkan timeout; cancel = zombie
   akuntansi.

## 5. Klasifikasi insiden & respons

| Level | Kriteria | Respons |
|-------|----------|---------|
| **S1 blocker** | Secret bocor; akses tanpa izin; kehilangan data | Hentikan jalur terkait → eskalasi Founder **segera** → post-mortem DevOps/Security |
| **S2 major** | Pelanggaran permission berdampak; regresi serius | Perbaiki sebelum merge; catat |
| **S3 minor** | Bug kecil; misleading output | Perbaiki pada task berikutnya |
| **S4 trivial** | Kosmetik | Catat bila relevan |

Insiden mencakup **perilaku agent** (mengarang status = S3; mencoba bypass
permission = S2; berhasil bypass = S1).

## 6. Verifikasi berkala (tanggung jawab Founder/operator)

| Aktivitas | Frekuensi |
|-----------|-----------|
| `selftest` runner | Mingguan |
| Audit run log acak (kebenaran laporan agent) | Mingguan |
| Rekonsiliasi cost event vs provider | Bulanan |
| Rotasi secret | Minimal per kuartal / saat curiga |
| Uji negative-test ulang (subset) | Setiap perubahan config permission |

## 7. Riwayat versi

| Versi | Tanggal | Perubahan | Sign-off |
|-------|---------|-----------|----------|
| 1.0 | 3 Nov 2026 | Baseline (turunan PRD-ELAS Batch 1) | ☐ Founder |
