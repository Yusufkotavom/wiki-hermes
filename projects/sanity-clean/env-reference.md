# Environment Reference (`.env` and `.env.example`)

Dokumen ini merangkum variabel yang dipakai project ini, mana yang wajib, mana opsional, dan contoh cara isi untuk `frontend/.env` dan `studio/.env`.

## 1) Frontend (`frontend/.env`)

### Wajib minimal (app jalan normal)

- `NEXT_PUBLIC_SITE_URL`
- `NEXT_PUBLIC_STUDIO_URL`
- `NEXT_PUBLIC_SANITY_VISUAL_EDITING` (opsional, default aktif; set `false` untuk mematikan overlay live editor pada Draft Mode)
- `NEXT_PUBLIC_SANITY_API_VERSION`
- `NEXT_PUBLIC_SANITY_PROJECT_ID`
- `NEXT_PUBLIC_SANITY_DATASET`
- `SANITY_API_READ_TOKEN`
- `REVALIDATE_SECRET`

Catatan:
- `SANITY_API_READ_TOKEN` dipakai juga oleh build-time redirect loader di `frontend/next.config.mjs` jika dokumen `redirect` tidak terbaca secara anonymous pada dataset published.

### Wajib jika pakai fitur newsletter

- `RESEND_API_KEY`
- `RESEND_AUDIENCE_ID`
- `NEXT_RESEND_TO_EMAIL`
- `NEXT_RESEND_FROM_EMAIL`

### Wajib jika pakai SEO Ops Dashboard

- `SEO_SESSION_SECRET`
- `SANITY_AUTH_TOKEN` (agar dashboard bisa simpan setting ke dokumen `seoOpsSettings`)

Catatan build/deploy:
- Redirect build-time akan mencoba token dengan prioritas:
  - `SANITY_API_READ_TOKEN`
  - `SANITY_DEPLOY`
  - `SANITY_AUTH_TOKEN`
  - `SANITY_DEV`

### Wajib/opsional jika pakai AI Writer custom

- `AI_GATEWAY_API_KEY` (opsional jika memakai OIDC `VERCEL_OIDC_TOKEN`)
- `VERCEL_OIDC_TOKEN` (direkomendasikan untuk mode gateway di Vercel)
- `AI_WRITER_GEMINI_KEYS` (opsional fallback env, format newline/comma list)
- `AI_WRITER_GROQ_KEYS` (opsional fallback env, format newline/comma list)
- `AI_WRITER_ACTION_SECRET` (opsional header secret tambahan untuk endpoint `/api/ai/rewrite/apply`)

### Wajib jika menyimpan secret terenkripsi dari dashboard

- Salah satu:
  - `SEO_SETTINGS_ENCRYPTION_KEY`
  - `SEO_SESSION_SECRET` (dipakai fallback key enkripsi)

### Opsional legacy fallback SEO Ops (jika tidak diisi dari dashboard)

- `SEO_DASHBOARD_PASSWORD` atau `SEO_DASHBOARD_PASSWORD_SHA256`
- `SEO_AUTO_SUBMIT_ON_REVALIDATE`
- `SEO_BING_INDEXNOW_ALIAS`
- `SEO_GOOGLE_INDEXING_ENABLED`
- `SEO_GOOGLE_INDEXING_AGGRESSIVE_MODE`
- `SEO_GOOGLE_SERVICE_ACCOUNT_JSON` / `GOOGLE_APPLICATION_CREDENTIALS`
- `SEO_INDEXNOW_ENABLED`
- `SEO_INDEXNOW_KEY`
- `SEO_INDEXNOW_HOST`
- `SEO_INDEXNOW_ENDPOINT`
- `SEO_INDEXNOW_KEY_LOCATION`
- `SEO_INDEXING_BATCH_SIZE`
- `SEO_INDEXING_RETRY_ATTEMPTS`
- `SEO_GSC_PRIORITY_CSV_PATH` (tetap opsional untuk migration priority)

### Opsional untuk script migrasi/audit

