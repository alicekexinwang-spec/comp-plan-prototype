# CompPlan Studio prototype — progress

_Single file: `comp-plan-prototype.html`. Architecture/conventions live in `CLAUDE.md`; this file
tracks status only._

## Latest change — global persona toggle + stage capabilities + reject-to-draft (+ Overview2 tab) (branch `feature/comp-plan-flavorconfig`)
Two features in one commit.
- **Overview2 tab** (`renderPlanOverviewByFlavor`): a second builder tab (after Overview) showing the plan
  **by flavor dimension** — a grid of one card per flavor (header meta + that flavor's Measures & Weightings,
  own payout matrices, bonus, tether, key policies). Complements the section-first Overview.
- **Persona-based views:** a **global top-right "Viewing as" toggle** (`#topbar-persona`/`setPersona`) with
  personas **Creator / 1st / 2nd / Executive** (`REVIEWERS`; Comp dropped for now). **Editing is gated by
  persona** (`canEditVersion`): Creator edits drafts + submits (Create-Plan + builder Submit shown only to
  Creator); 1st/2nd edit in-place only at their stage; Exec never edits. **Approval actions are persona ×
  stage**: reviewer-at-own-stage → Approve→next (all-flavors-approved gated) · Edit (1st/2nd only) · **Reject
  → back to draft** (`rejectVersionToDraft` — keeps per-flavor statuses so the creator sees what to fix);
  off-stage → "Awaiting …". Resubmit restarts at 1st review (flavors reset to pending); advancing into a new
  review stage also resets flavors. Per-flavor "Needs rework" relabelled **Reject**. The in-approval reviewer
  bar was removed (topbar toggle is the single control).

Verified on webui2 (no console errors): toggle lists Creator/1st/2nd/Exec (default Creator); Create-Plan hides
for non-creators; 1st reviewer sees Approve/Edit/Reject + per-flavor panel, Exec sees Approve/Reject (no Edit);
approve 2 + reject 1 blocks advance; Reject → draft retains `[approved,approved,needs_rework]`; Exec can't edit
a draft, Creator can; resubmit → first_review + flavors pending; approve-all advances 1st→2nd with flavors
reset; Overview2 renders 3 flavor cards; all 11 versions `validateVersionContent`=0. **Committed on
`feature/comp-plan-flavorconfig`.**

## Earlier change — overview polish: first tab, no system-pay col, rename, consolidate, side-by-side (branch `feature/comp-plan-flavorconfig`)
Refined the Plan Overview (`renderPlanOverview`, builder + approval): (1) **Overview is now the first tab**
and the builder **lands on it** (`flavorTabsHtml` `leadTabs`; `openBuilder` sets `activeBuilderFlavor='overview'`);
(2) removed the **System pay measure** column from the overview measures table; (3) renamed **"Pay measures"
→ "Measures & Weightings"** in the overview AND the builder flavor block; (4) **consolidated** Bonus / Tether
/ Key policies — identical values across flavors collapse to one row labelled "All flavors", else split with
"Flavor A, B" labels (`consolidate(displayFn)`); (5) **Measures & Weightings + Payout tables side by side**
in a `.builder-mp-grid`.

Verified on webui2 (no console errors): Overview first + active on open; measures table has no system-pay
column; both section headers read "Measures & Weightings"; side-by-side grid (2 cols); consolidation shows
"All flavors" on A01 and splits to "Flavor A, C"/"Flavor B" when a flavor diverges; all 11 versions
`validateVersionContent`=0. **Committed on `feature/comp-plan-flavorconfig`.**

## Earlier change — uncollapse sections + prorate→release validation + plan overview (branch `feature/comp-plan-flavorconfig`)
- **Bonus / Tether / Key policies** in the flavor block are now **always-visible sections** (no collapse).
- **Prorate VCT moved to Release Validation.** Removed the builder payout-editor checkbox; `prorateVct` is
  now tri-state `''`/`'Yes'`/`'No'`, set per payout table at release validation (`setReleaseValidationProrate`)
  and **required** — `advanceVersionStage` blocks release until every table's prorate (+ NHG + mechType) is
  set (`rvFlavorDot` shows completeness). Builder Save preserves it via `syncBuilderPayoutTablesFromDOM`.
