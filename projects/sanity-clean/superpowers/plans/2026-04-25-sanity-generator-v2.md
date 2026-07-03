# Sanity Generator V2 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a new dev-only page generator inside Sanity Studio that accepts bulk keyword sets plus variation rows, previews one output, and generates standard `page` documents without touching production runtime.

**Architecture:** The generator lives entirely in Sanity Studio as a new schema/tooling surface. Generation is deterministic first: `generatorTemplate + generatorDataset + generatorProgram -> draft page document`. Legacy `pageTemplate` and runtime resolver remain frozen during the build; the frontend continues rendering only standard `page` documents.

**Tech Stack:** Sanity Studio schema/doc views/actions, Next.js frontend, GROQ queries, repo-local Sanity scripts, TypeScript, pnpm workspace tooling.

---

## File Structure Map

### Studio: New Generator Surface

- Create: `studio/schemas/documents/generator-template.ts`
  - Minimal template schema for section skeleton, token definitions, and variation rules.
- Create: `studio/schemas/documents/generator-program.ts`
  - Operator-facing program document, equivalent to a content group.
- Create: `studio/schemas/documents/generator-dataset.ts`
  - Keyword sets, rows, import mode, and dedupe policy.
- Create: `studio/schemas/objects/generator-token-definition.ts`
  - Token contract for templates.
- Create: `studio/schemas/objects/generator-keyword-set.ts`
  - Bulk keyword entries.
- Create: `studio/schemas/objects/generator-row.ts`
  - Row-level service/city/industry/offer metadata.
- Create: `studio/schemas/objects/generator-section-variant.ts`
  - Optional/base section definitions and variation controls.
- Create: `studio/schemas/objects/generator-page-meta.ts`
  - Namespaced generator metadata attached to final `page`.
- Modify: `studio/schema-types.ts`
  - Register new documents and objects.
- Modify: `studio/structure.ts`
  - Add a top-level `Generator` group.
- Modify: `studio/defaultDocumentNode.ts`
  - Add a custom Studio view for `generatorProgram`.
- Modify: `studio/schemas/documents/page.ts`
  - Add `generator` metadata object to standard pages.

### Studio: Generator Logic and Views

- Create: `studio/lib/generator/types.ts`
  - Shared generator TypeScript types.
- Create: `studio/lib/generator/slug.ts`
  - Route-safe slug construction and uniqueness helpers.
- Create: `studio/lib/generator/variation.ts`
  - Angle/section/copy variation selection logic.
- Create: `studio/lib/generator/render.ts`
  - Deterministic page assembly from template + keyword + row.
- Create: `studio/lib/generator/dedupe.ts`
  - Existing-page detection and duplicate guardrails.
- Create: `studio/lib/generator/legacy.ts`
  - Read-only mapper for mining old `pageTemplate` content into the new structure.
- Create: `studio/components/generator/program-runner-pane.tsx`
  - Main four-panel Studio UI: setup, inputs, preview, run.
- Create: `studio/components/generator/preview-card.tsx`
  - One-row preview rendering and validation summary.
- Create: `studio/components/generator/run-summary.tsx`
  - Generated/skipped/conflicted/failed result reporting.

### Frontend: Minimal Contract Adjustments

- Modify: `frontend/sanity/queries/page.ts`
  - Optionally fetch `generator` metadata for debug/verification.
- Modify: `frontend/sanity/lib/fetch.ts`
  - Add explicit dev-only fetch helper if generator QA page needs it.
- Create: `frontend/app/(main)/generator-preview/[slug]/page.tsx`
  - Optional noindex internal route for dev preview if Studio iframe needs a predictable target.

### Scripts and Migration Support

- Create: `frontend/scripts/generator/export-legacy-templates.mjs`
  - Read old `pageTemplate` docs and export a migration inventory.
- Create: `frontend/scripts/generator/seed-generator-service-starters.mjs`
  - Seed clean starter template/program/dataset families in the development dataset only.
- Create: `frontend/scripts/generator/run-generator-smoke.mjs`
  - Dry-run generator pipeline against development dataset.

### Tracking Docs

- Modify: `docs/seo-updates.md`
  - Required update log for every repo change.
- Modify: `docs/astro-migration-megaplan.md`
  - Track generator rollout and legacy templating freeze.

## Implementation Notes Before Starting

