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
  `{ planMeta{planName,keyPolicies,designerNote},
  flavors[{flavorId,flavorLabel,role,flavorName,headcount,payMixBase,payMixVar,newHireGuarantee,
  measures[{description,value,vct,perf,pay,payoutTableId,mechType}],
  tether, bonuses[{type,payoutDetails,maxPayout}],
  payoutTables[{id,title,type,bandSetId,thresholds[{label,description,tierSetId,mcr,
  tiers[{attainmentFrom,attainmentTo,xPCR,vctAtMax,mcr}]}],mcr,note,globalVctCap,prorateVct,payCurveType,threshold}],
  reviewStatus, reviewNote}] }`.
  **FULL PER-FLAVOR OWNERSHIP:** measures, `tether`, `bonuses[]`, and `payoutTables[]` are all owned
  **per flavor** (no version-level `content.payoutTables` and no `planMeta.bonuses`/`.tether` — removed).
  The measure's `description` (name) + `value` are chosen from **config-driven dependent dropdowns** (see
  Configuration: Performance Measures) — pick a measure, then a Value filtered to that measure's values
  (changing the measure resets the value; `value` is optional, not submit-validated). Measures are labelled
  **M1/M2/M3** by index. Each measure has a **`payoutTableId`** linking it to exactly one payout table **on
  its own flavor** (chosen via a "Payout table" dropdown on the measure row, scoped to that flavor's tables),
  and the builder shows that **linked table read-only inline** under the measure row
  (`renderPayoutMatrixTables`). **Submit**
  (`validateVersionContent`) requires — **per flavor** — every measure to link to an existing table on
  that flavor and every one of that flavor's tables to have ≥1 measure linked. Measures carry no
  `measureId`/metric. Each flavor's `payoutTables` is added via a per-flavor **"Add payout table"** button;
  each has its own `id` (`payoutUid()`) + free-text `title`. Each flavor's `bonuses` is a **per-flavor list
  of up to 5** (`MAX_BUILDER_BONUSES`), each `{type,payoutDetails,maxPayout}` — **no bonus by default**
  (empty); Bonus Type is a dropdown (`BONUS_TYPE_OPTIONS`: Lead Referral / Linearity / M1 & M2 Achievement),
  the other two are free text (handlers `addBuilderBonus(fi)`/`removeBuilderBonus(fi,i)`/`setBuilderBonusField(fi,i,…)`,
  render `renderBuilderFlavorBonuses(fi)`). **HC / Paymix / tether / bonuses / payout tables / new-hire-guarantee
  / reviewStatus are all per-flavor** (not planMeta). HC/Paymix are optional; plan name + per-flavor role are
  required. `reviewStatus` (`pending`/`approved`/`needs_rework`) + `reviewNote` are set at the review stages
  (see Approvals).
- Keys: `planKey(planId,fy)` = `"planId|fy"`; `parsePlanKey` splits it.

## Hierarchy & rules
- **Compensation Plan (ID + fiscal year) → Versions → Flavors.** Final approval is plan-level and
  covers all flavors in a version, but **review is tracked per flavor** (see per-flavor review below).
- Statuses: `draft, pending_review, approved, withdrawn, published`. 8 approval stages:
  `draft, first_review, second_review, executive_review, release_validation, document_generation,
  payout_system_config, completed`. `statusFromStage()` derives status from stage; **only one
  version per plan may be in the approval queue at a time** (`getPendingApprovalVersions`).
- **Approvals screen** (`renderApprovalScreen`) = a **"view as reviewer" toggle** + a reviewer-scoped
  **queue** + the version detail. Reviewers are named people mapped to stages (`REVIEWERS`,
  `currentReviewerId`): Susan L.→1st Review, Val R.→2nd Review, Elena M.→Executive, Comp Design Team→
  Release Validation/Document Generation/Payout config. The queue lists versions at the current
  reviewer's stage(s). At the **review stages** (`REVIEW_STAGES`) any reviewer can **Approve / Edit /
  Withdraw**; **Edit is in-place** (`reviewerEditVersion` → builder editable via `reviewerEditVersionId`,
  even though the version is locked). At **Release Validation** the Comp Design Team sets per-flavor
  new-hire guarantee + per-measure **plan-mechanics type** (`m.mechType` from `PLAN_MECH_TYPES`,
  `setReleaseValidationMechType`) then **Release to downstream** (`advanceVersionStage`, gated until all
  NHG + all mechType are set). Approve/withdraw/publish are attributed to `currentReviewer().name`.
- **Per-flavor review status (net requirement):** at the **review stages** the approval detail shows a
  per-flavor panel where a reviewer sets each flavor's `reviewStatus` (`pending`/`approved`/`needs_rework`)
  + a `reviewNote` (`setFlavorReviewStatus`/`setFlavorReviewNote`, `FLAVOR_REVIEW_LABELS`). **Plan status is
  system-derived** from the flavor statuses (`deriveFlavorReviewSummary` → `{approved,rework,pending,label}`),
  shown in the approval detail (signoff row), the reviewer **queue**, and the **proposal tracker** — never
  stored. **Final approval stays plan-level** but is **gated**: `advanceVersionStage` blocks leaving any
  `REVIEW_STAGES` stage until **every flavor is `approved`** (a `needs_rework` flavor holds the whole
  version in place — the author fixes it via in-place `reviewerEditVersion`, then the reviewer re-approves;
  no auto-bounce to draft). `submitVersionById` / `withdrawVersion` reset all flavor `reviewStatus` to
  `pending`.
- Payout **%VCT is stepped/marginal (tax-bracket): cumulative Σ(bracket width × xPCR), capped by
  the table Cap**; open-ended top tier shows `—`.
- Numeric-entry inputs: **VCT Wt%** (measure row) and **Cap** (payout) are entered as digits with a
  `%` adornment and stored as `"NN%"` (`onBuilderVctInput`/`onBuilderCapInput`); **Threshold** is
  optional, integer-only, stored as a number or `''` (blank). Cap is optional; seeded tables start
  with no cap/threshold.
- Payout tables are added independently (not derived from measures), **per flavor**, each with a free-text
  `title` (handlers `addBuilderPayoutTable(fi)` / `removeBuilderPayoutTable(slot)` / `setBuilderPayoutTitle(slot,…)`;
  rendered per flavor by `renderBuilderFlavorPayouts(fi)` into `#builder-flavor-${fi}-payouts`). The DOM slot
  **encodes the flavor**: `slot = payoutSlotFor(fi,i) = 1000 + fi*100 + i`, and `builderPayoutSlotIndex[slot]
  = {fi,idx}` (all payout handlers resolve `{fi,idx}` via `payoutTableForSlot`; `syncBuilderPayoutTablesFromDOM`
  writes each slot back to `flavors[fi].payoutTables[idx]`). Each has a **type**
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
- Setup nav has three Config screens — **Quota Band Sets**, **Attainment Tier Sets**, and **Performance
  Measures** — backed by `quotaBandSets` / `attainmentTierSets` / `performanceMeasures`, persisted together
  to localStorage (`compplan_config_sets_v4`; `performanceMeasures` is added gracefully to older stores).
- **Performance Measures** (`config-measures`, `renderConfigMeasures`, `renderMeasureSetCard`): each entry
  `{id,name,values:[str]}` — a measure name + its own value list. Feeds the measure row's **Measure**
  dropdown (options = names) and the dependent **Value** dropdown (options = the chosen measure's `values`).
  CRUD: `addPerformanceMeasure`/`deletePerformanceMeasure`/`setMeasureName`/`addMeasureValue`/
  `removeMeasureValue`/`setMeasureValue`; lookup `findPerformanceMeasure(name)`. On the builder row
  `onBuilderMeasureNameChange` resets the value when the measure changes.
- Quota Band / Attainment Tier sets are backed by `quotaBandSets` / `attainmentTierSets`, persisted to
  localStorage (`compplan_config_sets_v4`). Each
  set = `{id,name,bounds:[num]}` (band sets also carry a parallel `descriptions:[str]`); ranges
  auto-derive from the upper bounds. These feed the payout builder's set-driven construction, and a
  band's description shows on applied payout bands + in Compare. Range labels are **explicit about
  endpoints**, with **different conventions per set type**: **quota bands are `[prev, to)`** (inclusive
  lower, exclusive upper — the boundary value belongs to the next/higher band): first `< X`, middle
  `≥ X – < Y`, open top `≥ X` (`boundsRangeLabel(…,lowerInclusive=true)`, `quotaBandDesc`). **Attainment
  tiers are `(prev, to]`**: first `≤ X`, middle `> X – Y`, open top `> X` (`boundsRangeLabel(…,false)`,
  `formatAttainmentRange`).

## Conventions
- **Layout — flavor tabs + collapsible sections (reused across Builder / Approval / Compare).** Two
  shared components avoid stacking every flavor's tables in one long column: (1) a **flavor tab strip**
  (`flavorTabsHtml(flavors,activeIdx,prefix,{dotFn,subFn,addOnclick})` + `switchFlavorTab(prefix,idx)`,
  built on the existing `.pa-pivot` CSS; active index kept in `activeBuilderFlavor`/`activeApprovalFlavor`
  so re-renders preserve selection) and (2) a **collapsible section** (`collapsibleHtml(id,title,body,
  {open,badge})` + `toggleCollapse(id)`, `.pa-collapse`/`.pa-collapse.open>.pa-collapse-body`). **Builder**
  (`renderBuilderFlavors`): flavors are tab panels (`#builder-flavorpanel-${fi}`) — **all rendered into the
  DOM, inactive ones just hidden**, so per-flavor payout/bonus hydration + the flavor-encoded slot map keep
  working; within a flavor (`renderBuilderFlavorBlock`) Pay measures is open, Payout tables/Bonuses/Tether
  are collapsed. **Approval** (`renderApprovalScreen`): the per-flavor review + release-validation panels
  are tabbed (`prefix='approval'`, `#approval-flavorpanel-${fi}`). **Compare** (`buildCompareTableBodyHtml`):
  each Flavor section header row is a `.cmp-section-toggle` collapsing its sibling `<tr>`s
  (`toggleCompareSection`) — Plan information + first flavor open, rest collapsed by default.
- Builder is driven entirely by `version.content` via `builderWorkingContent`; edits persist on
  **Save** (`saveBuilderVersion`) — no auto-save. `validateVersionContent` (enforced only at
  **Submit**, not Save) requires plan name, per-flavor role, and 100% VCT per flavor.
- Compare is **compensation-plan / version level** (NOT flavor level): `buildPlanSnapshots` emits
  **one snapshot per version** (key `version|<versionId>`, no flavor), and each dropdown option is one
  plan version. It's **symmetric** (Plan 1 / Plan 2, differences highlighted only), has editable
  per-plan notes. `buildFieldsFromContentVersion(planId,fy,planMeta,flavors)` flattens a version into:
  plan-level (`planId`, `planName`); then **everything per-flavor** keyed by flavor label:
  `flavor${L}Role|HC|PayMixBase|PayMixVar|NHG|Measure${n}Name`(=description)`|Vct|PerfPd|PayFreq` +
  `flavor${L}MeasureCount`; `flavor${L}PayoutTable${n}Title/…/Data/Cap` + `flavor${L}PayoutCount`;
  `flavor${L}Bonus${n}Type/Details/Max` + `flavor${L}BonusCount`; `flavor${L}Tether`; plus `flavorLabels`.
  The grid groups rows into sections **Plan information → Flavor A/B/C** — payout tables, bonuses, and tether
  are **folded into each flavor's section** (no standalone Payout/Bonus/Tether sections). Each plan's payout
  table renders as its **own standalone table** in its column (`renderSinglePayoutTable`; the special-render
  branch matches `/[Pp]ayoutTable\d+$/`), no diff highlighting. `buildCompareFieldMeta(flavors)` is rebuilt
  per comparison from the union of flavor labels with per-flavor `{measureCount,payoutCount,bonusCount}`
  (`compareFlavorShape`). The per-plan **note** textareas
  persist to the version (`content.planMeta.designerNote`, bridged via `get/setSnapshotNote` ↔
  `get/setVersionNoteById`; round-trips with the builder plan-notes). A **"Send for approval"** button
  (`#compare-submit-btn` → `sendComparePlanForApproval`) auto-submits the editable (draft/withdrawn)
  selected side via `submitVersionById` (its validation + one-in-queue guards); shown only when both
  sides are selected and one is submittable.
- **Payout table display (read-only builder view + Compare) is a matrix** for `quota_band`/`target_pct`:
  **quota bands = columns, attainment tiers = rows, xPCR in cells, MCR row** (`renderPayoutMatrixTables`
  / `buildPayoutMatrixTable`, class `.payout-matrix`). Bands are **merged into one matrix when they share
  the same tier axis** (`payoutBandsShareTiers` — same `tierSetId` / same tier bounds), else rendered as
  **separate one-column tables**. `attainment_only` keeps its tier-list rendering. The **editable** builder
  keeps the per-band stacked editor (display-only change).
- **Role View** (nav "Role View", screen id still `multi`, `renderRoleView`) replaced the old hardcoded
  Multi-Role View: a data-driven table of **every version × flavor across all FYs**, sorted by flavor
  **role** (Role is a **column**, not a section header). Columns: Plan # / Plan Name / Role / Flavor / FY / Version / HC / Pay·Mix, then one
  **column group per performance measure** (Measure / VCT Wt% / Perf period / Payout freq, up to
  `MAX_BUILDER_MEASURES`), then Bonus (`roleBonusSummary` joins the **flavor's** bonus types) / Tether
  (read from **`fl.bonuses`/`fl.tether`** — per-flavor, since each row is already one flavor). Has a filter bar
  (Role / FY / Status / Owner / search) feeding the same `renderRoleView`; Role & Owner selects are
  populated dynamically (`fillRoleViewSelect`, selection-preserving). Renders lazily via
  `showScreen('multi')` + at bootstrap. Plan #/Plan Name cells link to `openPlanDetail(pk)` and the
  Version cell links to `openVersionInBuilder(versionId)`.
- Legacy `plans[]` array + `syncLegacyFromVersions` shims still back parts of some screens; keep
  them in sync on writes (`reconcilePlanFlavors`, publish/clone paths do this).
- There is `renderBuilderMeasureBlock`/`renderBuilderMeasures` + `populateBuilderFromSnapshot`/
  `populateBuilderFromDraft`/`applyBlankBuilderDefaults` legacy dead code (pre-`version.content`, references
  builder element ids that no longer exist — harmless no-ops); the live builder uses `renderBuilderFlavors`
  (which fills per-flavor `renderBuilderFlavorPayouts`/`renderBuilderFlavorBonuses`).
- **Seed/demo content** is authored in `SEED_CONTENT` (keyed by `planId|fy`, with plan-level `tether`/
  `bonuses`/`payouts()`/`linkByTitle` **cloned into each flavor** by `seedVersionContent` — `sc.payouts()` is
  called **once per flavor** so each flavor's tables get unique `payoutUid` ids) via config-set-driven payout
  builders (`seedQuotaBandTable` / `seedAttainmentOnlyTable` / `seedTargetPctTable`, which resolve
  `quotaBandSets`/`attainmentTierSets` by name so seeded tables carry real `bandSetId`/`tierSetId`). The final
  linking pass links each measure to a table **on its own flavor** and then **prunes each flavor's payout
  tables to only those a measure links to** (so per-flavor validation passes — a flavor only owns the tables
  it uses). Demo data obeys the rules: per-flavor measures total 100% VCT, all three payout types appear, some
  caps (`%`)/integer thresholds are set, and bonuses are seeded on A01 & S08. The old field maps
  (`fy26Baselines`/`fy27CurrentFields`/`planFieldTemplates`/`payMixMap`/`roleMap`) are no longer used by
  seeding but remain for the legacy `buildPlanFieldData` fallback.

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
