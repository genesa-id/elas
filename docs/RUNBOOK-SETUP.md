# ELAS — Runbook Instalasi Server Linux

Versi 1.0 · 3 Nov 2026 · Target: Ubuntu Server 24.04 LTS (server lokal)
Pemilik: Founder (LFC) · Turunan `PRD-ELAS.md` §11 · Estimasi: 2–4 jam
Prinsip: **setiap langkah punya perintah nyata + verifikasi**. Jangan lanjut
sebelum verifikasi langkah hijau.

---

## 0. Prasyarat

| Item | Kebutuhan |
|------|-----------|
| Server | Ubuntu 24.04 LTS, minimal 4 vCPU / 8 GB RAM / 128 GB disk |
| Akses | SSH sebagai user dengan sudo (bukan root langsung) |
| Jaringan | Internet untuk install + model API; UI board cukup LAN |
| Akun | **Cline API key** (langganan) + **OpenRouter key** (free tier cukup) |
| Repo | Git repo project yang akan dibantu (2–3 awal) |

## 1. Struktur & user

```bash
sudo mkdir -p /srv/elas/{projects,worktrees,runner,backups} /opt/elas /var/log/elas
sudo useradd -r -s /usr/sbin/nologin elas-svc        # akun service Paperclip
sudo useradd -m -s /bin/bash elas-agent              # akun agent (non-root)
sudo usermod -aG docker elas-agent                   # HANYA untuk runner via sudo; lihat §5
```
Verifikasi: `ls /srv/elas` · `id elas-agent`.

> Catatan: bila ingin lebih ketat, TIDAK perlu menambahkan elas-agent ke docker —
> wrapper runner berjalan sebagai root via sudo dan menjalankan docker sendiri
> (disarankan; sesuai GENESA).

## 2. Instal dependensi

```bash
# Node 24 (pin versi; jangan auto-update)
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs docker-ce docker-ce-cli containerd.io git
node -v && docker --version
```

## 3. Paperclip (control plane) — systemd SYSTEM unit

```bash
sudo npm install -g paperclipai@<versi-pin>
sudo tee /etc/systemd/system/elas-paperclip.service >/dev/null <<'UNIT'
[Unit]
Description=ELAS Paperclip server
After=network-online.target
[Service]
User=elas-svc
Environment=HOME=/srv/elas
ExecStart=/usr/bin/paperclipai run --instance default --force
Restart=always
RestartSec=5
[Install]
WantedBy=multi-user.target
UNIT
sudo systemctl daemon-reload && sudo systemctl enable --now elas-paperclip
curl -s -o /dev/null -w 'HTTP %{http_code}\n' http://127.0.0.1:3100/api/health
```
Harus `HTTP 200`. Klaim board via UI `http://<ip-server>:3100` (bind LAN +
`allowed-hostname` sesuai kebutuhan).

## 4. Secret (WAJIK — ikuti SECURITY-POLICY §1)

1. Buat secret `OPENROUTER_API_KEY` dan `CLINE_API_KEY` via board/CLI
   (`paperclipai secrets create`) — **nilai masuk langsung, jangan lewat chat**.
2. Untuk jalur non-Paperclip: file `/srv/elas/.secrets/*.env`, **mode 600**,
   format `VAR=nilai` — **validasi format SEBELUM di-source**.
3. Verifikasi: `paperclipai secrets list` (metadata only) — nilai tidak
   pernah terlihat.

## 5. OpenCode (runtime agent)

```bash
# versi pin — sesuaikan dengan yang teruji (GENESA: 1.19.x)
npm install -g opencode-ai@<versi-pin>
mkdir -p /srv/elas/.config/opencode
cp docs/templates/opencode-global.json /srv/elas/.config/opencode/opencode.json
```
Catatan: HOME elas-svc/elas-agent mengarah ke `/srv/elas` (atau pasang di
`~/.config/opencode/` masing-masing). Validasi: `opencode models` (harus
menyertakan model `cline-pass/*` yang terdaftar).

## 6. Test runner (per project)

