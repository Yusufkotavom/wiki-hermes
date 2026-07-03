# Generator Template Pool Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add 1–3 template pool support to Generator Programs with deterministic per-row template selection.

**Architecture:** Keep the existing single `template` reference as the compatibility fallback. Add a focused selector utility in `studio/lib/generator/template-selection.ts`, then update `ProgramRunnerPane` to fetch active templates and pass the selected template into the existing `buildGeneratedPageDraft` function. Generated page lineage remains unchanged because the selected template is passed through the current draft builder.

**Tech Stack:** Sanity Studio v5 schema, React 19, TypeScript, Sanity UI, Node test runner.

---

## File Structure

- Create `studio/lib/generator/template-selection.ts`: deterministic template choice, independent from React and Sanity client.
- Create `studio/lib/generator/__tests__/template-selection.test.ts`: selector invariants.
- Modify `studio/lib/generator/types.ts`: add `templatePool` to program value shape where runtime needs it.
- Modify `studio/schemas/documents/generator-program.ts`: add `templatePool` array reference, max 3.
- Modify `studio/components/generator/program-runner-pane.tsx`: fetch template pool, select per row, show active/selected template info.

---

### Task 1: Deterministic selector utility

**Files:**
- Create: `studio/lib/generator/template-selection.ts`
- Test: `studio/lib/generator/__tests__/template-selection.test.ts`

- [ ] **Step 1: Write the failing tests**

Create `studio/lib/generator/__tests__/template-selection.test.ts`:

```ts
import test from "node:test";
import assert from "node:assert/strict";

import { selectTemplateForRow } from "../template-selection";
import type { GeneratorRow, GeneratorTemplateLite } from "../types";

const templates: GeneratorTemplateLite[] = [
  { _id: "template-a", title: "Template A" },
  { _id: "template-b", title: "Template B" },
  { _id: "template-c", title: "Template C" },
];

const row: GeneratorRow = {
  _key: "row-surabaya",
  key: "stable-surabaya",
  service: "Cetak Buku",
  city: "surabaya",
  primaryKeyword: "cetak buku surabaya",
};

test("selectTemplateForRow returns the same template for the same program and row", () => {
  const first = selectTemplateForRow({ programId: "program-1", row, templates });
  const second = selectTemplateForRow({ programId: "program-1", row, templates });

  assert.equal(first?._id, second?._id);
});

test("selectTemplateForRow uses row fallback data when key is missing", () => {
  const selected = selectTemplateForRow({
    programId: "program-1",
    row: { ...row, key: undefined, _key: undefined },
    templates,
  });

  assert.ok(selected);
  assert.ok(templates.some((template) => template._id === selected?._id));
});

test("selectTemplateForRow returns the only template without hashing ambiguity", () => {
  const selected = selectTemplateForRow({ programId: "program-1", row, templates: [templates[1]!] });

  assert.equal(selected?._id, "template-b");
});

test("selectTemplateForRow returns null for an empty template list", () => {
  const selected = selectTemplateForRow({ programId: "program-1", row, templates: [] });

  assert.equal(selected, null);
});
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `cd studio && node --import tsx lib/generator/__tests__/template-selection.test.ts`

Expected: FAIL with module not found for `../template-selection`.

- [ ] **Step 3: Implement the selector**

Create `studio/lib/generator/template-selection.ts`:

```ts
import type { GeneratorRow, GeneratorTemplateLite } from "./types";

type SelectTemplateForRowInput = {
  programId: string;
  row: GeneratorRow;
  templates: GeneratorTemplateLite[];
};

const stableHash = (value: string) => {
  let hash = 2166136261;
  for (let index = 0; index < value.length; index += 1) {
    hash ^= value.charCodeAt(index);
    hash = Math.imul(hash, 16777619);
  }
  return hash >>> 0;
};

const getRowSeed = (row: GeneratorRow) =>
  row.key || row._key || row.primaryKeyword || row.city || row.service || "row";