- The working tree currently contains unrelated cleanup changes from the earlier component/studio prune. Do not stage them while implementing generator work.
- Every content-writing path must prefer development credentials:
  - `SANITY_DEV`
  - fallback `SANITY_AUTH_TOKEN`
- No production dataset writes are allowed during this plan.
- Generated page `_id` values must avoid dots and use stable dev-only prefixes.

## Task 1: Establish Dev-Only Generator Schemas

**Files:**
- Create: `studio/schemas/documents/generator-template.ts`
- Create: `studio/schemas/documents/generator-program.ts`
- Create: `studio/schemas/documents/generator-dataset.ts`
- Create: `studio/schemas/objects/generator-token-definition.ts`
- Create: `studio/schemas/objects/generator-keyword-set.ts`
- Create: `studio/schemas/objects/generator-row.ts`
- Create: `studio/schemas/objects/generator-section-variant.ts`
- Create: `studio/schemas/objects/generator-page-meta.ts`
- Modify: `studio/schema-types.ts`
- Modify: `studio/schemas/documents/page.ts`

- [ ] **Step 1: Write the failing schema registration check**

Create a targeted smoke script at `studio/scripts/check-generator-schema.mjs`:

```js
import { schemaTypes } from "../schema-types.js";

const required = [
  "generatorTemplate",
  "generatorProgram",
  "generatorDataset",
  "generatorTokenDefinition",
  "generatorKeywordSet",
  "generatorRow",
  "generatorSectionVariant",
  "generatorPageMeta",
];

const names = new Set(schemaTypes.map((item) => item?.name).filter(Boolean));
const missing = required.filter((name) => !names.has(name));

if (missing.length > 0) {
  console.error("Missing generator schema types:", missing.join(", "));
  process.exit(1);
}

console.log("Generator schema types registered");
```

- [ ] **Step 2: Run the schema check to verify it fails**

Run: `node studio/scripts/check-generator-schema.mjs`

Expected: FAIL with one or more missing schema names.

- [ ] **Step 3: Add minimal generator object schemas**

Start with small objects instead of one monolith. Example `studio/schemas/objects/generator-keyword-set.ts`:

```ts
import { defineField, defineType } from "sanity";

export default defineType({
  name: "generatorKeywordSet",
  title: "Generator Keyword Set",
  type: "object",
  fields: [
    defineField({ name: "label", type: "string", validation: (Rule) => Rule.required() }),
    defineField({ name: "primaryKeyword", type: "string", validation: (Rule) => Rule.required() }),
    defineField({ name: "secondaryKeywords", type: "array", of: [{ type: "string" }] }),
    defineField({ name: "angle", type: "string" }),
  ],
});
```

Example `studio/schemas/objects/generator-page-meta.ts`:

```ts
import { defineField, defineType } from "sanity";

export default defineType({
  name: "generatorPageMeta",
  title: "Generator Page Meta",
  type: "object",
  fields: [
    defineField({ name: "programId", type: "string" }),
    defineField({ name: "templateId", type: "string" }),
    defineField({ name: "rowKey", type: "string" }),
    defineField({ name: "keywordKey", type: "string" }),
    defineField({ name: "generatedAt", type: "datetime" }),
    defineField({ name: "version", type: "string" }),
    defineField({ name: "aiUsed", type: "boolean", initialValue: false }),
  ],
});
```

- [ ] **Step 4: Add minimal document schemas**

Use the smallest schema that still supports the spec. Example `studio/schemas/documents/generator-program.ts`:

```ts
import { defineField, defineType } from "sanity";

export default defineType({
  name: "generatorProgram",
  title: "Generator Program",
  type: "document",
  fields: [
    defineField({ name: "title", type: "string", validation: (Rule) => Rule.required() }),
    defineField({
      name: "slug",
      type: "slug",
      options: { source: "title", maxLength: 96 },
      validation: (Rule) => Rule.required(),
    }),
    defineField({
      name: "template",
      type: "reference",
      to: [{ type: "generatorTemplate" }],
      validation: (Rule) => Rule.required(),
    }),
    defineField({
      name: "dataset",
      type: "reference",
      to: [{ type: "generatorDataset" }],
      validation: (Rule) => Rule.required(),
    }),
    defineField({ name: "routeBase", type: "string", validation: (Rule) => Rule.required() }),
    defineField({
      name: "status",
      type: "string",
      options: { list: ["draft", "ready", "paused"] },
      initialValue: "draft",
    }),
    defineField({
      name: "aiMode",
      type: "string",
      options: { list: ["off", "prepared"] },
      initialValue: "off",
    }),
  ],
});
```

