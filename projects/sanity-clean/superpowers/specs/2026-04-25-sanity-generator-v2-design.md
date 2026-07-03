# Sanity Generator V2 Design

Date: 2026-04-25
Status: Drafted and approved in working session for planning
Scope: New dev-only generator system for Next.js + Sanity block pages, replacing the current legacy templating direction over time without touching production runtime

## Goal

Build a new page generator experience that is operationally close to Page Generator Pro, but runs on top of Next.js and Sanity block content.

The editor experience should be simple:
- choose a base template
- enter bulk keywords and variation rows
- preview one result
- generate many pages

The generated output must stay:
- organic in tone
- structurally aligned in design
- unique per service or keyword angle
- safe from obvious duplication
- editable as normal `page` documents after generation

## Product Direction

The system is a hybrid generator:
- bulk input is driven by keyword diversification
- variation input is driven by row data
- output uses one consistent design system with controlled section variation
- generated pages are real Sanity `page` documents, not virtual runtime pages

This keeps the Page Generator Pro workflow, but avoids runtime complexity in the frontend.

## Non-Goals

- No production dataset writes in V1
- No automatic publish in V1
- No runtime page generation inside Next.js request/render flow
- No reuse of the old templating runtime as the foundation of the new generator
- No dependence on LiteLLM for V1 launch readiness, though the tooling must be ready for later AI integration

## Current-State Diagnosis

The current templating system is too mixed across schema, frontend runtime resolution, and route inference.

Main problems:
- `pageTemplate` is carrying too many responsibilities
- frontend template resolution is doing generation-like work at runtime
- route logic and template logic are coupled
- legacy rewrite and new template concepts overlap
- the mental model for editors is not simple enough to support bulk generation

The new system should move generation responsibility into Studio tooling and leave the frontend responsible only for rendering final pages.

## Recommended Architecture

Use a new generator stack inside Sanity Studio, isolated from production flow.

Core units:
- `generatorTemplate`
  - minimal block template and variation rules
- `generatorProgram`
  - one operator-facing generation project, similar to a Page Generator Pro content group
- `generatorDataset`
  - keyword sets, rows, import payloads, and run-ready inputs
- `page`
  - final generated output, still the only document type consumed by the frontend

Frontend responsibility:
- render `page` as usual
- do not infer or generate template output at runtime

Studio responsibility:
- configure programs
- preview one generated page
- generate pages in batch
- track generation metadata

## Where The New Generator Runs

The generator runs inside Sanity Studio as editorial tooling, not as a separate external app.

Why:
- keeps content operators in one place
- lets generated output land directly in Sanity documents
- supports preview and manual editing without extra sync layers
- reduces drift between template model and actual content schema

Heavy generation logic may still be executed by internal actions, scripts, or API handlers triggered from Studio, but Studio remains the control surface.

## Workflow

The target V1 workflow is:

1. create a `Generator Program`
2. choose one `Generator Template`
3. add `Keyword Sets`
4. add `Rows`
5. preview one combination
6. inspect slug, title, section plan, and content preview
7. generate draft pages in bulk
8. optionally open any output as a normal `page` and edit it manually

The Studio screen should be organized around four operator panels:
- `Program Setup`
- `Inputs`
- `Preview`
- `Run`

This is intentionally simpler than the current templating setup.

## Input Model

The generator uses both keyword sets and row data.

### Keyword Sets

Used for bulk diversification.

Examples:
- `jasa cetak buku`
- `cetak buku murah`
- `percetakan buku`

### Row Data

Used for entity-specific variation.

Examples:
- service
- city
- industry
- offer angle
- CTA target
- proof emphasis

### Composition Rule

Each generated page is built from:

`generatorTemplate + keyword set + row data + variation rules`

This allows the system to scale in bulk without producing near-clone pages.

## Anti-Duplicate Strategy

Bulk generation must not create pages that only swap one keyword while keeping the same narrative.

V1 requires three variation layers:

### 1. Angle Variation

Each page should emphasize one primary angle such as:
- price
- speed
- quality
- location coverage
- customization
- minimum order

### 2. Section Variation

Not every page should render the exact same section stack.

Allowed controls:
- optional sections
- swap between approved section variants
- limited section-order differences where safe

### 3. Copy Token Rules

Keywords, entities, and CTA values should be inserted into different semantic positions, not repeated flatly throughout the page.

Slots should distinguish:
- primary keyword
- supporting keyword
- service name
- city or segment
- proof point
- CTA language

## Design Consistency With Service Uniqueness

The user requirement is clear:
- each service page should feel unique
- the design system should still stay aligned

The system will handle this by separating:
- visual skeleton
- section choices
- content angle

The visual skeleton stays stable at template level:
- same base typography rhythm
- same block families
- same spacing and CTA treatment