- `ASTRO_SOURCE_ROOT`
- `ASTRO_SOURCE_DATA_ROOT`
- `GSC_SITE_URL`
- `GSC_START_DATE`
- `GSC_END_DATE`
- `GSC_OUT_DIR`
- `GSC_ROW_LIMIT`
- `GSC_BLOG_BASE`
- `GSC_CATEGORY_BASE`
- `GSC_MIN_IMPRESSIONS_AUTO`
- `GSC_SITEMAP_URL`
- `GSC_INPUT_CSV`
- `GSC_INPUT_COLUMN`
- `GSC_INSPECTION_CONCURRENCY`
- `GSC_INSPECTION_DELAY_MS`
- `GSC_MAX_URLS`
- `GSC_LANGUAGE_CODE`
- `MIGRATION_CSV`
- `INSPECTION_CSV`
- `METADATA_CSV`
- `MERGE_OUT_DIR`
- `SEO_AUDIT_INPUT_CSV`
- `SEO_AUDIT_INPUT_COLUMN`
- `SEO_AUDIT_OUT_DIR`
- `SEO_AUDIT_CONCURRENCY`
- `SEO_AUDIT_MAX_URLS`
- `SEO_AUDIT_TIMEOUT_MS`

---

## 2) Studio (`studio/.env`)

### Wajib minimal

- `SANITY_STUDIO_PREVIEW_URL`
- `SANITY_STUDIO_API_VERSION`
- `SANITY_STUDIO_PROJECT_ID`
- `SANITY_STUDIO_DATASET`
- `SANITY_STUDIO_HOSTNAME`
- `SANITY_AUTH_TOKEN`

### Opsional

- `SANITY_STUDIO_APP_ID` (legacy/advanced deploy target)
- `SANITY_DEV`
- `SANITY_DEPLOY`

Catatan operasional agent/script:
- Untuk otomasi import/mutasi Sanity oleh agent, gunakan kredensial **dev** terlebih dulu.
- `SANITY_DEV` dapat diisi token write dataset dev (bukan sekadar boolean flag).
- Prioritas token write yang dipakai script/agent: `SANITY_DEV` -> `SANITY_AUTH_TOKEN`.

---

## 3) Contoh `frontend/.env` (quick start)

