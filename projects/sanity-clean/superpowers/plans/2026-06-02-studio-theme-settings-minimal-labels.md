# Studio Theme Settings Minimal Labels Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove HEX codes from `Theme Settings` dropdown labels in Sanity Studio while keeping the stored values and visual swatches unchanged.

**Architecture:** This change stays inside the Studio schema constants that provide dropdown option labels. No schema shape, stored values, frontend fetch contract, or rendering logic changes are required.

**Tech Stack:** Sanity Studio, TypeScript

---

### Task 1: Simplify Theme Option Labels

**Files:**
- Modify: `studio/schemas/documents/theme-settings.ts`
- Modify: `docs/seo-updates.md`

- [ ] **Step 1: Inspect the current label constants**

Read `GEIST_COLOR_OPTIONS` and `GEIST_FOREGROUND_OPTIONS` in `studio/schemas/documents/theme-settings.ts` and confirm that only `title` text needs to change.

- [ ] **Step 2: Update labels with minimal scope**

Change labels such as `Gray 10 (#171717)` to `Gray 10` and keep `value` unchanged.

Expected code shape:

```ts
{ title: "Gray 10", value: "#171717" }
```

- [ ] **Step 3: Preserve fallback and visual behavior**

Do not change:

```ts
{ title: "Default (Follow Code)", value: "" }
components: { input: ColorOptionInput }
```

- [ ] **Step 4: Document the repository change**

Add an entry to `docs/seo-updates.md` that records the Studio schema file change, states `No direct SEO impact`, and notes verification status.

- [ ] **Step 5: Verify with a targeted sanity check**

Run a focused check that the updated file still parses cleanly within the repo toolchain. Prefer an existing lint or typecheck command if available for the Studio workspace.

## Self-Review

- Spec coverage: the plan only changes label text and preserves existing values and swatches.
- Placeholder scan: no TODO/TBD placeholders remain.
- Type consistency: all examples keep the current `title` and `value` object shape.