- [ ] **Step 5: Register types and extend `page`**

Add imports to `studio/schema-types.ts` and attach metadata to `studio/schemas/documents/page.ts`:

```ts
defineField({
  name: "generator",
  title: "Generator Metadata",
  type: "generatorPageMeta",
  group: "settings",
});
```

- [ ] **Step 6: Run schema checks and Studio typecheck**

Run:
- `node studio/scripts/check-generator-schema.mjs`
- `pnpm --filter studio run typecheck`

Expected:
- `Generator schema types registered`
- TypeScript passes for `studio`

- [ ] **Step 7: Commit**

```bash
git add studio/schemas/documents/generator-template.ts \
  studio/schemas/documents/generator-program.ts \
  studio/schemas/documents/generator-dataset.ts \
  studio/schemas/objects/generator-token-definition.ts \
  studio/schemas/objects/generator-keyword-set.ts \
  studio/schemas/objects/generator-row.ts \
  studio/schemas/objects/generator-section-variant.ts \
  studio/schemas/objects/generator-page-meta.ts \
  studio/schema-types.ts \
  studio/schemas/documents/page.ts \
  studio/scripts/check-generator-schema.mjs
git commit -m "feat: add generator v2 schema scaffolding"
```

## Task 2: Add Generator Desk Structure and Program View

**Files:**
- Modify: `studio/structure.ts`
- Modify: `studio/defaultDocumentNode.ts`
- Create: `studio/components/generator/program-runner-pane.tsx`
- Create: `studio/components/generator/preview-card.tsx`
- Create: `studio/components/generator/run-summary.tsx`

- [ ] **Step 1: Write the failing desk visibility check**

Add `studio/scripts/check-generator-structure.mjs`:

```js
import fs from "node:fs";

const structure = fs.readFileSync(new URL("../structure.ts", import.meta.url), "utf8");

for (const needle of ["Generator", "generatorProgram", "generatorTemplate", "generatorDataset"]) {
  if (!structure.includes(needle)) {
    console.error(`Missing structure entry: ${needle}`);
    process.exit(1);
  }
}

console.log("Generator structure entries present");
```

- [ ] **Step 2: Run the structure check to verify it fails**

Run: `node studio/scripts/check-generator-structure.mjs`

Expected: FAIL because the Generator group is not present yet.

- [ ] **Step 3: Add a dedicated Generator section in the Studio desk**

Add a grouped desk section in `studio/structure.ts`:

```ts
S.listItem()
  .title("Generator")
  .icon(LayoutTemplate)
  .child(
    S.list()
      .title("Generator")
      .items([
        orderableDocumentListDeskItem({
          type: "generatorProgram",
          title: "Programs",
          icon: LayoutTemplate,
          S,
          context,
        }),
        orderableDocumentListDeskItem({
          type: "generatorTemplate",
          title: "Templates",
          icon: Blocks,
          S,
          context,
        }),
        orderableDocumentListDeskItem({
          type: "generatorDataset",
          title: "Datasets",
          icon: Files,
          S,
          context,
        }),
      ])
  )
```

- [ ] **Step 4: Add a custom document view for `generatorProgram`**

Extend `studio/defaultDocumentNode.ts`:

```ts
import { ProgramRunnerPane } from "./components/generator/program-runner-pane";

if (schemaType === "generatorProgram") {
  return S.document().views([
    S.view.form(),
    S.view.component(ProgramRunnerPane).title("Generator Run"),
  ]);
}
```

- [ ] **Step 5: Implement the first minimal pane**

Create `studio/components/generator/program-runner-pane.tsx` with four sections and placeholder live data:

```tsx
export function ProgramRunnerPane(props: { documentId: string }) {
  return (
    <div style={{ padding: 24 }}>
      <h2>Generator Run</h2>
      <section><h3>Program Setup</h3></section>
      <section><h3>Inputs</h3></section>
      <section><h3>Preview</h3></section>
      <section><h3>Run</h3></section>
    </div>
  );
}
```

- [ ] **Step 6: Re-run checks**