```env
NEXT_PUBLIC_SITE_URL=http://localhost:3000
NEXT_PUBLIC_STUDIO_URL=http://localhost:3333
NEXT_PUBLIC_SANITY_VISUAL_EDITING=true
NEXT_PUBLIC_SITE_ENV=development

NEXT_PUBLIC_SANITY_API_VERSION=2026-03-23
NEXT_PUBLIC_SANITY_PROJECT_ID=replace-with-project-id
NEXT_PUBLIC_SANITY_DATASET=production
SANITY_API_READ_TOKEN=replace-with-read-token

REVALIDATE_SECRET=replace-with-strong-random-secret

RESEND_API_KEY=replace-with-resend-api-key
RESEND_AUDIENCE_ID=replace-with-resend-audience-id
NEXT_RESEND_TO_EMAIL=team@example.com
NEXT_RESEND_FROM_EMAIL=Schema UI <noreply@example.com>

SEO_SESSION_SECRET=replace-with-strong-session-secret
SANITY_AUTH_TOKEN=replace-with-sanity-write-token
# SEO_SETTINGS_ENCRYPTION_KEY=replace-with-32+chars-secret

# Optional legacy fallback:
# SEO_DASHBOARD_PASSWORD=replace-with-dashboard-password
# SEO_DASHBOARD_PASSWORD_SHA256=
# SEO_AUTO_SUBMIT_ON_REVALIDATE=true
# SEO_BING_INDEXNOW_ALIAS=false
# SEO_GOOGLE_INDEXING_ENABLED=false
# SEO_GOOGLE_INDEXING_AGGRESSIVE_MODE=true
# GOOGLE_APPLICATION_CREDENTIALS=/abs/path/service-account.json
# SEO_GOOGLE_SERVICE_ACCOUNT_JSON=
# SEO_INDEXNOW_ENABLED=false
# SEO_INDEXNOW_KEY=replace-with-indexnow-key
# SEO_INDEXNOW_HOST=www.example.com
# SEO_INDEXNOW_ENDPOINT=https://api.indexnow.org/indexnow
# SEO_INDEXING_BATCH_SIZE=100
# SEO_INDEXING_RETRY_ATTEMPTS=2
SEO_GSC_PRIORITY_CSV_PATH=./tmp/gsc/gsc-pages-priority.csv

# AI writer optional fallback:
# AI_GATEWAY_API_KEY=
# VERCEL_OIDC_TOKEN=
# AI_WRITER_GEMINI_KEYS=key1,key2,key3
# AI_WRITER_GROQ_KEYS=key1,key2,key3
# AI_WRITER_ACTION_SECRET=replace-with-strong-shared-secret

# Optional migration/audit scripts:
# ASTRO_SOURCE_ROOT=/abs/path/to/astro/src/pages
# ASTRO_SOURCE_DATA_ROOT=/abs/path/to/astro/src/data
# GSC_SITE_URL=https://www.example.com/
# GSC_START_DATE=2025-01-01
# GSC_END_DATE=2025-12-31
# GSC_OUT_DIR=./tmp/gsc
# GSC_ROW_LIMIT=25000
# GSC_BLOG_BASE=/blog
# GSC_CATEGORY_BASE=/blog/category
# GSC_MIN_IMPRESSIONS_AUTO=1
# GSC_SITEMAP_URL=https://www.example.com/sitemap.xml
# GSC_INPUT_CSV=./tmp/gsc/gsc-pages.csv
# GSC_INPUT_COLUMN=page
# GSC_INSPECTION_CONCURRENCY=3
# GSC_INSPECTION_DELAY_MS=0
# GSC_MAX_URLS=0
# GSC_LANGUAGE_CODE=id-ID
# MIGRATION_CSV=./tmp/gsc/gsc-migration-curation.csv
# INSPECTION_CSV=./tmp/gsc/gsc-url-inspection.csv
# METADATA_CSV=./tmp/gsc/seo-metadata-audit.csv
# MERGE_OUT_DIR=./tmp/gsc
# SEO_AUDIT_INPUT_CSV=./tmp/gsc/gsc-pages.csv
# SEO_AUDIT_INPUT_COLUMN=page
# SEO_AUDIT_OUT_DIR=./tmp/gsc
# SEO_AUDIT_CONCURRENCY=8
# SEO_AUDIT_MAX_URLS=0
# SEO_AUDIT_TIMEOUT_MS=15000
```

---

## 4) Contoh `studio/.env` (quick start)

```env
SANITY_STUDIO_PREVIEW_URL=http://localhost:3000
SANITY_STUDIO_API_VERSION=2026-03-23
SANITY_STUDIO_PROJECT_ID=replace-with-project-id
SANITY_STUDIO_DATASET=production
SANITY_STUDIO_HOSTNAME=replace-with-hostname
SANITY_AUTH_TOKEN=replace-with-deploy-token

SANITY_DEV=replace-with-sanity-dev-write-token
SANITY_DEPLOY=false
# SANITY_STUDIO_APP_ID=replace-with-studio-app-id
```

---

## 5) Password Dashboard via Studio (Recommended)

Set password baru di `SEO Dashboard > Settings` (field `New dashboard password`). Sistem akan menyimpan SHA256 hash ke dokumen `seoOpsSettings`.

## 6) Legacy: Generate `SEO_DASHBOARD_PASSWORD_SHA256`

Linux/macOS:

```bash
printf "your-strong-password" | sha256sum
```

Lalu isi output hash ke:

```env
SEO_DASHBOARD_PASSWORD_SHA256=<hash-hex>
```

Gunakan **salah satu** antara `SEO_DASHBOARD_PASSWORD` atau `SEO_DASHBOARD_PASSWORD_SHA256`.

## 7) Dashboard-First Setup Guide

Panduan langkah lengkap (Google API, login bootstrap, enkripsi, troubleshooting):
- `docs/seo-dashboard-setup.md`
