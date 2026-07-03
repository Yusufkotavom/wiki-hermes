# Sanity Blocks & Page Creation Reference

Dokumen ini merupakan panduan lengkap mengenai struktur *page*, daftar *block* yang tersedia, struktur *payload JSON* untuk integrasi, serta setup AI Generator Shortcodes.

## 1. Aturan Membuat Page (Hybrid Route)

Halaman utama (seperti `/layanan`, `/percetakan`, `/software`, atau *homepage*) diatur menggunakan pola **Hybrid Page**. Artinya, sebagian halaman dikendalikan oleh kode statis di frontend (sebagai *shell*), dan sebagian dirender menggunakan *block* dari Sanity.

### Skema Dokumen `page`
Payload utama untuk membuat sebuah *page*:
- `title` (String): Judul halaman
- `slug` (Slug): Endpoint URL
- `topBlockCount` (Number): Jumlah block pertama (dari array `blocks`) yang akan dirender di atas *code-owned shell*. Sisa block akan dirender di bagian bawah.
- `blocks` (Array): Kumpulan *block* (komponen UI) yang membangun konten halaman.
- `seo` (Object): Pengaturan SEO spesifik (title, description, image).

**Workflow CLI (Pembuatan Otomatis):**
Jangan membuat *page* hybrid secara manual. Gunakan CLI:
```bash
pnpm --filter frontend run hybrid:create -- --slug=<slug> --preset=<preset>
```

---

## 2. Daftar Sanity Blocks

Terdapat **35+** jenis block yang dikelompokkan ke dalam beberapa kategori. Seluruh definisi lengkap (termasuk tipe TypeScript) dapat ditemukan di `frontend/sanity.types.ts` dan schema di `studio/schemas/blocks/`.

### Hero & Header
- `hero-1`, `hero-2`, `hero-vercel`, `stats-hero-block`
- `section-header` (Sering digunakan untuk pembuka seksi)
- `split-row`

### Content & Grid
- `grid-row` (Kumpulan *card* dalam kolom)
- `carousel-1`, `carousel-2`
- `timeline-row`
- `logo-cloud-1`
- `flexible-builder` (Builder mirip Elementor untuk layout kolom dinamis)

### CTA & Conversion
- `cta-1`, `whatsapp-cta`
- `form-newsletter`

### SEO & E-E-A-T
- `company-info`, `testimonials-block`, `pricing-block`, `faq-block`
- `faqs`, `process-faq-block`, `features-package-block`, `service-types-block`
- `problem-solution-block`, `value-props-block`, `eeat-block`, `metrics-rail-block`
- `highlights-block`, `reviews-block`, `quote-spotlight-block`, `micro-badges-block`
- `related-links-block`

---

## 3. Struktur Payload JSON (Block Guardrails)

Aturan mutlak untuk semua Payload Block:
1. Setiap objek di dalam array **wajib** memiliki properti `_key`.
2. Objek bertipe `link` **wajib** menyertakan `isExternal` (boolean).

### Contoh: `section-header`
```json
{
  "_key": "header-unik-1",
  "_type": "section-header",
  "colorVariant": "background",
  "sectionWidth": "default",
  "stackAlign": "left",
  "tagLine": "Teks kecil di atas judul",
  "title": "Judul Utama",
  "description": "Deskripsi paragraf"
}
```

### Contoh: `grid-row`
```json
{
  "_key": "grid-unik-1",
  "_type": "grid-row",
  "gridColumns": "grid-cols-3",
  "columns": [
    {
      "_key": "kolom-1",
      "_type": "grid-card",
      "title": "Fitur 1",
      "excerpt": "Penjelasan fitur",
      "link": {
        "_key": "link-1",
        "_type": "link",
        "title": "Baca lebih lanjut",
        "href": "/fitur",
        "isExternal": false
      }
    }
  ]
}
```

### Contoh: `cta-1` (Dengan Portable Text)
```json
{
  "_key": "cta-unik-1",
  "_type": "cta-1",
  "colorVariant": "primary",
  "title": "Mulai Proyek Anda",
  "body": [
    {
      "_key": "body-block-1",
      "_type": "block",
      "style": "normal",
      "markDefs": [],
      "children": [
        {
          "_key": "span-1",
          "_type": "span",
          "marks": [],
          "text": "Hubungi tim kami sekarang juga."
        }
      ]
    }
  ],
  "links": [
    {
      "_key": "link-cta",
      "_type": "link",
      "title": "WhatsApp Kami",
      "href": "https://wa.me/...",
      "isExternal": true,
      "buttonVariant": "default"
    }
  ]
}
```

---

## 4. AI Generator Shortcodes Setup

Jika Anda menggunakan skrip otomasi (Generator) ke database Sanity, Anda dapat menggunakan *AI Prompt Injection*.
- **Cara pakai**: Sisipkan string `[aigen:Buatkan paragraf pembuka...]` di properti string mana pun (misal `description`, atau isi `text` dalam Portable Text).
- Sistem akan mengeksekusi AI *di latar belakang* sebelum menyimpan ke draft Sanity.

**Persyaratan Environment (`frontend/.env.local`):**
Sistem AI mendukung *multi-key* dan *fallback*:
```env
# Primary (Gemini)
OPENAI_API_KEYS="AIzaSy...,AIzaSyB..."
OPENAI_BASE_URL="https://generativelanguage.googleapis.com/v1beta/openai/chat/completions"
OPENAI_MODEL="gemini-2.5-flash"

# Fallback
AI_FALLBACK_PROVIDERS="groq,openrouter"
GROQ_API_KEY="gsk_..."
# (Referensi detail lihat di docs/ai-generator-shortcodes.md)
```

Lihat selengkapnya di [hybrid-page-cli-workflow.md](./hybrid-page-cli-workflow.md) dan [ai-generator-shortcodes.md](./ai-generator-shortcodes.md).