Run:
- `node studio/scripts/check-generator-structure.mjs`
- `pnpm --filter studio run typecheck`

Expected:
- `Generator structure entries present`
- Studio typecheck passes

- [ ] **Step 7: Commit**

```bash
git add studio/structure.ts \
  studio/defaultDocumentNode.ts \
  studio/components/generator/program-runner-pane.tsx \
  studio/components/generator/preview-card.tsx \
  studio/components/generator/run-summary.tsx \
  studio/scripts/check-generator-structure.mjs
git commit -m "feat: add generator v2 studio navigation and program view"
```

## Task 3: Build Deterministic Generator Core

**Files:**
- Create: `studio/lib/generator/types.ts`
- Create: `studio/lib/generator/slug.ts`
- Create: `studio/lib/generator/variation.ts`
- Create: `studio/lib/generator/render.ts`
- Create: `studio/lib/generator/dedupe.ts`
- Test: `studio/lib/generator/__tests__/render.test.ts`

- [ ] **Step 1: Write the failing generator core tests**

Create `studio/lib/generator/__tests__/render.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { buildGeneratedPageDraft } from "../render";

describe("buildGeneratedPageDraft", () => {
  it("builds a page draft with generator metadata and slug", () => {
    const result = buildGeneratedPageDraft({
      program: { _id: "gp-1", routeBase: "/percetakan" },
      template: { _id: "gt-1", title: "Printing", designFamily: "printing" },
      keywordSet: { _key: "kw-1", primaryKeyword: "jasa cetak buku" },
      row: { _key: "row-1", service: "cetak-buku", city: "surabaya" },
    });

    expect(result._type).toBe("page");
    expect(result.slug.current).toContain("surabaya");
    expect(result.generator.programId).toBe("gp-1");
  });

  it("changes section plan when angle changes", () => {
    const fast = buildGeneratedPageDraft({
      program: { _id: "gp-1", routeBase: "/percetakan" },
      template: { _id: "gt-1", title: "Printing", designFamily: "printing" },
      keywordSet: { _key: "kw-fast", primaryKeyword: "cetak buku cepat", angle: "speed" },
      row: { _key: "row-1", service: "cetak-buku", city: "surabaya" },
    });

    const price = buildGeneratedPageDraft({
      program: { _id: "gp-1", routeBase: "/percetakan" },
      template: { _id: "gt-1", title: "Printing", designFamily: "printing" },
      keywordSet: { _key: "kw-price", primaryKeyword: "cetak buku murah", angle: "price" },
      row: { _key: "row-1", service: "cetak-buku", city: "surabaya" },
    });

    expect(fast.blocks).not.toEqual(price.blocks);
  });
});
```

- [ ] **Step 2: Run the test to verify it fails**

Run: `pnpm --filter studio exec vitest run studio/lib/generator/__tests__/render.test.ts`

Expected: FAIL because the generator core does not exist yet.

- [ ] **Step 3: Define shared types**

Create `studio/lib/generator/types.ts`:

```ts
export type GeneratorKeywordSet = {
  _key: string;
  primaryKeyword: string;
  secondaryKeywords?: string[];
  angle?: string;
};

export type GeneratorRow = {
  _key: string;
  service?: string;
  city?: string;
  industry?: string;
  ctaLabel?: string;
};

export type GeneratorProgramLite = {
  _id: string;
  routeBase: string;
};

export type GeneratorTemplateLite = {
  _id: string;
  title: string;
  designFamily: string;
};
```

- [ ] **Step 4: Implement slug, variation, and render helpers**

Example `studio/lib/generator/slug.ts`:

```ts
export const buildGeneratorSlug = ({
  routeBase,
  service,
  city,
  primaryKeyword,
}: {
  routeBase: string;
  service?: string;
  city?: string;
  primaryKeyword: string;
}) => {
  const raw = [routeBase, service, city || primaryKeyword]
    .filter(Boolean)
    .join(" ")
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/^-+|-+$/g, "");

  return raw.slice(0, 96);
};
```

Example `studio/lib/generator/render.ts`:

```ts
import { buildGeneratorSlug } from "./slug";

export const buildGeneratedPageDraft = ({ program, template, keywordSet, row }: any) => {
  const slug = buildGeneratorSlug({
    routeBase: program.routeBase,
    service: row.service,
    city: row.city,
    primaryKeyword: keywordSet.primaryKeyword,
  });

  return {
    _type: "page",
    title: `${keywordSet.primaryKeyword} ${row.city || ""}`.trim(),
    slug: { current: slug },
    topBlockCount: 0,
    blocks: [
      {
        _type: "hero-1",
        _key: "hero-1",
        title: `${keywordSet.primaryKeyword} untuk ${row.city || row.service || "target utama"}`,
      },
    ],
    generator: {
      programId: program._id,
      templateId: template._id,
      rowKey: row._key,
      keywordKey: keywordSet._key,
      generatedAt: new Date().toISOString(),
      version: "v2",
      aiUsed: false,
    },
  };
};
```

- [ ] **Step 5: Add duplicate detection**

Create `studio/lib/generator/dedupe.ts`:

```ts
export const detectDuplicateSlug = (
  existing: Array<{ slug?: { current?: string } | null }>,
  nextSlug: string,
) => existing.some((item) => item?.slug?.current === nextSlug);
```

- [ ] **Step 6: Run tests and typecheck**

Run:
- `pnpm --filter studio exec vitest run studio/lib/generator/__tests__/render.test.ts`
- `pnpm --filter studio run typecheck`

Expected:
- render tests pass
- studio typecheck passes

- [ ] **Step 7: Commit**

```bash
git add studio/lib/generator/types.ts \
  studio/lib/generator/slug.ts \
  studio/lib/generator/variation.ts \
  studio/lib/generator/render.ts \
  studio/lib/generator/dedupe.ts \
  studio/lib/generator/__tests__/render.test.ts
git commit -m "feat: add deterministic generator v2 core"
```

## Task 4: Connect Program View to Preview and Batch Run

**Files:**
- Modify: `studio/components/generator/program-runner-pane.tsx`
- Modify: `studio/components/generator/preview-card.tsx`
- Modify: `studio/components/generator/run-summary.tsx`
- Create: `frontend/scripts/generator/run-generator-smoke.mjs`

- [ ] **Step 1: Write the failing smoke-run script**

Create `frontend/scripts/generator/run-generator-smoke.mjs`:

```js
console.error("Generator smoke script not implemented");
process.exit(1);
```

- [ ] **Step 2: Verify the smoke script fails**

Run: `node frontend/scripts/generator/run-generator-smoke.mjs`

Expected: FAIL with the placeholder error.

- [ ] **Step 3: Add preview data loading in the pane**

Update `studio/components/generator/program-runner-pane.tsx` to:
- load the current `generatorProgram`
- resolve linked `generatorTemplate` and `generatorDataset`
- select the first `keywordSet` and first `row`
- call `buildGeneratedPageDraft`
- pass result to `PreviewCard`

Use a narrow loader shape:

```ts
const previewInput = {
  program,
  template,
  keywordSet: dataset.keywordSets?.[0],
  row: dataset.rows?.[0],
};
```

- [ ] **Step 4: Add batch-run behavior with dry-run first**

The run button should first use dry-run output:

```ts
const dryRun = combinations.map(({ keywordSet, row }) =>
  buildGeneratedPageDraft({ program, template, keywordSet, row }),
);
```

Then display counts:

```ts
{
  generated: 0,
  skipped: 0,
  conflicted: dryRun.filter((item) => duplicateCheck(item)).length,
  failed: 0,
}
```

- [ ] **Step 5: Upgrade the smoke script to real dry-run logic**

Replace the placeholder script with:

```js
import { buildGeneratedPageDraft } from "../../../studio/lib/generator/render.js";

const sample = buildGeneratedPageDraft({
  program: { _id: "gp-dev", routeBase: "/percetakan" },
  template: { _id: "gt-dev", title: "Printing", designFamily: "printing" },
  keywordSet: { _key: "kw-1", primaryKeyword: "jasa cetak buku", angle: "quality" },
  row: { _key: "row-1", service: "cetak-buku", city: "surabaya" },
});

console.log(JSON.stringify({ ok: true, slug: sample.slug.current, title: sample.title }, null, 2));
```

- [ ] **Step 6: Run smoke + Studio typecheck**

Run:
- `node frontend/scripts/generator/run-generator-smoke.mjs`
- `pnpm --filter studio run typecheck`

Expected:
- JSON output with `{ "ok": true, ... }`
- Studio typecheck passes

- [ ] **Step 7: Commit**

