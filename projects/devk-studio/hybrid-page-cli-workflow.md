# Hybrid Page CLI Workflow

Dokumen ini menjelaskan cara cepat dan aman untuk mengubah page utama menjadi hybrid:

- route shell tetap code-owned
- konten support atas/bawah diambil dari Sanity `page`
- pemisahan block dikendalikan oleh `topBlockCount`

Workflow ini cocok untuk page utama seperti:

- `index`
- `layanan`
- `pembuatan-website`
- `percetakan`
- `software`

## Arti `Supported Hybrid Slug`

Istilah ini lebih tepat daripada `eligible`.

Yang dimaksud `supported hybrid slug` adalah:

- slug yang route frontend-nya sudah memakai `PageHybridShell`
- atau slug baru yang akan di-bootstrap lewat `hybrid:route:create`

Contoh slug yang saat ini sudah supported:

- `index`
- `layanan`
- `pembuatan-website`
- `percetakan`
- `software`

Kalau slug sudah supported:

- Anda bisa langsung pakai `hybrid:create`
- action `Apply Hybrid Preset` bisa muncul di Studio

Kalau slug belum supported:

- `hybrid:create` saja tidak cukup
- karena script itu hanya membuat atau mengubah document `page` di Sanity
- route frontend belum tentu membaca document tersebut

Untuk membuat slug baru menjadi supported hybrid slug:

1. wire route ke `PageHybridShell` secara manual
2. atau gunakan `hybrid:route:create --write`

Rule sederhananya:

- slug sudah supported -> pakai `hybrid:create`
- slug belum supported -> pakai `hybrid:route:create`

## Tujuan

CLI ini dibuat agar kita tidak perlu lagi membuat hybrid page secara manual satu per satu lewat script ad-hoc atau patch langsung ke dataset.

Command utama:

```bash
pnpm --filter frontend run hybrid:create -- --slug=<slug> --preset=<preset>
```

Default command di atas adalah `dry-run`, jadi aman untuk inspeksi dulu sebelum menulis data.

Jika Anda juga ingin scaffold route dan component file di repo, gunakan:

```bash
pnpm --filter frontend run hybrid:route:create -- --slug=<slug> --preset=<preset>
```

Command ini juga `dry-run` secara default.

## Prinsip Dasar

Hybrid page di repo ini mengikuti aturan:

- shell utama tetap di code
- Sanity hanya menambah block support di atas dan bawah shell
- satu document memakai satu `blocks[]`
- `topBlockCount` menentukan berapa block pertama yang dirender sebelum shell

Contoh:

- `topBlockCount = 2`
- block ke-1 dan ke-2 tampil di atas shell
- sisanya tampil di bawah shell

## Guardrails

CLI ini mengikuti guardrail Sanity yang sudah dipakai di repo:

- write auth default dev-first:
  - `SANITY_DEV`
  - fallback `SANITY_AUTH_TOKEN`
- document publik memakai `_id` tanpa titik
- array items wajib punya `_key`
- object `link` wajib punya `isExternal`
- write akan diverifikasi dengan public-read tanpa token setelah commit

Lihat juga:

- [sanity-seed-guardrails.md](/home/ubuntu/next-js-sanity-starter/docs/sanity-seed-guardrails.md)
- [AGENTS.md](/home/ubuntu/next-js-sanity-starter/AGENTS.md)

## Preset yang Tersedia

Saat ini CLI mendukung:

- `main-landing`
- `homepage`

### `main-landing`

Cocok untuk page utama cluster seperti:

- `/layanan`
- `/pembuatan-website`
- `/percetakan`
- `/software`

Seed default:

- `topBlockCount: 2`
- `section-header`
- `grid-row`
- `cta-1`

### `homepage`

Cocok untuk `/` atau slug `index`.

Seed default:

- `topBlockCount: 2`
- `section-header`
- `grid-row`
- `cta-1`

Kontennya diprefill agar selaras dengan pola homepage hybrid yang sekarang dipakai repo.

## Mode

### 1. `seed-missing`

