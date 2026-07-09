# CompPlan Studio prototype — progress

_Single file: `comp-plan-prototype.html`. Architecture/conventions live in `CLAUDE.md`; this file
tracks status only._

## Latest change — builder input formatting (uncommitted)
Tightened three builder inputs:
- **VCT Wt%** (measure row): numeric entry with a `%` adornment; stored `"NN%"` (`onBuilderVctInput`).
- **Threshold** (payout): optional, integer-only (`type=text inputmode=numeric` + regex); no default
  `0`; stored as a number or `''`.
- **Cap** (payout): optional, numeric entry with a `%` adornment; stored `"NN%"` (`onBuilderCapInput`).
- Seed cleanup: seeded payout tables now start with **no cap and no threshold** (old free-text cap
  notes dropped, since cap is a user-entered %). `collectPayoutTable`/`normalizePayoutToScheme` updated.
- New CSS `.pct-suffix` mirrors `.xpcr-suffix`. Stored formats unchanged for `parsePctValue`,
  `flavorVctSum`, validation, and Compare. Verified on webui2 (no console errors). **Not committed.**

## Earlier change — bonus becomes a list of up to 5 (committed 10e92f3)
Bonus is no longer a single `planMeta.bonus`/`bonusDescription` pair. Now `planMeta.bonuses` is a
**version-level array of up to 5** (`MAX_BUILDER_BONUSES`), each `{type, payoutDetails, maxPayout}`:
- Bonus Type dropdown = Lead Referral Bonus / Linearity Bonus / M1 & M2 Achievement Bonus (+ a
  "Select bonus type…" placeholder); Payout Details + Maximum payout are free text.
- **No bonus by default** (empty list) for new AND seeded plans (old free-text bonus strings dropped).
- Builder: "Bonus" section is now `#builder-bonus-container` rendered by `renderBuilderBonuses`, with
  Add bonus / Remove per row (`addBuilderBonus`/`removeBuilderBonus`/`setBuilderBonusField`); fields
  bind directly to the model (no DOM-sync). Save/submit preserve `planMeta.bonuses`.
- Compare: bonuses shown as positional `Bonus N` sections (Type / Payout details / Maximum payout);
  `buildCompareFieldMeta(payoutCount,bonusCount)`; new field keys `bonusNType/Details/Max` +
  `bonusCount`; the old single "Bonus" row is gone, `tether` moved to its own "Tether" section.
- Verified on webui2 (no console errors): empty default, add/cap-at-5, save/reopen persistence, seed
  shape, Compare (incl. unequal counts), read-only view. `CLAUDE.md` updated. **Not yet committed.**

## Earlier change — decouple measures from payout tables (committed b0666fd)
Performance measures and payout tables are now **independent — no mapping**:
- Measures lost `measureId` and the "Measure Type Value" dropdown; the free-text
  "Performance Measure(s)" (`description`) is a measure's only name.
- `content.payoutTables` is now a **version-level array** of `{id,title,type,…}` (was an object
  keyed by `measureId`); tables are added/removed manually via an **"Add payout table"** button,
  each with an editable free-text title. Save-time orphan-prune removed.
- Compare shows **measures and payout tables as separate sections**; payout tables listed
  positionally (`Payout table N`, with title + merged render). `buildCompareFieldMeta(payoutCount)`
  rebuilt per comparison. New field keys: `payoutTableNTitle/N/NData/NCap` + `payoutTableCount`.
- Seed migration: `seedVersionContent` builds the deduped array with `id`+`title` (from the former
  metric name); seeded measures get `description` = metric name.
- Verified on webui2 (no console errors): builder edit + independence + save/reopen persistence;
  seed shape; Compare (incl. unequal-count edge); read-only view hides edit affordances.
- `CLAUDE.md` updated (domain model, payout, Compare). **Not yet committed.**

## Completed since last handoff (branch `feature/comp-plan-flavors`, PR #1)
Commits `a5495b8 … 449d0a5` (pushed). Highlights:
- **Per-flavor fields:** HC (numeric, optional), Paymix (optional), and new-hire-guarantee moved off
  `planMeta` onto each flavor. Plan Info shows Plan ID + Plan Name on one line; optional labels
  dropped, required fields marked; **validation runs only at Submit** (plan name, per-flavor role,
  100% VCT).
- **New-hire guarantee** relocated from creation to the **Release Validation** approval stage
  (per-flavor, required to advance out of that stage).
- **Builder measures table:** columns are **Performance Measure(s)** (free text) then **Measure Type
  Value** (dropdown); new flavors/measures start blank with "Select…" placeholders. Bonus/Tether
  descriptions are long-text; added an optional **Key Policies** section (`planMeta.keyPolicies`).
- **Payout tables redesigned** with a per-table **type** (quota_band / target_pct / attainment_only)
  and made **set-driven** (see CLAUDE.md): the 36 hardcoded quota schemes were removed. xPCR is a
  numeric field with an "x" suffix; MCR guidance ≤20% (demo values ≤10%).
- **Configuration tab** (Setup nav): two screens — Quota Band Sets and Attainment Tier Sets — with
  CRUD, upper-bound-only rows + auto ranges, localStorage persistence. Seeded with the real 34 quota
  band sets + 11 attainment tier sets. Quota band sets have editable per-band descriptions (default
  auto-derived, e.g. `<=$1.5 Million USD`) that also show on applied payout bands + in Compare.
- **Compare rework:** symmetric Plan 1/Plan 2 (no baseline/swap/status column — highlight-only),
  no "show differences only" toggle, editable per-plan notes, payout shown as a merged compare table.
- **Layout:** Performance measures & Payout tables shown side-by-side (2-col, stacks < 820px).
- **CLAUDE.md** updated (UI/UX principle; domain model, payout/config, validation now current).

## Current status
- Feature-complete for everything requested; working tree clean; all commits **pushed** to PR #1.
- HEAD: `449d0a5`. UI-only prototype: version data is in-memory (resets on reload); the only
  persistence is config sets in localStorage.

## Remaining work / not yet done
- PR #1 is open, not yet merged/reviewed.
- Not done (never requested): propagating attainment-tier descriptions (tier sets have none);
  payout `payCurveType` (Linear/Stepped) is stored but not consulted by the %VCT calc.

## Known issues / caveats
- Legacy dead code remains (`renderBuilderMeasureBlock` / `renderBuilderMeasures`, older
  clone/duplicate helpers) — harmless; live builder uses `renderBuilderFlavors` /
  `renderBuilderPayoutTables`.
- `preview_screenshot` times out on this renderer — verify via `preview_eval` + `preview_console_logs`.
- Port 4599 is often held by another chat; use `webui2` (4610). Commit diffs can look inflated from
  LF↔CRLF toggling.
- Seeded/published payout tables predate the type/set model; opening one editable shows the type/set
  prompts (read-only display + Compare still render existing tiers fine).

## Recommended next task
Get PR #1 reviewed/merged, or continue refining. For new work: plan-mode workflow (explore → clarify
via AskUserQuestion → write plan file → ExitPlanMode), implement, verify on `webui2`, commit the one
html file only when asked.
