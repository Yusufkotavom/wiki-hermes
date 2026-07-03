# AI Writer Production Guide

Dokumen ini fokus ke implementasi production AI rewrite di project ini.

## Scope

Yang termasuk:
- Konfigurasi provider/model/prompt lewat SEO Dashboard (disimpan ke Sanity `aiWriterSettings`).
- Endpoint config + generate + rewrite apply.
- Trigger rewrite dari SEO Dashboard/API (bukan document action di Studio).

Yang tidak termasuk:
- Auto-publish.
- Workflow approval multi-user.
- Cost dashboard terpisah.

## Arsitektur

1. Konfigurasi disimpan di dokumen Sanity: `aiWriterSettings`.
2. Secret key disimpan terenkripsi lewat API backend.
3. Dashboard/API memanggil endpoint rewrite backend.
4. Backend generate konten lalu patch ke draft target.

## Endpoint Aktif

- `GET /api/ai/config/status`
- `POST /api/ai/config/save`
- `POST /api/ai/generate`
- `POST /api/ai/rewrite/apply`

Catatan auth:
- `config/*` dan `generate` memakai auth cookie dashboard SEO.
- `rewrite/apply` mengikuti mekanisme auth backend/dashboard aktif.

## Provider Mode

- `gateway` (rekomendasi production): Vercel AI Gateway.
- `direct-gemini`: rotasi key Gemini.
- `direct-groq`: rotasi key Groq.

## Environment Wajib

Frontend (`frontend/.env`):
- `SEO_SESSION_SECRET`
- `SANITY_AUTH_TOKEN`

Opsional:
- `VERCEL_OIDC_TOKEN` (gateway via OIDC).
- `AI_GATEWAY_API_KEY` (gateway static key fallback).
- `AI_WRITER_GEMINI_KEYS`, `AI_WRITER_GROQ_KEYS` (fallback env key pool).

## Setup Cepat (Gateway)

```bash
vercel link
vercel env pull .env.local
```

Di Studio `AI Writer Settings`:
1. `enabled = true`
2. `mode = gateway`
3. `defaultModel = openai/gpt-5.4` (atau model valid provider/model)
4. Isi `fallbackModels` dan `gatewayProviderOrder` jika perlu.
5. Isi prompt template (`globalSystem`, `postRewrite`, `serviceRewrite`, `projectRewrite`).

## Format Key Rotation

Untuk direct mode, isi key sebagai newline list:

```text
key-1
key-2
key-3
```

Backend akan mencoba key satu per satu sampai sukses.

## Contoh Save Config

```json
{
  "enabled": true,
  "mode": "gateway",
  "defaultModel": "openai/gpt-5.4",
  "fallbackModels": ["google/gemini-2.5-flash"],
  "gatewayProviderOrder": ["openai", "google"],
  "temperature": 0.4,
  "maxOutputTokens": 1400,
  "prompts": {
    "globalSystem": "Anda editor SEO bahasa Indonesia.",
    "postRewrite": "Rewrite artikel tanpa mengubah intent slug."
  }
}
```

## Rewrite Apply Flow

1. Jalankan rewrite dari SEO Dashboard/API.
2. Tambah instruksi opsional bila diperlukan.
3. Backend rewrite dan patch `title`, `excerpt`, `body` ke draft.
4. Review manual sebelum publish.

## Guardrail yang Sudah Aktif

- Validasi model format untuk gateway: `provider/model`.
- Guard panjang prompt pada generate endpoint.
- Rewrite apply hanya untuk `post/service/project`.

## Production Checklist

1. Secret env sudah terpasang (`SEO_SESSION_SECRET`, `SANITY_AUTH_TOKEN`).
2. Mode `gateway` aktif dan model valid.
3. Prompt template final sudah diisi.
4. Smoke test:
   - `/api/ai/config/status`
   - `/api/ai/generate` (3 doc type)
5. Team editorial confirm review-before-publish policy.