Mode default. Ini mode paling aman untuk page live.

Perilaku:

- jika document belum ada: dibuat
- jika document sudah ada:
  - `title` hanya diisi jika kosong
  - `topBlockCount` hanya diisi jika kosong
  - `blocks` hanya diisi jika kosong

Gunakan mode ini untuk:

- page live yang sudah pernah disentuh editor
- rollout awal hybrid tanpa risiko menimpa isi existing

Contoh:

```bash
pnpm --filter frontend run hybrid:create -- --slug=layanan --preset=main-landing --mode=seed-missing --write
```

### 2. `upsert`

Mode ini mengganti field hybrid utama dengan isi preset.

Perilaku:

- jika document belum ada: dibuat
- jika document sudah ada:
  - `title` ditimpa sesuai preset atau `--title`
  - `slug` diset ulang
  - `topBlockCount` ditimpa
  - `blocks` ditimpa dengan preset baru

Gunakan mode ini untuk:

- reset cepat
- rebuild hybrid seed secara disengaja
- document yang memang ingin diselaraskan ulang

Contoh:

```bash
pnpm --filter frontend run hybrid:create -- --slug=percetakan --preset=main-landing --mode=upsert --write
```

### 3. `create`

Mode ini hanya membuat document baru.

Perilaku:

- gagal jika slug sudah ada

Gunakan mode ini untuk:

- page baru
- rollout yang harus memastikan tidak ada overwrite sama sekali

Contoh:

```bash
pnpm --filter frontend run hybrid:create -- --slug=software --preset=main-landing --mode=create --write
```

## Dry Run vs Write

### Dry run

Untuk melihat hasil tanpa menulis:

```bash
pnpm --filter frontend run hybrid:create -- --slug=layanan --preset=main-landing
```

Output akan menampilkan:

- apakah slug termasuk hybrid route yang dikenal
- apakah document sudah ada
- `topBlockCount`
- jumlah block hasil
- document target yang akan ditulis

### Write

Untuk benar-benar commit ke Sanity:

```bash
pnpm --filter frontend run hybrid:create -- --slug=layanan --preset=main-landing --write
```

Setelah write, script akan:

- commit ke dataset
- query ulang via public-read
- memastikan slug, `topBlockCount`, dan block count terbaca publik

## Slug Whitelist

Secara default, `hybrid:create` hanya mengizinkan slug yang memang sudah diketahui memakai pola hybrid route di frontend:

- `index`
- `layanan`
- `pembuatan-website`
- `percetakan`
- `software`

Kalau slug belum masuk daftar supported hybrid slug, command akan gagal.

Ini disengaja agar kita tidak seed document yang tidak pernah dibaca route frontend.

Kalau memang perlu bypass:

```bash
pnpm --filter frontend run hybrid:create -- --slug=some-page --preset=main-landing --allow-unwired
```

Gunakan `--allow-unwired` hanya jika:

- route memang akan segera di-wire ke `PageHybridShell`
- atau Anda sedang menyiapkan seed sebelum wiring code

## Contoh Workflow yang Direkomendasikan

### Rollout aman untuk page live

1. Jalankan dry run:

```bash
pnpm --filter frontend run hybrid:create -- --slug=percetakan --preset=main-landing
```

2. Review output
3. Commit dengan mode aman:

```bash
pnpm --filter frontend run hybrid:create -- --slug=percetakan --preset=main-landing --mode=seed-missing --write
```

4. Buka Sanity Studio dan cek:
   - subtitle list `Hybrid · Top N`
   - isi `blocks[]`
   - nilai `topBlockCount`
5. Cek page di frontend

### Reset cepat preset hybrid

Jika ingin menimpa ulang seed hybrid:

```bash
pnpm --filter frontend run hybrid:create -- --slug=layanan --preset=main-landing --mode=upsert --write
```

Gunakan dengan hati-hati karena ini akan mengganti `blocks`.

## Cara Menambah Preset Baru

Preset hidup di:

