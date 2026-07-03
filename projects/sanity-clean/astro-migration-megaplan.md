# Astro Migration Megaplan

## Current Status Snapshot (Already Done)
- [x] Synced `features-package-block` query contract with Studio schema for `cardStyle`.
- [x] Synced `features-package-block` query contract with Studio schema for `cta`.
- [x] Confirmed renderer path `frontend/components/blocks/seo/features-package-block.tsx` already consumes both fields.
- [x] Fixed `grid-row` column class generation by replacing dynamic Tailwind interpolation with static mapping.
- [x] Extended `grid-row` alignment/style behavior to additional card types used in mixed rows.
- [x] Added missing query coverage for `eeat-block`, `metrics-rail-block`, `highlights-block`, `reviews-block`, and `micro-badges-block`.
- [x] Unified block projections across shared, reusable-section, and legacy-page query entry-points.
- [x] Fixed Studio runtime plugin registration for markdown schema (`markdownSchema()`), removing boot-time `is not a function` crash.
- [x] Improved homepage LCP image discoverability for Sanity `index` page by making hero blocks (`hero-2`, `hero-vercel`) statically imported and forcing eager/high-priority image fetch hints.
- [x] Unified `whatsapp-cta` block presentation with shared section design system (`SectionShell` + `SectionPanel`) for consistent CMS-rendered CTA styling.
- [x] Fixed metadata OG generator URL builder so dynamic `/api/og` uses active deployment domain (not stale `seo.siteUrl` fallback).
- [x] Added dedicated `ogSettings.ogBaseUrl` and wired metadata OG URL generation to use OG settings as the primary source.
- [x] Hardened frontend verification pipeline (`typecheck` and `test:templates`) to avoid false failures from stale/missing generated test artifacts.
- [x] Restored Sanity Live Editor (Presentation runtime) with Draft Mode-only rendering to keep public PageSpeed path lightweight.
- [x] Reduced global header JS by removing Radix dropdown dependency from desktop navigation and simplifying theme toggle interaction.
- [x] Fixed Studio OG generation action URL priority to use `ogSettings.ogBaseUrl` before env/CMS fallbacks.
- [x] Removed stale `devk.my.id` fallback defaults from SEO seed sources to prevent reintroducing wrong OG/canonical base URL.
- [x] Removed hardcoded `siteUrl` from SEO seed JSON and switched seed runtime to env-resolved site URL injection.
- [x] Removed hardcoded `/home-sanity` canonical fallback domain; script now requires env URL source.
- [x] Refactored `seoSettings` Studio form to a single navigation pattern with grouped tabs and non-collapsible major sections.
- [x] Enforced strict Studio OG URL source: only `ogSettings.ogBaseUrl` (no fallback).
- [x] Added CORS headers for `/api/og` proxy path so Studio-origin fetch can generate OG images.

## Workstream TODO
- [x] Fix mismatch that forced `Features / Value Props` layout to always render as grid.
- [x] Fix missing CTA payload for `Features / Value Props` block.
- [x] Run full frontend build/test verification after query sync.
- [x] Re-run frontend build after multi-query block coverage updates.
- [x] Re-run Studio build after markdown plugin registration fix.
- [x] Ensure homepage hero/LCP images are discoverable in initial HTML and not lazy-loaded.
- [x] Align WhatsApp CTA block visuals with existing CTA design language.
- [x] Normalize metadata site URL resolution across env vars (`NEXT_PUBLIC_SITE_URL`, `VERCEL_PROJECT_PRODUCTION_URL`, `VERCEL_URL`) and CMS fallback.
- [x] Synced OG settings schema/query/frontend metadata contract for centralized OG endpoint URL source.
- [x] Added deterministic template-test runner with explicit skip behavior when optional contract test file is absent.
- [x] Added env kill-switch `NEXT_PUBLIC_SANITY_VISUAL_EDITING` and token guard for safe live editor runtime activation.
- [x] Verified home route client manifest no longer includes `components/ui/dropdown-menu.tsx`.
- [x] Ensure Studio "Generate OG Image" action resolves `/api/og` from `ogSettings.ogBaseUrl` as primary source.
- [x] Replace stale `devk.my.id` hardcoded defaults in SEO seed scripts used for CMS/bootstrap flows.
- [x] Ensure SEO seed flow does not hardcode domain in repository JSON; inject `siteUrl` from runtime env.
- [x] Ensure `/home-sanity` seed canonical URL fails fast when site URL env is missing.
- [x] Ensure `/structure/seoSettings` authoring UX is consistent (no mixed top-level tab/accordion navigation).
- [x] Prevent Studio OG generation from using any fallback source; require `ogSettings.ogBaseUrl` only.
- [x] Ensure `/api/og` responds with CORS headers for allowed Studio origins (including preflight).

## Blockers
- None.