export const selectTemplateForRow = ({ programId, row, templates }: SelectTemplateForRowInput) => {
  if (templates.length === 0) return null;
  if (templates.length === 1) return templates[0] ?? null;

  const hash = stableHash(`${programId}:${getRowSeed(row)}`);
  return templates[hash % templates.length] ?? null;
};
```

- [ ] **Step 4: Run tests to verify they pass**

Run: `cd studio && node --import tsx lib/generator/__tests__/template-selection.test.ts`

Expected: PASS for all selector tests.

---

### Task 2: Schema and runtime types

**Files:**
- Modify: `studio/schemas/documents/generator-program.ts`
- Modify: `studio/lib/generator/types.ts`

- [ ] **Step 1: Update runtime types**

In `studio/lib/generator/types.ts`, add optional template pool support to `GeneratorProgramLite`:

```ts
templatePool?: ReferenceValue[];
```

Place it near the existing `ref?: ReferenceValue` and `dataset?: GeneratorDatasetLite` fields.

- [ ] **Step 2: Update Sanity schema**

In `studio/schemas/documents/generator-program.ts`, insert a `templatePool` field after the existing `template` field:

```ts
defineField({
  name: "templatePool",
  title: "Template Pool",
  type: "array",
  description:
    "Optional: choose 1–3 templates for deterministic per-row variation. If empty, the single Template field is used.",
  of: [
    defineField({
      name: "template",
      type: "reference",
      to: [{ type: "generatorTemplate" }],
    }),
  ],
  validation: (Rule) => Rule.max(3),
}),
```

- [ ] **Step 3: Typecheck schema/types**

Run: `cd studio && pnpm typecheck`

Expected: TypeScript completes without errors.

---

### Task 3: Runner pool fetching and selection

**Files:**
- Modify: `studio/components/generator/program-runner-pane.tsx`

- [ ] **Step 1: Import selector and extend types**

Add:

```ts
import { selectTemplateForRow } from "../../lib/generator/template-selection";
```

Update `GeneratorProgramValue` with:

```ts
templatePool?: ReferenceValue[];
```

Update `LinkedGeneratorData` with:

```ts
templates: GeneratorTemplateLite[];
```

- [ ] **Step 2: Add template pool query**

Add near `TEMPLATE_QUERY`:

```ts
const TEMPLATES_QUERY = `*[_type == "generatorTemplate" && _id in $ids]{
  _id, title,
  routeBase, slugPattern, seoMeta, aggregateRatingDefaults,
  tokenDefinitions[]{_key, name, label, sourceField, fallbackValue, required},
  blocks
}`;
```

- [ ] **Step 3: Load active template IDs**

In the `useEffect`, compute:

```ts
const legacyTemplateId = program.template?._ref;
const poolTemplateIds = (program.templatePool ?? [])
  .map((reference) => reference?._ref)
  .filter((id): id is string => typeof id === "string" && id.length > 0)
  .slice(0, 3);
const templateIds = poolTemplateIds.length > 0 ? poolTemplateIds : legacyTemplateId ? [legacyTemplateId] : [];
```

Use `templateIds` instead of the single `templateId` guard. Fetch templates via `TEMPLATES_QUERY`; keep dataset and existing pages fetches as they are.

Set linked data with:

```ts
const templatesById = new Map((templates ?? []).map((template) => [template._id, template]));
const orderedTemplates = templateIds.flatMap((id) => {
  const template = templatesById.get(id);
  return template ? [template] : [];
});
setLinkedData({ template: orderedTemplates[0] ?? null, templates: orderedTemplates, dataset, existingPages: existingPages ?? [] });
```

- [ ] **Step 4: Add selected template helpers**

After `selectedRow`, add:

```ts
const activeTemplates = linkedData.templates.length > 0
  ? linkedData.templates
  : linkedData.template
    ? [linkedData.template]
    : [];

const selectedTemplate = selectedRow
  ? selectTemplateForRow({ programId: programLite._id, row: selectedRow, templates: activeTemplates })
  : null;
```

Use `selectedTemplate` for preview draft. In `buildDryRunResult`, select a template per row inside the loop and pass it to `buildGeneratedPageDraft`.

- [ ] **Step 5: Update blocking and summary text**

Replace single-template checks with active-template checks:

```ts
if (activeTemplates.length === 0) blockingIssues.push("Select at least one generator template.");
```

Add notes:

```ts
notes.push(`Templates: ${activeTemplates.length}`);
if (selectedTemplate?.title) notes.push(`Preview template: ${selectedTemplate.title}`);
```

Show in setup card:

```tsx
<Text size={1}>Template: {activeTemplates.length > 1 ? `${activeTemplates.length} templates in pool` : valueOrFallback(activeTemplates[0]?.title)}</Text>
{selectedTemplate?.title && <Text size={1}>Preview Template: {selectedTemplate.title}</Text>}
```

- [ ] **Step 6: Typecheck runner**

Run: `cd studio && pnpm typecheck`

Expected: TypeScript completes without errors.

---

### Task 4: Focused verification and docs log

**Files:**
- Modify: `docs/seo-updates.md` only if the change affects SEO-generated metadata behavior in practice.

- [ ] **Step 1: Run selector tests**

Run: `cd studio && node --import tsx lib/generator/__tests__/template-selection.test.ts`

Expected: PASS.

- [ ] **Step 2: Run existing render tests**

Run: `cd studio && node --import tsx lib/generator/__tests__/render.test.ts`

Expected: PASS.

- [ ] **Step 3: Run Studio typecheck**

Run: `cd studio && pnpm typecheck`

Expected: PASS.

- [ ] **Step 4: Decide SEO log**

If the implementation changes generated SEO fields or metadata logic, append a dated entry to `docs/seo-updates.md`. If lineage/template selection changes only the source template passed into existing SEO interpolation, no SEO log is required.