```bash
git add studio/components/generator/program-runner-pane.tsx \
  studio/components/generator/preview-card.tsx \
  studio/components/generator/run-summary.tsx \
  frontend/scripts/generator/run-generator-smoke.mjs
git commit -m "feat: connect generator v2 preview and dry-run workflow"
```

## Task 5: Add Safe Write Path to Development Dataset Only

**Files:**
- Create: `studio/lib/generator/write.ts`
- Create: `frontend/scripts/generator/seed-generator-service-starters.mjs`
- Modify: `studio/components/generator/program-runner-pane.tsx`

- [ ] **Step 1: Write a failing dev-write guard test script**

Create `frontend/scripts/generator/check-dev-write-guard.mjs`:

```js
if (!process.env.SANITY_DEV && !process.env.SANITY_AUTH_TOKEN) {
  console.error("Missing development write credential");
  process.exit(1);
}

if ((process.env.SANITY_STUDIO_DATASET || "").toLowerCase() === "production") {
  console.error("Refusing to target production dataset");
  process.exit(1);
}

console.log("Generator write guard passed");
```

- [ ] **Step 2: Run the guard to verify current env expectations**

Run: `node frontend/scripts/generator/check-dev-write-guard.mjs`

Expected:
- PASS in the dev environment
- or explicit fail if the env is unsafe

- [ ] **Step 3: Implement write helper**

Create `studio/lib/generator/write.ts`:

```ts
export const assertGeneratorWriteTarget = (dataset: string) => {
  if (dataset === "production") {
    throw new Error("Generator V2 must not write to production");
  }
};

export const buildGeneratedPageId = (slug: string) => `generator-page-${slug}`;
```

- [ ] **Step 4: Add starter seed script**

Create `frontend/scripts/generator/seed-generator-service-starters.mjs` to upsert clean starter families:
- website
- software
- printing

Use ids like:

```js
const docs = [
  { _id: "generator-template-website-starter-dev", _type: "generatorTemplate", title: "Website Service Starter" },
  { _id: "generator-dataset-website-starter-dev", _type: "generatorDataset", title: "Website Service Dataset" },
  { _id: "generator-program-website-starter-dev", _type: "generatorProgram", title: "Website Service Starter Program" },
];
```

- [ ] **Step 5: Hook the real write action into the pane**

The `Generate Drafts` button should:
- validate env/dataset
- dry-run first
- upsert missing `page` drafts with `generator-page-<slug>` ids
- skip existing conflicting slugs

Expected write result shape:

```ts
{
  generated: ["generator-page-jasa-cetak-buku-surabaya"],
  skipped: [],
  conflicted: ["generator-page-jasa-cetak-buku-jakarta"],
  failed: [],
}
```

- [ ] **Step 6: Run write-path verification**

Run:
- `node frontend/scripts/generator/check-dev-write-guard.mjs`
- `pnpm --filter studio run typecheck`
- `pnpm --filter frontend run typecheck`

Expected:
- dev guard passes
- both typechecks pass

- [ ] **Step 7: Commit**

```bash
git add studio/lib/generator/write.ts \
  studio/components/generator/program-runner-pane.tsx \
  frontend/scripts/generator/check-dev-write-guard.mjs \
  frontend/scripts/generator/seed-generator-service-starters.mjs
git commit -m "feat: add generator v2 dev-only write path"
```

## Task 6: Freeze Legacy Templating and Export Migration Inventory

**Files:**
- Create: `studio/lib/generator/legacy.ts`
- Create: `frontend/scripts/generator/export-legacy-templates.mjs`
- Modify: `docs/astro-migration-megaplan.md`

- [ ] **Step 1: Write the failing legacy inventory export**

Create `frontend/scripts/generator/export-legacy-templates.mjs`:

```js
console.error("Legacy template export not implemented");
process.exit(1);
```

- [ ] **Step 2: Verify it fails**

Run: `node frontend/scripts/generator/export-legacy-templates.mjs`

Expected: FAIL with placeholder error.

- [ ] **Step 3: Implement a read-only exporter**

Replace the script with a JSON exporter that fetches:
- `pageTemplate`
- `pageLocation`
- `serviceLocation`

Minimal query:

```js
const pageTemplates = await client.fetch(`
  *[_type == "pageTemplate"]{
    _id,
    title,
    lane,
    variant,
    "slug": slug.current,
    shellId,
    topBlockCountDefault
  }
