# Panduan Membuat & Mengelola Halaman (Sanity Blocks)

Di proyek DevK Studio, setiap halaman menggunakan pola **Hybrid Route**. Artinya, `app/[slug]/page.tsx` di frontend hanya bertindak sebagai cangkang kosong yang dirender berdasarkan data JSON (blocks) dari Sanity.

## 1. Pendekatan CLI (Direkomendasikan untuk Pemula)

Jika Anda ingin membuat halaman baru dari terminal (misal halaman layanan spesifik), Anda dapat menggunakan skrip generator bawaan.

**Perintah:**
```bash
cd frontend
pnpm run hybrid:create -- --slug=nama-halaman --preset=service
```

Skrip ini akan otomatis:
1. Membuat *shell* komponen frontend React (`frontend/components/hybrid/generated/nama-halaman-middle-section.tsx`).
2. Mendaftarkan halaman tersebut di *routing* Next.js.
3. Menyuntikkan draft JSON dokumen ke Sanity (berisi Hero, Value Props, dll).

## 2. Pendekatan Script Seeding (Untuk Bulk Update / Migrasi)

Jika Anda ingin merakit *landing page* yang kompleks (seperti halaman *Home*, *About*, dll), lebih aman menulis skrip *seeding* menggunakan Node.js. Ini memastikan Anda punya "rekam jejak" (Source of Truth) di dalam repositori kode.

**Letak Skrip Seeding:**
Semua skrip otomasi berada di direktori `frontend/scripts/`.

**Contoh Skrip:**
- `seed-devk-index.mjs` (Digunakan untuk merakit `page-index` / Home)
- `seed-all-pages.mjs` (Digunakan untuk halaman standar seperti About, Contact, Services)

**Cara Menjalankan Skrip:**
```bash
cd frontend
node scripts/seed-devk-index.mjs --write
```
*(Catatan: Tanpa flag `--write`, skrip hanya melakukan DRY RUN / simulasi).*

## 3. Anatomi Landing Page Berkualitas Tinggi

Saat merakit *landing page* di dalam skrip (atau via Sanity Studio langsung), pastikan Anda menyusun blok-blok (*blocks*) dengan urutan psikologis konversi yang tepat:

1. **`hero-1` atau `hero-2`**: Hook utama (Value Proposition & CTA).
2. **`logo-cloud-1`**: Bukti kredibilitas sekilas (Trust Signal).
3. **`problem-solution-block`**: Agitasi masalah pelanggan dan bagaimana Anda menyelesaikannya.
4. **`value-props-block`**: 3 alasan mengapa mereka harus memilih Anda.
5. **`grid-row` / `service-types-block`**: Etalase produk/layanan yang Anda jual.
6. **`testimonials-block`**: Social proof mendalam (kata pelanggan).
7. **`faq-block`**: Menjawab keraguan (Bantahan Masalah).
8. **`cta-1`**: Dorongan aksi terakhir sebelum mencapai *footer*.

## 4. Mengunggah dan Menyisipkan Gambar via Skrip

Karena skrip *seeding* (`seed-*.mjs`) biasanya hanya memuat teks, Anda memerlukan skrip terpisah untuk mengunggah aset gambar (JPG/PNG) ke Sanity lalu menyambungkannya ke blok yang tepat.

**Skrip yang digunakan:**
`frontend/scripts/upload-images.mjs`

Skrip ini menggunakan `@sanity/client` untuk membaca file gambar dari direktori lokal (`fs.createReadStream`), mengunggahnya sebagai aset, lalu menggunakan `.patch(doc._id)` untuk menginjeksi ID aset gambar ke komponen `hero-1` di Sanity.