- [hybrid-page-presets.mjs](/home/ubuntu/next-js-sanity-starter/frontend/scripts/lib/hybrid-page-presets.mjs)

Langkah umum:

1. Tambahkan builder preset baru
2. Daftarkan ke `HYBRID_PRESET_BUILDERS`
3. Pastikan semua block:
   - punya `_key`
   - link punya `isExternal`
4. Jalankan dry run
5. Jalankan write ke slug uji
6. Verifikasi public-read

Preset baru sebaiknya tidak terlalu besar. Mulai dari:

- `section-header`
- `grid-row`
- `cta-1`

dan biarkan editor refine sisanya di Studio.

## Kapan Tidak Menggunakan CLI Ini

Jangan gunakan CLI ini untuk:

- city pages massal
- route yang belum punya shell hybrid di frontend
- page yang memang ingin dijadikan full Sanity
- import konten besar dari Astro/HTML/MDX

CLI ini fokus untuk:

- hybrid main pages
- seed cepat
- rollout aman

## Kapan Memakai `hybrid:route:create`

Gunakan `hybrid:route:create` jika:

- slug belum punya route hybrid di repo
- Anda ingin generator membuat file route otomatis
- Anda ingin slug otomatis didaftarkan ke registry hybrid
- Anda ingin page Sanity sekaligus di-seed

Yang dilakukan command ini saat `--write`:

1. membuat `frontend/app/(main)/<slug>/page.tsx`
2. membuat `frontend/components/hybrid/generated/<slug>-middle-section.tsx`
3. menambahkan slug ke registry hybrid Studio dan script
4. menjalankan seed Sanity lewat `hybrid:create`

Guardrail anti-duplicate:

- command akan gagal jika route file target sudah ada
- command akan gagal jika middle-section component target sudah ada
- jadi tidak akan menimpa route hybrid yang sudah pernah dibuat

Contoh dry run:

```bash
pnpm --filter frontend run hybrid:route:create -- --slug=ubah-ke-hybrid --preset=main-landing
```

Contoh write:

```bash
pnpm --filter frontend run hybrid:route:create -- --slug=ubah-ke-hybrid --preset=main-landing --write
```

Jika slug sudah punya route hybrid dan Anda hanya ingin seed/update Sanity:

- gunakan `hybrid:create`
- jangan gunakan `hybrid:route:create`

## Studio Action: `Apply Hybrid Preset`

Selain CLI, sekarang editor juga bisa menerapkan preset hybrid langsung dari Sanity Studio.

Action ini muncul pada document `page` yang memang termasuk supported hybrid slug:

- `index`
- `layanan`
- `pembuatan-website`
- `percetakan`
- `software`

Nama action:

- `Apply Hybrid Preset`

Perilakunya:

1. action hanya muncul pada page hybrid yang memang sudah didukung route frontend
2. editor memilih preset:
   - `main-landing`
   - `homepage`
3. editor memilih mode:
   - `seed-missing`
   - `upsert`
4. action menulis ke draft document
5. editor review hasilnya lalu publish jika sudah sesuai

Catatan penting:

- action ini tidak publish otomatis
- action ini menulis ke draft agar aman untuk review
- `seed-missing` tetap mode paling aman untuk page live
- `upsert` cocok hanya jika memang ingin mengganti seed hybrid secara sengaja

Kapan pakai Studio action:

- saat editor ingin bekerja langsung dari Studio
- saat slug sudah hybrid dan route frontend sudah siap
- saat tidak perlu membuat file route baru di repo

Kapan jangan pakai Studio action:

- jika route hybrid belum ada
- jika Anda butuh scaffold file route dan component
- jika Anda butuh bootstrap slug baru end-to-end

Untuk slug baru end-to-end:

- gunakan `hybrid:route:create`

Untuk slug existing yang hanya butuh seed/update CMS:

- gunakan `hybrid:create`
- atau gunakan action `Apply Hybrid Preset`

## Troubleshooting

### `slug is not in the current hybrid route whitelist`

Artinya slug belum dikenal sebagai route hybrid.

Solusi:

