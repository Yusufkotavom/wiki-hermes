# Studio Theme Settings Minimal Labels

## Goal

Make the Sanity Studio `Theme Settings` UI feel more minimal by removing HEX codes from dropdown labels, while keeping the existing color swatch preview so editors can still recognize each option visually.

## Current State

- `studio/schemas/documents/theme-settings.ts` defines `GEIST_COLOR_OPTIONS` and `GEIST_FOREGROUND_OPTIONS` with labels like `Gray 10 (#171717)`.
- The schema already uses the custom `ColorOptionInput` component, so the editor can see the selected color visually.
- Frontend theme behavior reads stored values, not display labels.

## Proposed Change

- Update the option labels in `GEIST_COLOR_OPTIONS` and `GEIST_FOREGROUND_OPTIONS` to remove the raw HEX suffix.
- Keep the option `value` unchanged so stored data and frontend rendering remain fully compatible.
- Keep `ColorOptionInput` unchanged unless inspection during implementation shows it depends on the old label format.

## Recommended Labels

- Keep semantic names such as `Gray 10`, `Blue 6`, `Green 6`, `Dark Background`.
- Keep `Default (Follow Code)` as-is because it explains fallback behavior without exposing implementation detail.
- If duplicate human-readable labels appear, use a short disambiguator such as `Dark Background 1` and `Dark Background 2` rather than HEX.

## Impact

- Studio editing UX becomes cleaner and less technical.
- No schema shape changes.
- No content migration required.
- No frontend query, fetch, or rendering changes required.

## Risks

- Two similar colors may be harder to distinguish in text alone.
- This risk is acceptable because the Studio already uses visual color swatches.

## Verification

1. Open `Theme Settings` in Sanity Studio.
2. Confirm dropdown labels no longer include HEX codes.
3. Confirm swatches still render correctly in the custom input.
4. Confirm selecting and saving an existing option still stores the same value.

## Out of Scope

- Removing manual override fields.
- Changing preset behavior.
- Changing frontend theme token logic.
