# ELAS — Model Registry

Versi 1.0 · 3 Nov 2026 · Pemilik: Founder · Turunan `PRD-ELAS.md` §10
Aturan: setiap perubahan model per agent = **change event** tercatat di sini.

---

## 1. Provider

| Provider | Sifat | Biaya | Untuk |
|----------|-------|-------|-------|
| **OpenRouter** (`:free` models) | API key | **$0** (free tier) | Default: Developer, QA, Coordinator |
| **ClinePass** (`cline-pass/*`) | Langganan `api.cline.bot` (OpenAI-compatible) | **$9,99/bulan flat** | Analisis berat + fallback saat OpenRouter habis |

Catatan penting (F31): jalur **`cline-pass/<model>`** = langganan; ID vendor
(`xiaomi/...`, `deepseek/...`) = bayar per token (butuh Cline Credits).

## 2. Alokasi per agent

| Agent | Model utama | Fallback | Catatan |
|-------|-------------|----------|---------|
| ELAS Coordinator | OpenRouter `:free` (mis. nemotron-inkling class) | `cline-pass/mimo-v2.6-flash` | Tugas ringan: routing/ringkasan |
| Developer (×2–3) | OpenRouter `:free` (mis. `nvidia/nemotron-3.5-super-vl:free`) | `cline-pass/mimo-v2.6-flash` | Volume tinggi → free dulu |
| QA | OpenRouter `:free` (mis. `thinkingmachines/inkling:free`) | `cline-pass/deepseek-v4.1-flash-1017` | Review butuh ketelitian |
| DevOps/Security | `cline-pass/mimo-v2.6-pro` | `cline-pass/mimo-v2.6-flash` | Analisis berat (reasoning) |

**Aturan penempatan model kustom** (F34): model provider kustom wajib terdaftar
di blok `models` `opencode-global.json` — `OPENCODE_ALLOW_ALL_MODELS` saja
tidak cukup.

## 3. Kebijakan fallback (operasional)

1. Gejala: 402 / `Prompt tokens limit exceeded` / model tak merespons.
2. Tindakan: PATCH `adapterConfig.model` agent ke slug `cline-pass/*`
   (terbukti memicu re-verify otomatis) → catat change event.
3. Bila ClinePass pun bermasalah: tunda task model-berat; jalankan task
   deterministik (test/docs) yang tidak butuh model.

## 4. Budget & cost accounting

- Budget policy per agent: `hardStopEnabled:false`, `warnPercent:80`,
  `unpricedUsagePolicy:'allow'` — **terapkan ke SEMUA scope** (F22).
- **Setiap run = cost event** (`provider`, `model`, `costCents`, `costStatus`).
  Model free = `costCents 0` dengan status `reported` — **bukan "tidak dicatat"**.
- Rekonsiliasi bulanan: cost events vs dashboard provider (selisih = investigasi;
  tugas gratis tidak boleh dianggap gratis tanpa bukti).
- Naikkan/ubah budget bulanan per agent = keputusan Founder (change event).

## 5. Pin versi & review

- Pin versi OpenCode (mis. 1.19.x) — jangan auto-update diam-diam.
- Review katalog model: setiap kuartal / saat provider mengubah ketersediaan.
- Daftar model yang tersedia diverifikasi dengan `opencode models` (harus
  mencantumkan semua slug di atas).

## 6. Log perubahan

| Tanggal | Perubahan | Alasan | Sign-off |
|---------|-----------|--------|----------|
| 3 Nov 2026 | Baseline (Batch 2) | — | ☐ Founder |