- wire route ke `PageHybridShell` dulu
- atau pakai `--allow-unwired` jika memang sengaja

### `Generated document failed page guards`

Artinya seed yang dibangun melanggar guardrail.

Biasanya karena:

- ada object array tanpa `_key`
- ada `link` tanpa `isExternal`

Periksa preset builder.

### Public read gagal setelah `--write`

Biasanya karena:

- `_id` tidak aman
- dataset/env tidak sinkron
- reference publik tidak terbaca

Pastikan document publik:

- tidak memakai `_id` bertitik
- memakai dataset yang benar
- tidak menyimpan reference rusak

## Smoke Test Skenario

Berikut skenario nyata yang sudah diuji untuk workflow ini:

### 1. Slug belum supported -> `hybrid:create` harus gagal

Command:

```bash
pnpm --filter frontend run hybrid:create -- --slug=unsupported-hybrid-probe --preset=main-landing
```

Hasil:

- gagal dengan pesan bahwa slug belum menjadi supported hybrid slug
- ini benar, karena route frontend untuk slug tersebut belum di-wire ke `PageHybridShell`

### 2. Slug existing yang sudah supported -> `hybrid:create` dry run sukses

Command:

```bash
pnpm --filter frontend run hybrid:create -- --slug=layanan --preset=main-landing
```

Hasil:

- script menemukan `page-layanan`
- menampilkan `topBlockCount` dan `blockCount`
- tidak menulis data karena dry run

### 3. Slug baru -> `hybrid:route:create` dry run sukses

Command:

```bash
pnpm --filter frontend run hybrid:route:create -- --slug=hybrid-e2e-smoke --preset=main-landing
```

Hasil:

- menampilkan target route file
- menampilkan target component file
- menampilkan registry yang akan diubah
- menampilkan command seed Sanity yang akan dipanggil

### 4. Slug baru -> `hybrid:route:create --write` sukses

Command:

```bash
pnpm --filter frontend run hybrid:route:create -- --slug=hybrid-e2e-smoke --preset=main-landing --write
```

Hasil:

- route file berhasil dibuat
- middle-section component berhasil dibuat
- slug berhasil didaftarkan ke registry hybrid
- document Sanity `page-hybrid-e2e-smoke` berhasil dibuat
- public-read berhasil mengembalikan slug, `topBlockCount`, dan `blockCount`

### 5. Slug yang sama dijalankan lagi -> duplicate guard harus gagal

Command:

```bash
pnpm --filter frontend run hybrid:route:create -- --slug=hybrid-e2e-smoke --preset=main-landing
```

Hasil:

- gagal dengan pesan duplicate route protection
- file yang sudah ada tidak ditimpa

### 6. Cleanup test slug

Setelah smoke test:

- route test dihapus kembali dari repo
- component test dihapus kembali dari repo
- slug test dihapus kembali dari registry hybrid
- document Sanity test dihapus kembali
- public-read diverifikasi mengembalikan `null`

Jadi smoke test ini membuktikan:

- guard unsupported slug bekerja
- dry run bekerja
- write end-to-end bekerja
- duplicate protection bekerja
- cleanup juga aman

## File yang Terkait

- [create-hybrid-page.mjs](/home/ubuntu/next-js-sanity-starter/frontend/scripts/create-hybrid-page.mjs)
- [create-hybrid-route.mjs](/home/ubuntu/next-js-sanity-starter/frontend/scripts/create-hybrid-route.mjs)
- [hybrid-page-presets.mjs](/home/ubuntu/next-js-sanity-starter/frontend/scripts/lib/hybrid-page-presets.mjs)
- [sanity-page-guards.mjs](/home/ubuntu/next-js-sanity-starter/frontend/scripts/lib/sanity-page-guards.mjs)
- [page-hybrid-shell.tsx](/home/ubuntu/next-js-sanity-starter/frontend/components/hybrid/page-hybrid-shell.tsx)
- [hybrid-content-page-workflow skill](/home/ubuntu/next-js-sanity-starter/skills/hybrid-content-page-workflow/SKILL.md)
