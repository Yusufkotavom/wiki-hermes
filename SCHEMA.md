# Wiki Schema

## Domain
Personal knowledge hub + cross-project references + content production + social media + task/plan tracking.

## Conventions
- File names: lowercase, hyphens, no spaces (e.g., `devk-studio.md`, `content-calendar.md`)
- Every wiki page starts with YAML frontmatter (see below)
- Use `[[wikilinks]]` to link between pages (minimum 2 outbound links per page when feasible)
- When updating a page, always bump the `updated` date
- Every new page must be added to `index.md` under the correct section
- Every action must be appended to `log.md`
- Provenance markers: On pages that synthesize 3+ sources, append `^[raw/articles/source-file.md]` at the end of paragraphs whose claims come from a specific source.

## Frontmatter
```yaml
---
title: Page Title
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity | concept | comparison | query | summary | task | content | note
tags: [from taxonomy below]
sources: [raw/articles/source-name.md]
# Optional quality signals:
confidence: high | medium | low
contested: true
contradictions: [other-page-slug]
due: YYYY-MM-DD           # for tasks/content with deadlines
status: inbox | active | done | archived
---
```

`confidence` and `contested` are optional but recommended for opinion-heavy or fast-moving topics. Lint surfaces `contested: true` and `confidence: low` pages for review.

## Tag Taxonomy
- Domains: llm, sanity, nextjs, cms, seo, aeo, content, social
- Project: devk-studio, sanity-clean
- Types: concept, entity, task, content, research, note, plan, reference
- Meta: comparison, timeline, idea, link, tool

Rule: every tag on a page must appear in this taxonomy. If a new tag is needed, add it here first.

## Page Thresholds
- Create a page when an entity/concept appears in 2+ sources OR is central to one source
- Add to existing page when a source mentions something already covered
- DON'T create a page for passing mentions, minor details, or things outside the domain
- Split a page when it exceeds ~200 lines — break into sub-topics with cross-links
- Archive a page when its content is fully superseded — move to `_archive/`, remove from index

## Sections
- `inbox/` — raw ideas, quick notes, links, inbox items awaiting triage
- `tasks/` — tasks, plans, todo, retrospectives; prefer `status` + optional `due`
- `content/` — article drafts, content plans, social copy, publication schedules
- `projects/` — per-project hub pages and project-specific notes
- `concepts/` — reusable concepts and learnings
- `research/` — sources and research notes for content
- `comparisons/` — tool/topic comparisons
- `queries/` — saved answers worth keeping
- `raw/` — immutable source material

## Update Policy
When new information conflicts with existing content:
1. Check the dates — newer sources generally supersede older ones
2. If genuinely contradictory, note both positions with dates and sources
3. Mark the contradiction in frontmatter: `contradictions: [page-name]`
4. Flag for user review in the lint report
