# Generator Template Pool Design

## Goal

Allow a Generator Program to use 1–3 selected templates and choose one template per generated page, so mass-generated pages can vary layout/content structure while staying stable between Dry Run and Generate Drafts.

## Current behavior

- `generatorProgram` stores one `template` reference.
- `ProgramRunnerPane` loads that one template with `TEMPLATE_QUERY`.
- Preview, Dry Run, and Generate Drafts call `buildGeneratedPageDraft({ program, template, row })`.
- The generated page lineage stores the template actually used in `generator.templateId` and `generator.template`.
- AI shortcodes are resolved after the draft is built, so template choice must happen before AI resolution.

## Decision

Add a template pool to `generatorProgram` and select one template deterministically per row/page.

The runner will use `templatePool` when it contains at least one template. Existing programs without `templatePool` will continue using the existing `template` field. This keeps old documents valid and avoids a migration requirement.

## Studio schema

Add `templatePool` to `studio/schemas/documents/generator-program.ts`:

- Type: `array` of references to `generatorTemplate`.
- Validation: `max(3)`.
- Description: selecting multiple templates enables deterministic per-row random variation.
- Existing `template` remains required for compatibility and fallback.

Optional future cleanup: after all programs use `templatePool`, the single `template` field can be made optional or renamed, but that is out of scope.

## Runtime selection

Add a small deterministic selector in generator runtime code:

- Input: `programId`, `row.key || row._key || row.primaryKeyword`, and the resolved template pool.
- Output: one `GeneratorTemplateLite`.
- Algorithm: stable non-cryptographic hash over the seed string, then `hash % templates.length`.
- If pool length is 0, use the legacy single template.
- If pool length is 1, use that template.

This is intentionally deterministic, not `Math.random()`, because Dry Run and Generate Drafts must agree. Re-running the same program against the same row should not silently switch templates.

## Runner data flow

Update `studio/components/generator/program-runner-pane.tsx`:

1. Read `program.templatePool` from the displayed program document.
2. Fetch all selected pool templates in one query.
3. Build `linkedData` with `templates: GeneratorTemplateLite[]` instead of only `template` internally.
4. Keep `template: GeneratorTemplateLite | null` compatibility where it simplifies fallback.
5. Preview uses the selected template for `selectedRow`.
6. Dry Run loops rows and selects the template for each row before calling `buildGeneratedPageDraft`.
7. Generate Drafts writes the drafts produced by Dry Run, preserving each draft's selected `templateId`.

## UI behavior

In the runner setup card:

- Show the legacy template title when only one template is active.
- Show `Template Pool: N templates` when multiple templates are active.
- For Preview Output, show the selected template title for the current preview row.

Blocking states:

- Block when neither `templatePool` nor legacy `template` resolves.
- Keep existing dataset/status blocking behavior.

## Generated page lineage

No generated page schema change is required.

`buildGeneratedPageDraft` already records:

- `generator.templateId`
- `generator.template`

Because the selected template is passed into the existing function, lineage remains accurate per page.

## Token and SEO constraints

All templates in a pool should be compatible with the same dataset. The runner will not try to merge or reconcile token definitions across templates.

Practical rule:

- If a template references a missing token, existing interpolation behavior applies and the missing token becomes an empty string.
- QA remains per generated draft, so poor output from incompatible templates is visible during Dry Run.

## Non-goals

- Do not mix sections/blocks from multiple templates into one page.
- Do not make template choice non-deterministic per click/run.
- Do not change AI shortcode semantics.
- Do not migrate existing Sanity documents automatically.

## Testing

Add or update focused tests for the deterministic selector and draft generation flow if existing test seams allow it.

Minimum verification:

- Typecheck Studio/runner changes.
- Verify a program with only legacy `template` still builds a preview.
- Verify a program with 2–3 pool templates selects stable template IDs for the same rows across repeated Dry Run calls.
- Verify generated drafts store the selected template ID in lineage.