Uniqueness comes from:
- chosen sections
- headline and proof emphasis
- FAQ set
- CTA framing
- service and segment details

This prevents visual fragmentation while keeping page intent specific.

## Data Model

The initial schema should stay minimal.

### `generatorTemplate`

Required fields:
- `title`
- `slug`
- `outputType` default `page`
- `baseSections`
- `optionalSections`
- `tokenDefinitions`
- `variationRules`
- `designFamily`
- `status`

### `generatorProgram`

Required fields:
- `title`
- `slug`
- `templateRef`
- `routeBase`
- `programType`
- `status`
- `generationMode`
- `datasetRef`
- `defaultSeoPattern`
- `aiMode` default `off`

### `generatorDataset`

Required fields:
- `title`
- `keywordSets`
- `rows`
- `importMode`
- `dedupePolicy`
- `status`

### Generated `page`

Generated pages remain standard `page` documents, with generation metadata added under a namespaced object:
- `generator.programId`
- `generator.templateId`
- `generator.rowKey`
- `generator.keywordKey`
- `generator.generatedAt`
- `generator.version`
- `generator.aiUsed`

## Manual Editing Contract

Generated pages are first-class editable pages.

Rules:
- manual edits are allowed after generation
- generator reruns must not silently overwrite manual edits
- rerun behavior should support explicit modes later:
  - create only missing
  - regenerate selected drafts
  - sync structure only

V1 should default to conservative behavior:
- generate draft pages
- skip existing outputs unless operator explicitly regenerates selected targets

## AI Integration Boundary

AI is optional for V1 launch but the system should be ready for it.

The correct place for AI is the generator pipeline layer, not the old template schema or frontend runtime.

Future AI integration path:
- Studio operator chooses AI mode per program or per run
- generation action calls a provider adapter
- adapter can later route through LiteLLM
- AI output is constrained by template slots and section rules

AI should be used to:
- enrich draft copy
- generate angle-specific variants
- diversify FAQ and CTA wording
- improve narrative transitions

AI should not be allowed to:
- invent unsupported block shapes
- bypass template section rules
- mutate live production content directly

## Dev Isolation

The project must not affect production while being built.

V1 isolation rules:
- use development credentials first
- write only to development dataset by default
- use new schema names
- keep old templating system untouched during initial build
- generated page IDs must use a safe dev-only naming strategy
- no frontend production route changes until cutover is deliberate

This work should be treated as a new project inside the repo, not an in-place production refactor.

## Legacy Templating Strategy

Do not delete the entire old templating system as the first step.

Recommended migration stance:
- freeze the old templating engine as legacy
- keep useful template content as migration source material
- do not let the new system depend on old runtime resolver behavior
- extract only the valuable pieces into the new `generatorTemplate` model

Migration phases:

### Phase 1

Build the new generator stack in isolation.

### Phase 2

Map useful old template parts into the new template model.

### Phase 3

Test generation flow end-to-end in development dataset.

### Phase 4

After the new generator is stable, retire old templating runtime and schema surfaces.

## Error Handling

The generator should fail visibly and narrowly.

Expected validation classes:
- missing template reference
- missing required tokens
- duplicate target slug
- invalid route base
- invalid section composition
- unsupported AI response shape
- dataset import shape mismatch

Preview should block generation when required validation fails.

Batch generation should report:
- generated
- skipped
- conflicted
- failed

## Verification Strategy

V1 verification should include:
- Studio schema validation
- preview generation for one row
- batch generation for a small dev sample
- duplicate-slug detection
- manual edit preservation checks
- frontend rendering checks for generated `page` documents
- public-read checks on generated documents where applicable

The frontend should only need to prove that a generated `page` renders correctly. It should not own generator logic.

## Implementation Shape

The first implementation plan should focus on:

1. new schemas for `generatorTemplate`, `generatorProgram`, and `generatorDataset`
2. a minimal Studio workflow for setup, inputs, preview, and run
3. deterministic non-AI generation pipeline
4. dev-only output to standard `page` docs
5. generation metadata and dedupe protection
6. migration inventory for useful legacy template assets

AI integration and full LiteLLM routing should be treated as a follow-up layer after the deterministic pipeline is stable.

## Decision Summary

Chosen design decisions:
- generator lives inside Sanity Studio
- output is real `page` documents
- input model uses both keyword sets and row data
- generated pages remain manually editable
- design stays aligned through stable block templates
- uniqueness comes from controlled content and section variation
- AI is optional and added later through a generator adapter
- old templating is frozen, mined for value, then retired after migration
- all work stays dev-only until an explicit cutover

This design intentionally resets the templating direction around operator simplicity:

`template + input data + preview + generate`

That is the core product behavior the current system is missing.