- **Plan Overview** (`renderPlanOverview(content)`): read-only all-flavors view — Pay measures (table per
  flavor), Payout tables **deduped** across flavors (`payoutOverviewKey`, "Used by: Flavor A/B/C" chip),
  Bonus/Tether/Key policies per flavor. Surfaced as a builder **Overview tab** + an approval-detail card.

Verified on webui2 (no console errors): 0 collapsibles / 0 prorate checkboxes in the flavor block; Overview
tab renders and dedupes A01's 6 tables→2 unique; Release Validation shows per-table prorate selects, the gate
blocks (message names prorate) then advances once all set, prorate persists + survives builder sync; all 11
versions `validateVersionContent`=0. **Committed on `feature/comp-plan-flavorconfig`.**

## Earlier change — flavor-level key policies + builder measures/payouts side-by-side (branch `feature/comp-plan-flavorconfig`)
- **Key policies → per flavor.** Removed `planMeta.keyPolicies` + the plan-level "Key Policies" section;
  each flavor owns `keyPolicies` (seed/blank/add-flavor/normalizer), edited in a **Key policies** collapsible
  in the flavor block. `planMeta` is now `{planName,designerNote}`.
- **Builder layout:** Pay measures + Payout tables now sit **side by side** in a 2-column grid
  (`.builder-mp-grid`, `auto-fit`/`minmax`; measures table in an `overflow-x:auto` scroller), both always
  visible; Bonuses/Tether/Key policies stay collapsible below. **Removed** the per-measure read-only payout
  matrix preview (payout tables are now in the adjacent column, linked via the row's dropdown).

Verified on webui2 (no console errors): side-by-side grid + column headers; 0 inline previews; payout mount
hydrates in the right column and across flavor tabs; per-flavor Key policies round-trips onto
`content.flavors[fi].keyPolicies`; `planMeta`=`{planName,designerNote}`; all 11 versions
`validateVersionContent`=0. **Committed on `feature/comp-plan-flavorconfig`.**

## Earlier change — Performance Measures config = value → system pay-measure map (branch `feature/comp-plan-flavorconfig`)
Reworked the Performance Measures config from "measure name → its own value list" to a **flat list**: each
entry `{id,value,systemPayMeasure}` maps one **performance-measure value** (editable, seeded with 17) to one
**system pay measure** from the **fixed** `SYSTEM_PAY_MEASURES` constant (29 options). Config screen is now a
single value+dropdown table (`renderConfigMeasures`; CRUD `setPerfMeasureValue`/`setPerfMeasureSystem`).
Builder measure row: the **Performance measure** dropdown lists the 17 values; picking one **auto-derives**
the row's read-only **System pay measure** (`systemPayMeasureFor`, stored on `m.value`). Compare label →
"M n · System pay measure". Storage key bumped `compplan_config_sets_v4` → `_v5` (shape change). Seed plan
measures keep their short names (legacy option, prepended). Removed dead handlers (`renderMeasureSetCard`
etc.).

Verified on webui2 (no console errors): 17 config rows w/ seeded mapping + 29-option dropdown; add/remap
persists to `_v5`; builder auto-fills system pay measure on select + save round-trip; all 11 versions
`validateVersionContent`=0; Compare label present. **Committed on `feature/comp-plan-flavorconfig`.**

## Earlier change — pay measures: config dropdowns + M# + inline linked payout (branch `feature/comp-plan-flavorconfig`)
- **Global config `performanceMeasures`** (new Setup → Performance Measures screen; `{id,name,values:[str]}`)
  persisted with the other config sets in `compplan_config_sets_v4` (added gracefully). CRUD mirrors the
  band-set pattern (`renderConfigMeasures`/`renderMeasureSetCard`, `addPerformanceMeasure`, etc.).
- **Measure model gains `value`.** The row's free-text name → a **Measure** dropdown (config names) + a
  **dependent Value** dropdown (that measure's values; resets on measure change via
  `onBuilderMeasureNameChange`). `value` optional (not submit-validated).
- **M1/M2/M3** sequence badges on measure rows; "Measure n" → "M n" in Role View + Compare.
- **Linked payout table shown read-only inline** under each measure row (`renderPayoutMatrixTables`).
- Downstream: Compare adds `flavor${L}Measure${n}Value` + "M n · Value" rows; Role View appends value to
  the measure cell + search.

Verified on webui2 (no console errors): config seeded (6) + add/value persists across reload; builder
dependent dropdowns + reset + inline matrix; all 11 versions `validateVersionContent`=0; save round-trips
`value`; Compare/Role View labels. **Committed on `feature/comp-plan-flavorconfig`.**

## Earlier change — scannable layout: flavor tabs + collapsible sections (branch `feature/comp-plan-flavorconfig`)
Replaced the "all flavors + all tables stacked in one long column" layout with two reusable components
applied across the three key screens (Power-Apps-canvas style: simple, flat, limited nesting):
- **Shared:** `flavorTabsHtml`/`switchFlavorTab` (on existing `.pa-pivot`) + `collapsibleHtml`/`toggleCollapse`
  (new `.pa-collapse`). Active flavor kept in `activeBuilderFlavor`/`activeApprovalFlavor`.
- **Builder:** flavors are now **tabs** (one at a time; all panels stay in the DOM hidden so payout-slot
  hydration is untouched); within a flavor, Pay measures open, Payout/Bonus/Tether collapsed. "Add flavor"
  moved into the tab strip.
- **Approval:** per-flavor review + release-validation panels are tabbed per flavor.
- **Compare:** each Flavor section is a collapsible row-group (`toggleCompareSection`); Plan info + first
  flavor open, rest collapsed.

Display/layout only — no data-model or logic change. Verified on webui2 (no console errors): builder 3
tabs + collapsibles, cross-flavor payout edits persist, validate=0; approval review tabbed, derived
summary updates on approve; compare sections collapse/expand, diff counts intact. **Committed on
`feature/comp-plan-flavorconfig`.**

## Earlier change — flavor-owned config + per-flavor review status (branch `feature/comp-plan-flavorconfig`)
Re-based this branch onto HEAD `1f40271`, then two phases:
- **Phase 1 — full per-flavor ownership.** Moved `bonuses`, `tether`, and `payoutTables` from
  `planMeta`/version-level down into each **flavor** (`content.payoutTables` and `planMeta.bonuses`/`.tether`
  removed; `planMeta` = `{planName,keyPolicies,designerNote}`). Measures link to a payout table **on their own
  flavor**. Builder renders per-flavor payout/bonus/tether editors with a **flavor-encoded slot map**
  (`payoutSlotFor(fi,i)=1000+fi*100+i`, `builderPayoutSlotIndex[slot]={fi,idx}`; new
  `renderBuilderFlavorPayouts`/`renderBuilderFlavorBonuses`, `onBuilderFlavorTether`). `validateVersionContent`
  checks measure↔table linking **per flavor**. Compare folds payout/bonus/tether into each Flavor section
  (`buildFieldsFromContentVersion` per-flavor keys `flavor${L}PayoutTable${n}*`/`Bonus${n}*`/`Tether`;
  `buildCompareFieldMeta(flavors)`; `compareFlavorShape` carries payout/bonus counts). Role View reads
  `fl.bonuses`/`fl.tether`. Seed clones plan-level config into each flavor (unique payout ids) and **prunes
  each flavor's tables to only those its measures use** (so per-flavor validation passes).
- **Phase 2 — per-flavor review + derived plan status.** Each flavor has `reviewStatus`
  (`pending`/`approved`/`needs_rework`) + `reviewNote`. Review stages show a per-flavor Approve / Needs-rework
  panel (`setFlavorReviewStatus`/`setFlavorReviewNote`). Plan status is **derived** (`deriveFlavorReviewSummary`)
  and shown in the approval detail, reviewer queue, and proposal tracker. `advanceVersionStage` is **gated** at
  review stages until every flavor is approved; `submitVersionById`/`withdrawVersion` reset statuses to pending.

Verified on webui2 (no console errors): all 11 seeded versions `validateVersionContent()==[]`; builder
per-flavor add/edit/remove + save round-trip; Compare renders per-flavor sections + matrices; Role View
per-flavor bonus/tether; review panel + gate (blocks on needs-rework/pending, advances when all approved) +
submit/withdraw reset. See `CLAUDE.md` for the updated model. **Committed on `feature/comp-plan-flavorconfig`
(off `master`); not on PR #1.**

## Earlier change — reviewer-scoped approval queue + view-as toggle + release validation (committed 1f40271)
Approvals screen rebuilt: a **"view as reviewer" toggle** (`REVIEWERS` — Susan L./Val R./Elena M./Comp
Design Team, each mapped to stages; `currentReviewerId`) + a **reviewer-scoped queue** (versions at that
reviewer's stage) + the version detail. Review stages (1st/2nd/Exec) allow **Approve / Edit / Withdraw**;
**Edit is in-place** (`reviewerEditVersion` unlocks the builder for the pending version via
`reviewerEditVersionId`; `openBuilder`/`saveBuilderVersion` honor it). **Release Validation** (Comp Design
Team) captures per-flavor new-hire guarantee + per-measure **mechanics type** (`m.mechType` from
`PLAN_MECH_TYPES`, `setReleaseValidationMechType`); **Release to downstream** (`advanceVersionStage`) is
gated until all NHG + mechType set. Actor attribution uses `currentReviewer().name`. Verified on webui2
(no console errors): toggle/queue counts, review actions, edit-in-place persists (version stays
pending_review), release gate blocks then advances (→ Document Generation, attributed to Comp Design Team).
**Not committed.**

## Earlier change — quota-band range convention + Role View flavor column (committed bca1341)
1. **Quota bands are now `[prev, to)`** (inclusive lower, exclusive upper — the boundary value belongs
   to the next/higher band): `< 1.5` / `≥ 1.5 – < 3.5` / `≥ 3.5`. Added a `lowerInclusive` param to
   `boundsRangeLabel` (passed true from the band config card + live update + `onQuotaBandSetChange` +
   `seedQuotaBandTable`); updated `quotaBandDesc` (`<`/`≥ – <`/`≥`). Attainment tiers keep `(prev, to]`
   (`formatAttainmentRange` + tier config unchanged).
2. **Role View**: added a **Flavor** column (A/B/C) after Role.
Verified on webui2 (no console errors): band formatter/desc + applied band labels use the new convention,
tiers unchanged, Role View shows the Flavor column. (Config-set descriptions persist in localStorage —
clear `compplan_config_sets_v4` for regenerated defaults.) **Not committed.**

## Earlier change — explicit range endpoints (uncommitted)
Quota-band / attainment-tier range labels now make endpoint inclusion explicit (bands/tiers are
`(prev, to]`): first `≤ X`, middle `> X – Y`, open top `> X`. Updated `boundsRangeLabel` (config set
screens + applied band labels), `formatAttainmentRange` (payout summary / matrix / Compare), and the
inline range in `syncPayoutVctCalcs` (editable builder tier table). Display-only. Verified on webui2 (no
console errors): formatters + Compare matrix rows read `≤ 100%` / `> 100% – 150%` / `> 150%`. **Not committed.**

## Earlier change — pay measure → payout table link + submit validation (uncommitted)
Each pay measure now has a **"Payout table" dropdown** on its measure row (stores `m.payoutTableId`;
options = the version's payout tables). A measure links to one table; a table can be linked by many
measures. **Submit** (`validateVersionContent`) now requires every measure to link to an existing table
and every table to have ≥1 measure. Adding/removing a payout table refreshes the dropdowns;
removing a table clears measures that pointed to it. Seed: `linkByTitle` on multi-table plans (A01,
B01·FY27, S08·FY27) + a final pass sets each measure's `payoutTableId`. Verified on webui2 (no console
errors): all seeded versions valid (validate = []), A01 maps correctly, dropdown selection works,
read-only disabled, both validation errors fire. **Not committed.**

## Earlier change — Compare: persist notes + send for approval (uncommitted)
- **Plan notes** in Compare now persist to the version (`content.planMeta.designerNote`): bridged the
  existing `get/setSnapshotNote` to new `get/setVersionNoteById`, so note edits save on the version and
  round-trip with the builder (`loadBuilderPlanNote`/`saveBuilderPlanNote`; `saveBuilderVersion` +
  `submitCurrentVersion` preserve `designerNote`).
- **Send for approval**: added `#compare-submit-btn` + `sendComparePlanForApproval` — auto-detects the
  editable (draft/withdrawn) selected side and submits via the guarded `submitVersionById`
  (validation + one-in-queue). Shown only when both sides selected and one is submittable.
- Verified on webui2 (no errors): note edit → `getVersion(...).content.planMeta.designerNote`, shows in
  compare box + builder; M01 draft → submit → `pending_review`/`first_review`, button hides; A01 draft →
  queue-guard alert (A01 v2 already pending); two published → no button. **Not committed.**

## Earlier change — Role View: clickable plan/version (uncommitted)
Role View rows are now drill-downs: **Plan #** and **Plan Name** link to `openPlanDetail(pk)` (plan
detail screen); the **Version** label links to `openVersionInBuilder(versionId)` (version in the builder,
read-only if locked). Added `pk`/`versionId` to each row + a `.rv-link` style. Verified on webui2 (no
errors): links carry the right handlers; invoking them opens `screen-plan` / `screen-builder`.
**Not committed.**

## Earlier change — Role View: Role as a column (uncommitted)
Changed Role View so **Role is a regular column** (Plan # / Plan Name / Role / FY / Version / HC /
Pay·Mix / measure groups / Bonus / Tether) instead of a full-width section-header row; rows are flat but
still sorted by role so same-role rows stay adjacent. Removed the section-row logic + `.roleview-group`
CSS. Verified on webui2 (no errors): 0 group rows, 22 flat rows, Role column populated, Role filter →
4 Solution Architect rows. **Not committed.**

## Earlier change — Role View filters (uncommitted)
Added a filter bar to the Role View: **Role / FY / Status / Owner / search**, all feeding
`renderRoleView`. Role & Owner selects are populated dynamically from the data (`fillRoleViewSelect`,
preserves current selection); FY/Status are static. Rows filtered before grouping; `maxN` and subtitle
reflect the filtered set; empty combo shows "No plans match filters". Verified on webui2 (no errors):
role→1 group/4 rows, FY26→4 rows, Published→10 rows, search "renewal"→20 rows, selection preserved.
**Not committed** (stacks on the Role View change below).

## Earlier change — Role View (uncommitted)
Replaced the hardcoded "Multi-Role View" with a data-driven **Role View** (`renderRoleView`; nav label
"Role View", screen id still `multi`):
- Lists **every version × flavor across FY26 + FY27**, grouped by flavor **role** (section header rows).
- Columns: Plan # / Plan Name / FY / Version / HC / Pay·Mix, then a **column group per performance
  measure** (Measure / VCT Wt% / Perf period / Payout freq, up to 4), then Bonus (`roleBonusSummary`) /
  Tether. Two-row header (measure super-headers colspan-4); wide table scrolls horizontally.
- Removed `multiData`/`buildMulti`; wired `renderRoleView` into bootstrap + `showScreen('multi')`.
- Verified on webui2 (no console errors): 12 role groups, 22 plan versions; Solution Architect shows A01
  FY26 v1 + FY27 v1/v2/v3 with per-version VCT (50/55/60%), Linearity bonus, tether values. **Not committed.**

## Earlier change — payout table as a matrix (display) (committed 20fa100)
Display-only: read-only builder view + Compare now render `quota_band`/`target_pct` payout tables as the
classic **matrix** — quota bands = columns, attainment tiers = rows, xPCR cells, MCR row.
- New helpers `payoutBandsShareTiers` / `buildPayoutMatrixTable` / `renderPayoutMatrixTables`; bands
  **merge into one matrix when they share a tier set**, else render as **separate one-column tables**.
- Wired into `renderSinglePayoutTable` (Compare) and `renderPayoutTableEditor` (read-only branch only);
  editable builder editor unchanged. `attainment_only` unchanged. Added `.payout-matrix` CSS.
- Verified on webui2 (no console errors): read-only B01 quota_band → merged matrix (Quota Size × 3 bands,
  tier rows, xPCR); Compare A01 v1/v2 → 4 matrices with MCR row (7%); S08 target_pct (differing tier sets)
  → separate stacked tables; editable A01 draft still shows per-band editor. **Not committed.**

## Earlier change — payout-table layout tightened (uncommitted)
Layout-only polish (no model/logic change) on both payout surfaces:
- **Builder tier table**: dropped the wasted empty 5th column (now 4 cols: Upper bound % / Range /
  xPCR / %VCT|VCT-at-max, ~299px), recomputed scoped column widths, and trimmed paddings/margins
  (`.payout-tier-table td` 3px 6px, `.payout-threshold-block`/`-hdr`, `.payout-options-bar`).
- **Compare table** (`renderSinglePayoutTable`): band label now shown **once per band via `rowspan`**
  (spanning its tiers + MCR row) instead of repeating on every row; added `<colgroup>` widths +
  `cp-band`/`cp-att`/`cp-xpcr` cell classes (rowspan-safe, not `nth-child`).
- Verified on webui2 (no console errors): builder shows 4-col tier tables (3 bands, readonly ok);
  Compare A01 v1 vs v2 shows each band once with matching rowspans and aligned columns. **Not committed.**

## Earlier change — refreshed dummy/seed data (uncommitted)
Rewrote `seedVersionContent` to author demo content from a new `SEED_CONTENT` map (keyed by
`planId|fy`) instead of the legacy field-map pipeline, so the demo now matches the new model:
- Added config-set-driven payout builders `seedQuotaBandTable` / `seedAttainmentOnlyTable` /
  `seedTargetPctTable` (resolve `quotaBandSets`/`attainmentTierSets` by name → real `bandSetId`/`tierSetId`).
- Every flavor's measures total **100% VCT**; realistic measure descriptions; all three payout **types**
  present (quota_band ×14, target_pct ×1, attainment_only ×2); caps (200/250/150%) + integer thresholds
  (50/60) on several tables; **bonuses** seeded on A01 (Linearity) and S08 (M1&M2 + Lead Referral).
- FY26 baselines and A01 v1/v2/v3 differ (measures/tether/top-tier xPCR) so Compare shows real diffs.
- Legacy field maps left in place (still used by `buildPlanFieldData` fallback).
- Verified on webui2 (no console errors): 0 VCT-sum failures, 0 payout id/title/bandSetId/tierSetId
  issues, builder renders A01 (3 flavors @100%, titled tables, cap/threshold, 1 bonus), submit
  validation passes, Compare renders per-flavor + payout + bonus sections. **Not committed.**

## Earlier change — Compare payout tables shown per-plan (uncommitted)
In Compare, each `Payout table N` row now renders **each plan's payout table as its own standalone
table** in its column (`renderSinglePayoutTable`: Band/Attainment/xPCR) instead of one merged
Plan1/Plan2 table; **no diff highlighting** on the payout tables. Removed the now-unused merged
`renderComparePayoutTable`. Verified on webui2 (no errors): 2 tables per payout row, no colspan, no
diff cells. **Not committed** (stacks on the plan-level Compare change, also uncommitted).

## Earlier change — Compare at compensation-plan level (uncommitted)
Compare now operates at the **plan/version level**, not the flavor level:
- `buildPlanSnapshots` emits **one snapshot per version** (key `version|<versionId>`, no flavor); each
  dropdown option is one plan version (no "Flavor A/B/C" text).
- `buildFieldsFromContentFlavor` → **`buildFieldsFromContentVersion`**: plan-level fields once
  (`planId`, `planName`, `tether`, payout tables, bonuses) + **per-flavor** keys `flavor${L}…`
  (role/HC/paymix/NHG/measures) + `flavorLabels`.
- `buildCompareFieldMeta(payoutCount,bonusCount,flavors)` adds a **Flavor A/B/C** section per flavor
  (union across both sides via `compareFlavorShape`); grid order: Plan information → Flavor sections →
  Payout table N → Bonus N → Tether. `COMPARE_BASE_META` trimmed (HC/paymix/role/NHG moved per-flavor).
- HC delta keys off `meta.hc`; change summary prefixes flavor/payout/bonus labels with their section.
  Copy "plan flavor" → "plan/plan version". Presets (already `version|<id>`) now resolve exactly.
- Verified on webui2 (no console errors): 11 per-version snapshots, dropdown has no "Flavor", grid
  sections correct, unequal flavor sets (A vs A/B/C) render without crash, builder preset works. **Not
  committed.**

## Earlier change — builder input formatting (committed 73562e7)
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