`);
```

Write to:
- `frontend/tmp/generator-legacy-template-inventory.json`

- [ ] **Step 4: Implement a read-only legacy mapper**

Create `studio/lib/generator/legacy.ts`:

```ts
export const mapLegacyTemplateToGeneratorSeed = (legacy: {
  title?: string;
  lane?: string;
  shellId?: string;
}) => ({
  title: legacy.title || "Migrated Legacy Template",
  designFamily: legacy.lane || "generic",
  shellId: legacy.shellId || null,
});
```

- [ ] **Step 5: Run inventory export**

Run: `node frontend/scripts/generator/export-legacy-templates.mjs`

Expected:
- file created at `frontend/tmp/generator-legacy-template-inventory.json`
- no writes to public page content

- [ ] **Step 6: Update migration tracking**

Mark the legacy freeze and generator migration inventory in `docs/astro-migration-megaplan.md`.

- [ ] **Step 7: Commit**

```bash
git add studio/lib/generator/legacy.ts \
  frontend/scripts/generator/export-legacy-templates.mjs \
  frontend/tmp/generator-legacy-template-inventory.json \
  docs/astro-migration-megaplan.md
git commit -m "chore: export legacy template inventory for generator v2 migration"
```

## Task 7: Verify Frontend Rendering and Repo Tracking

**Files:**
- Modify: `frontend/sanity/queries/page.ts`
- Modify: `frontend/sanity/lib/fetch.ts`
- Modify: `docs/seo-updates.md`
- Modify: `docs/astro-migration-megaplan.md`

- [ ] **Step 1: Add minimal frontend debug contract**

If needed for QA, extend `frontend/sanity/queries/page.ts` with generator metadata:

```ts
generator{
  programId,
  templateId,
  rowKey,
  keywordKey,
  generatedAt,
  version,
  aiUsed
},
```

- [ ] **Step 2: Run frontend typecheck to catch query drift**

Run: `pnpm --filter frontend run typecheck`

Expected: PASS, or type errors that force query/typing alignment.

- [ ] **Step 3: Run end-to-end repo checks**

Run:
- `pnpm --filter studio run typecheck`
- `pnpm --filter frontend run typecheck`
- `node frontend/scripts/generator/run-generator-smoke.mjs`
- `node frontend/scripts/generator/export-legacy-templates.mjs`

Expected:
- all checks pass
- dry-run and legacy export produce deterministic output

- [ ] **Step 4: Update required logs**

Add one entry per implementation cycle to:
- `docs/seo-updates.md`
- `docs/astro-migration-megaplan.md`

Each entry must list:
- date
- changed files
- summary
- SEO/integration impact
- verification

- [ ] **Step 5: Final commit**

```bash
git add docs/seo-updates.md \
  docs/astro-migration-megaplan.md \
  frontend/sanity/queries/page.ts \
  frontend/sanity/lib/fetch.ts
git commit -m "docs: track generator v2 implementation verification"
```

## Spec Coverage Check

- `generator in Studio, not external`: covered by Tasks 1 and 2.
- `bulk keyword + row variation`: covered by Tasks 1, 3, and 4.
- `aligned design, unique services`: covered by Task 3 variation logic.
- `manual edits preserved`: covered by Task 5 duplicate/skip behavior and draft-only writes.
- `AI-ready but not dependent on AI`: covered by Task 1 schema field `aiMode` and Task 3 deterministic core.
- `do not disturb prod`: covered by Task 5 write guards and Task 6 freeze-first migration strategy.
- `legacy templating should be frozen then mined`: covered by Task 6.

## Placeholder Scan

The only allowed placeholders in this plan are deliberate minimal starter implementations used to force a failing test or failing smoke script before the real implementation step. No open-ended `TODO/TBD` implementation gaps remain.

## Type Consistency Check

- New schema names are consistent:
  - `generatorTemplate`
  - `generatorProgram`
  - `generatorDataset`
  - `generatorPageMeta`
- Render function name is consistent across tasks:
  - `buildGeneratedPageDraft`
- Metadata field names are consistent across schema, render, and query tasks:
  - `programId`
  - `templateId`
  - `rowKey`
  - `keywordKey`
  - `generatedAt`
  - `version`
  - `aiUsed`
