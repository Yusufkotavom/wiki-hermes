---
title: Sanity-clean follow-up warning cleanup
created: 2026-07-03
updated: 2026-07-03
type: plan
tags: [plan, sanity-clean, frontend, blocks, cleanup]
status: active
due: 2026-07-04
---

# Sanity-clean warning cleanup

Resolve build warnings from `pnpm --filter frontend run build` and align block renderers/schema/queries/generators.

## Context

Frontend build succeeds, but emits many warnings:
- Unknown/missing block renderers:
  - `benefits-block`
  - `faq-block`
  - `value-props-block`
  - `stats-hero-block`
  - `eeat-block`
  - `metrics-rail-block`
  - `micro-badges-block`
  - `process-faq-block`

These indicate schema/types exist, GROQ queries reference them, generators seed them, but React renderers are missing or not mapped.

## Known warning types
- `benefits-block`
- `faq-block`
- `value-props-block`
- `stats-hero-block`
- `eeat-block`
- `metrics-rail-block`
- `micro-badges-block`
- `process-faq-block`

## Classification
- Deprecated blocks per prior consolidation: `eeat-block`, `metrics-rail-block`, `micro-badges-block`, `process-faq-block`, `highlights-block`, `related-links-block`, `stats-hero-block`, `benefits-block`, `faq-block`
- Active-but-missing blocks: `value-props-block` is emitted by current generators, but no renderer exists in `frontend/components/blocks/`

## Cleanup execution
1. Add minimal renderer and componentMap entry for `value-props-block`
2. Remove active GROQ import/file references to deprecated `benefits-block`
3. Audit documented warning status after rebuild
4. Rebuild frontend + studio, verify warnings reduced/removed
5. Commit changes scoped to block cleanup

## Status after patch 1
- Added `frontend/components/blocks/seo/value-props-block.tsx`
- Mapped `value-props-block` in `frontend/components/blocks/index.tsx`
- Removed `benefitsBlockQuery` from:
  - `frontend/sanity/queries/shared/blocks.ts`
  - `frontend/sanity/queries/legacy-page.ts`
  - `frontend/sanity/queries/reusable-section.ts`
- Build result:
  - frontend: build succeeds
  - studio: build succeeds
- Remaining warnings source: generator/seed payloads and legacy fixture content reference deprecated block types; these are not coming from active live query projections for normal frontend routes.
