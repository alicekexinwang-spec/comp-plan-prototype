# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

# Project: Adobe CompPlan Studio — FY27 prototype

**Deliverable:** `comp-plan-prototype.html` — one self-contained HTML file (~5000 lines, all
CSS/JS inline). No build, no backend; only CDN deps (Chart.js, Tabler icons). Open directly or
serve statically. State lives in in-memory JS arrays near the top of the `<script>`.

## UI/UX (prototypes)
Optimize for **end-user clarity**. Exclude internal/system/process metadata from the UI, and
include only the minimum user-facing information needed to understand and use the screen.

## Domain model (source of truth = those arrays)
- `compPlans` — plan registry, one per (`planId` e.g. `A01`, `fiscalYear`): planName, owner,
  roleGroup, status, `activePublishedVersionId`.
- `planFlavors` — flavor children: `flavorId` (`A01.A`), `planId`, `flavorLabel` (A/B/C), `role`,
  `flavorName`, `fiscalYear`.
- `planVersions` — one approval chain per plan. **Its field named `flavorId` actually holds the
  PLAN id** (historical naming). Each: `versionId` (`ver-A01-FY27-2`), `status`, `approvalStage`,
  timestamps, `isActivePublished`, and `content`.
- `version.content` (the real per-version store the builder reads/writes) =
  `{ planMeta{planName,bonuses[{type,payoutDetails,maxPayout}],tether,keyPolicies},
  flavors[{flavorId,flavorLabel,role,flavorName,headcount,payMixBase,payMixVar,newHireGuarantee,
  measures[{description,vct,perf,pay}]}],
  payoutTables[{id,title,type,bandSetId,thresholds[{label,description,tierSetId,mcr,
  tiers[{attainmentFrom,attainmentTo,xPCR,vctAtMax,mcr}]}],mcr,note,globalVctCap,prorateVct,payCurveType,threshold}] }`.
  **Performance measures and payout tables are INDEPENDENT — no mapping.** Measures have no
  `measureId`; the free-text "Performance Measure(s)" (`description`) is the measure's only name.
  `payoutTables` is a **version-level array** (shared across the version's flavors) added manually via
  an **"Add payout table"** button; each has its own `id` (`payoutUid()`) + free-text `title`.
  `planMeta.bonuses` is a **version-level list of up to 5** (`MAX_BUILDER_BONUSES`), each
  `{type,payoutDetails,maxPayout}` — **no bonus by default** (empty); Bonus Type is a dropdown
  (`BONUS_TYPE_OPTIONS`: Lead Referral / Linearity / M1 & M2 Achievement), the other two are free text
  (handlers `addBuilderBonus`/`removeBuilderBonus`/`setBuilderBonusField`, render `renderBuilderBonuses`).
  **HC / Paymix / new-hire-guarantee are per-flavor** (not planMeta). HC/Paymix are optional; plan
  name + per-flavor role are required.
- Keys: `planKey(planId,fy)` = `"planId|fy"`; `parsePlanKey` splits it.

## Hierarchy & rules
- **Compensation Plan (ID + fiscal year) → Versions → Flavors.** Approval is plan-level and
  covers all flavors in a version.
- Statuses: `draft, pending_review, approved, withdrawn, published`. 8 approval stages:
  `draft, first_review, second_review, executive_review, release_validation, document_generation,
  payout_system_config, completed`. `statusFromStage()` derives status from stage; **only one
  version per plan may be in the approval queue at a time** (`getPendingApprovalVersions`).
- Payout **%VCT is stepped/marginal (tax-bracket): cumulative Σ(bracket width × xPCR), capped by
  the table Cap**; open-ended top tier shows `—`.
- Payout tables are added independently (not derived from measures), each with a free-text `title`
  (handlers `addBuilderPayoutTable` / `removeBuilderPayoutTable` / `setBuilderPayoutTitle`; the
  builder maps DOM slot → array index via `builderPayoutSlotIndex`). Each has a **type**
  (`quota_band` | `target_pct` | `attainment_only`) and is
  **set-driven** (the old 36 `QUOTA_BAND_SCHEMES` are gone): `quota_band` picks a **quota band set**
  (→ bands, remembered on `pt.bandSetId`) and each band picks an **attainment tier set** (→ tier
  upper bounds, remembered on `threshold.tierSetId`); `target_pct` = manually-added bands, each with a
  tier set; `attainment_only` = one implicit band + tier set (per-tier free-text "VCT at tier max",
  table-level `pt.mcr`). Tier upper bounds + band labels are **set-defined/read-only**; the user fills
  xPCR per tier and MCR per band. Handlers: `onQuotaBandSetChange`, `setBandTierSet`.
- **New-hire guarantee** is entered per-flavor at the **Release Validation** approval stage
  (`setReleaseValidationGuarantee`; `advanceVersionStage` blocks leaving `release_validation` until
  every flavor has one) — NOT at plan creation.

## Configuration (reusable sets)
- Setup nav has two Config screens — **Quota Band Sets** and **Attainment Tier Sets** — backed by
  `quotaBandSets` / `attainmentTierSets`, persisted to localStorage (`compplan_config_sets_v4`). Each
  set = `{id,name,bounds:[num]}` (band sets also carry a parallel `descriptions:[str]`); ranges
  auto-derive from the upper bounds. These feed the payout builder's set-driven construction, and a
  band's description shows on applied payout bands + in Compare.

## Conventions
- Builder is driven entirely by `version.content` via `builderWorkingContent`; edits persist on
  **Save** (`saveBuilderVersion`) — no auto-save. `validateVersionContent` (enforced only at
  **Submit**, not Save) requires plan name, per-flavor role, and 100% VCT per flavor.
- Compare is **symmetric** (Plan 1 / Plan 2, no baseline/swap/status-column — differences highlighted
  only), always shows all fields, has editable per-plan notes, and shows **measures and payout tables
  as separate sections** — payout tables listed positionally (`Payout table N`) each with its title +
  a **merged table** (`renderComparePayoutTable`). Snapshots (`buildPlanSnapshots`) are derived per
  (version, flavor) and emit positional field keys (measures: `measureNName`(=description)/`measureNVct`/…;
  payout: `payoutTableNTitle`/`payoutTableN`/`payoutTableNData`/`payoutTableNCap` + `payoutTableCount`;
  bonuses: positional `Bonus N` sections `bonusNType`/`bonusNDetails`/`bonusNMax` + `bonusCount`).
  `buildCompareFieldMeta(payoutCount,bonusCount)` is rebuilt per comparison from `max(both counts)`.
- Legacy `plans[]` array + `syncLegacyFromVersions` shims still back parts of some screens; keep
  them in sync on writes (`reconcilePlanFlavors`, publish/clone paths do this).
- There is `renderBuilderMeasureBlock`/`renderBuilderMeasures` legacy dead code (pre-`version.content`);
  the live builder uses `renderBuilderFlavors` + `renderBuilderPayoutTables`.

## Run / verify
- Preview server `webui` (port 4599) from `CODEX/.claude/launch.json`; open
  `/comp-plan-prototype.html`. If 4599 is held by another chat, use `webui2` (port 4610,
  `static-server-4610.js`) — same file, local-only config. `preview_screenshot` is unreliable on this
  renderer — verify via `preview_eval` DOM reads + `preview_console_logs` (error level). Config sets
  live in localStorage; clear `compplan_config_sets_v4` for a fresh seed.

## Git
- Repo root is the **home dir** (`C:\Users\kexinw`) with many unrelated untracked files —
  **only ever stage `OneDrive - Adobe/Documents/Claude Project/comp-plan-prototype.html`** (and
  these handoff docs). Work on branch `feature/comp-plan-flavors`; commit only when asked; PR #1 →
  `master` on `alicekexinwang-spec/comp-plan-prototype`. The file's line endings toggle LF/CRLF,
  so some commit diffs look large (line-ending noise) — content changes are smaller.