```bash
sudo tee /opt/elas/run-tests.sh >/dev/null <<'SH'
#!/bin/bash
# Wrapper runner ELAS — root-owned. Verbs: test|typecheck|lint|verify|reset-db|selftest
set -euo pipefail
[ "$(id -u)" = 0 ] || { echo "harus via sudo"; exit 1; }
. /opt/elas/runner.conf            # WS=, NET=, DB_URL=, IMAGE_TAG= per project
VERB="${1:-}"; SUITE="${2:-}"
case "$VERB" in
  test)      [ -n "$SUITE" ] || exit 2; [[ "$SUITE" =~ ^[a-zA-Z0-9*._/,-]+$ ]] || exit 2
             CMD=(node scripts/run-tests.mjs "$SUITE"); MEM=8g; TMO=900 ;;
  typecheck) [ -z "$SUITE" ] || exit 2; CMD=(npx tsc --noEmit); MEM=8g; TMO=900 ;;
  lint)      [ -z "$SUITE" ] || exit 2; CMD=(npx eslint .); MEM=4g; TMO=600 ;;
  verify)    [ -z "$SUITE" ] || exit 2; CMD=(npm run verify); MEM=8g; TMO=3600 ;;
  reset-db)  [ -z "$SUITE" ] || exit 2; CMD=(bash -c "npx prisma db push --force-reset"); MEM=2g; TMO=600 ;;
  selftest)  [ -z "$SUITE" ] || exit 2; CMD=(bash /opt/elas/selftest.sh); MEM=512m; TMO=120 ;;
  *) echo "verb tidak dikenal"; exit 2 ;;
esac
timeout --signal=TERM --kill-after=30 "$TMO" docker run --rm \
  --user 999:999 --network "$NET" --cpus=2 --memory="$MEM" --memory-swap="$MEM" \
  -v "$WS":/workspace:rw -e DATABASE_URL="$DB_URL" -e JWT_SECRET="$JWT_TEST" \
  --entrypoint bash "$IMAGE_TAG" -c 'cd /workspace && exec "$@"' _ "${CMD[@]}"
echo "$(date -Is) verb=$VERB suite=$SUITE rc=$? ws=$WS" >> /var/log/elas/runner.log
SH
sudo chmod 755 /opt/elas/run-tests.sh
echo 'elas-agent ALL=(root) NOPASSWD: /opt/elas/run-tests.sh *' | sudo tee /etc/sudoers.d/elas-runner
sudo chmod 440 /etc/sudoers.d/elas-runner && sudo visudo -c
```

**Image runner per project** (build saat lockfile berubah — fase prep
berinternet; container run TANPA internet):
```bash
docker build -t "elas-runner:<hash-lockfile>" /srv/elas/projects/<project>
```
`selftest.sh` = uji keamanan a–f (env tanpa key, egress terblokir, host
tak ter-mount, docker.sock tak terlihat, uid non-root, hanya /workspace) —
adaptasi dari GENESA `selftest.sh` (lihat `NEG-TEST-RUNBOOK.md`).

Verifikasi: `sudo -u elas-agent sudo /opt/elas/run-tests.sh selftest` → SEMUA-LULUS.

## 7. Git hooks per project

```bash
cd /srv/elas/projects/<project>
git config core.hooksPath .githooks
# .githooks/pre-commit: tolak commit main/master + secret scan (tanpa cetak nilai)
#   + tolak staging .env/node_modules. Root-owned 555.
```
Salin pola hook dari GENESA `templates/pre-commit-hook.sh`.
Verifikasi: `git checkout -b main-test && git commit --allow-empty -m x` →
**DITOLAK**; commit di branch task → **DITERIMA**.

## 8. Daftarkan project + agent di Paperclip

1. Project per repo: `POST /api/companies/:id/projects` lalu
   `POST /api/projects/:id/workspaces {cwd:"/srv/elas/projects/<project>", repoUrl, isPrimary:true}`
   — **workspace mengikuti project** (pola isolasi paralel; jangan andalkan
   `adapterConfig.cwd`).
2. Buat 4 agent (Coordinator, Developer ×2, QA, DevOps/Security) — payload
   mengikuti `roles/*.md` + profile config §5; env = secret_ref kedua key.
3. Kunci permissions: `canCreateAgents:false, canCreateSkills:false,
   canAssignTasks:false` (kecuali Coordinator `canAssignTasks:true`).
4. Budget policy per agent: `hardStopEnabled:false, warnPercent:80,
   unpricedUsagePolicy:'allow'` — **terapkan ke SEMUA scope**.
Verifikasi: `paperclipai agent list` → semua `ready`.

## 9. Backup & restore test (wajib saat instalasi!)

```bash
sudo tee /etc/systemd/system/elas-backup.service >/dev/null <<'UNIT'
[Unit]
Description=ELAS daily backup (Paperclip DB + projects)
[Service]
Type=oneshot
ExecStart=/opt/elas/backup.sh
UNIT
sudo tee /etc/systemd/system/elas-backup.timer >/dev/null <<'UNIT'
[Unit]
Description=Daily 03:00
[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true
[Install]
WantedBy=timers.target
UNIT
sudo systemctl enable --now elas-backup.timer
```
`backup.sh`: `paperclipai db:backup` + `tar` `/srv/elas/projects` →
`/srv/elas/backups/` (opsional: rsync ke lokasi terpisah).
**Uji restore SEKALI penuh** (restore DB + 1 repo ke lokasi uji) —
backup tanpa uji restore = belum ada backup.

## 10. Smoke test M0 (pintu masuk operasi)

1. Buat 1 task R0 kecil (template `templates/task-template.yaml`) →
   Developer mengerjakan → runner hijau → QA approve → Founder merge.
2. Verifikasi: commit ada, run log tercatat, cost event tercatat, `selftest` hijau.

## 11. Pemeriksaan kesehatan rutin

Lihat `CHARTER.md` §7 (30 detik). Tambahan: `journalctl -u elas-paperclip -f`
untuk debug; `docker ps` memastikan container runner tidak menumpuk (`--rm`).

---

## Riwayat versi

| Versi | Tanggal | Perubahan | Sign-off |
|-------|---------|-----------|----------|
| 1.0 | 3 Nov 2026 | Baseline (Batch 2) | ☐ Founder |
