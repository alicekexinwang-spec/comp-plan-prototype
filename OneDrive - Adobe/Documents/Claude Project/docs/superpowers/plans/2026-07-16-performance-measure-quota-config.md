# Performance Measure Quota Configuration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove reusable Payout Table Setups, move quota settings onto Performance Measure configuration, and restrict proposal payout-table editing to tier sets, xPCR, MCR, and the existing note.

**Architecture:** Extend reusable and proposal performance-measure records with snapshot quota fields. Generate payout-table band rows from the linked measure's configured quota-band set, while keeping tier data and notes on the payout table. Preserve existing one-to-one measure/table linkage and adapt read-only rendering and comparison to combine measure quota metadata with payout tier data.

**Tech Stack:** Single-file HTML/CSS/JavaScript prototype, browser localStorage, Chart.js and Tabler Icons from CDN; no build system or automated test framework.

## Global Constraints

- Minimal code changes.
- Reuse existing UI/components/styles.
- No unrelated refactoring.
- Preserve existing functionality.
- Keep the existing optional payout-table note editable.
- xPCR remains per attainment tier; MCR remains per quota band.
- Verification uses static JavaScript checks and browser regression because the approved prototype workflow has no automated test harness.

---

### Task 1: Move quota ownership into Performance Measure configuration

**Files:**
- Modify: `comp-plan-prototype.html` configuration markup/state around the Setup navigation, `CONFIG_SETS_SEED`, `seedConfigSets`, and `renderConfigMeasures`

**Interfaces:**
- Produces: normalized performance measure `{id,value,systemPayMeasure,bandSetId,payCurveType,threshold,globalVctCap}`
- Removes: `payoutSetups`, `renderConfigPayouts`, Payout Tables screen/nav/CRUD and config-scoped payout slots

- [ ] **Step 1: Record the failing structural checks**

Run:

```powershell
rg -n "config-payouts|payoutSetups|renderConfigPayouts|Payout table setup" comp-plan-prototype.html
rg -n "setPerfMeasureBandSet|setPerfMeasureCurve|setPerfMeasureThreshold|setPerfMeasureCap" comp-plan-prototype.html
```

Expected: the first command returns current standalone-library references; the second returns no handlers.

- [ ] **Step 2: Remove standalone configuration registration and state**

Remove the Payout Tables sidebar item, `screen-config-payouts`, screen registry entry, show-screen dispatch, `payoutSetups`, payout setup normalization/seeding/lookups, and CRUD/editor functions. Persist only:

```js
function persistConfigSets(){
  try{localStorage.setItem(CONFIG_SETS_STORAGE_KEY,JSON.stringify({quotaBandSets,attainmentTierSets,performanceMeasures}));}catch(e){}
}
```

Normalize legacy entries with:

```js
const normMeasure=s=>({
  id:s.id||payoutUid(),
  value:s.value||'',
  systemPayMeasure:s.systemPayMeasure||'',
  bandSetId:s.bandSetId||'',
  payCurveType:s.payCurveType||'Linear',
  threshold:(s.threshold!=null?s.threshold:''),
  globalVctCap:s.globalVctCap||''
});
```

- [ ] **Step 3: Add quota controls to each Performance Measure configuration row**

Reuse `.form-select`, `.form-input`, `.xpcr-cell`, `.pct-suffix`, and the existing band/curve option generation. Add columns/controls for Quota Band Set, Pay Curve, Threshold, and Cap, wired to:

```js
function setPerfMeasureBandSet(id,v){const m=findMeasureById(id);if(m){m.bandSetId=v;persistConfigSets();}}
function setPerfMeasureCurve(id,v){const m=findMeasureById(id);if(m){m.payCurveType=v||'Linear';persistConfigSets();}}
function setPerfMeasureThreshold(id,v){const m=findMeasureById(id);if(m){m.threshold=v===''?'':parseInt(v,10);persistConfigSets();}}
function setPerfMeasureCap(id,v){const m=findMeasureById(id);if(m){m.globalVctCap=v?String(v).replace(/%/g,'')+'%':'';persistConfigSets();}}
```

New rows default to:

```js
{id:payoutUid(),value:'',systemPayMeasure:'',bandSetId:'',payCurveType:'Linear',threshold:'',globalVctCap:''}
```

- [ ] **Step 4: Verify configuration ownership**

Run the structural checks again.

Expected: zero standalone-library references; all four Performance Measure handlers exist; `persistConfigSets` includes no `payoutSetups`.

- [ ] **Step 5: Commit**

```powershell
git add -- comp-plan-prototype.html
git commit -m "feat: move quota setup to performance measures"
```

### Task 2: Apply measure quota snapshots and simplify the payout editor

**Files:**
- Modify: `comp-plan-prototype.html` builder measure handlers, payout-table rendering/collection, synchronization, and validation

**Interfaces:**
- Consumes: normalized reusable Performance Measure configuration from Task 1
- Produces: proposal measure quota snapshot fields and payout table `{id,title,note,prorateVct,thresholds}`

- [ ] **Step 1: Record failing ownership checks**

Run:

```powershell
rg -n "sourceSetupId|sourceSetupName|onPayoutSetupRefChange|builder-p.*-(bandset|curve|threshold|cap)" comp-plan-prototype.html
rg -n "applyPerformanceMeasureQuota|quotaConfigForMeasure" comp-plan-prototype.html
```

Expected: obsolete setup and payout-level quota controls exist; new measure helpers do not.

- [ ] **Step 2: Add measure quota helpers**

Add focused helpers near `onBuilderMeasureNameChange`:

```js
function quotaConfigForMeasure(value){
  const c=findPerfMeasureByValue(value)||{};
  return{bandSetId:c.bandSetId||'',payCurveType:c.payCurveType||'Linear',threshold:(c.threshold??''),globalVctCap:c.globalVctCap||''};
}
function buildQuotaBands(setId){
  const set=findBandSet(setId);let prev=0;
  return ((set&&set.bounds)||[]).map((b,i)=>{
    const to=(b===''||b==null)?null:Number(b);
    const band={id:payoutUid(),label:boundsRangeLabel(prev,to,'',true),description:(set.descriptions&&set.descriptions[i])||'',tierSetId:'',mcr:null,tiers:[]};
    if(to!=null)prev=to;
    return band;
  });
}
function applyPerformanceMeasureQuota(m,pt,value){
  const c=quotaConfigForMeasure(value);
  Object.assign(m,c);
  pt.thresholds=buildQuotaBands(c.bandSetId);
}
```

Update `onBuilderMeasureNameChange` to synchronize current payout DOM first, set `description` and `value`, apply the selected configuration to the measure and linked table, preserve table identity/title/note/prorate, then re-render the flavor.

- [ ] **Step 3: Simplify builder payout rendering**

Remove setup selection, provenance text, and editable band/curve/threshold/cap controls from `renderBuilderFlavorPayouts`. Keep the existing header/title, `#builder-p{slot}-table` tier editor, and `#builder-p{slot}-note` textarea. Show the measure-owned quota settings as non-editable context only if necessary for clarity, using `.pa-val`; do not create duplicated payout inputs.

Make `renderPayoutThresholdBlock` always use the existing editable Attainment Tier Set selector when the proposal is editable. Retain xPCR and MCR inputs and read-only behavior.

- [ ] **Step 4: Restrict payout collection and synchronization**

Make `collectPayoutTable` return only:

```js
return{
  title:document.getElementById(`builder-p${slot}-title`)?.value||'',
  thresholds,
  note:document.getElementById(`builder-p${slot}-note`)?.value||'',
  prorateVct:''
};
```

In `syncBuilderPayoutTablesFromDOM`, preserve `id` and existing `prorateVct`. Remove setup provenance and payout-level quota preservation. Calculate `%VCT` using the linked measure's `globalVctCap` rather than a payout cap input.

- [ ] **Step 5: Add submit validation for measure quota ownership**

Within the existing per-measure validation loop, add a clear error when a selected measure has no `bandSetId`. Retain all existing plan-name, role, VCT, maximum-measure, and linkage validation.

- [ ] **Step 6: Verify syntax and ownership**

Extract the inline script and run `node --check` on it. Repeat the ownership searches.

Expected: JavaScript syntax passes; setup provenance and payout-level quota inputs have zero live-path references; measure quota helpers exist.

- [ ] **Step 7: Commit**

```powershell
git add -- comp-plan-prototype.html
git commit -m "feat: simplify payout tables to tier details"
```

### Task 3: Adapt seeds, read-only rendering, and comparison

**Files:**
- Modify: `comp-plan-prototype.html` seed builders/final pass, Overview deduplication, payout rendering, and compare field generation

**Interfaces:**
- Consumes: measure-owned quota fields and payout-owned tier details from Task 2
- Produces: valid seeded proposal snapshots and unchanged user-facing matrices/comparison semantics

- [ ] **Step 1: Replace library-based seed copying**

Keep existing seed table factories as internal demo builders, but split their output so quota fields are copied onto each seeded measure and tier/note/prorate data remains on its linked payout table. Remove `findPayoutSetupByName`, `seedPayoutLibrary`, provenance stamping, and `payoutSetups` lookup.

For each seeded measure/table pair:

```js
m.bandSetId=demo.bandSetId||'';
m.payCurveType=demo.payCurveType||'Linear';
m.threshold=(demo.threshold??'');
m.globalVctCap=demo.globalVctCap||'';
const copy={id:payoutUid(),title:m.description||'',note:demo.note||'',prorateVct:'',thresholds:cloneFreshThresholds(demo.thresholds||[])};
m.payoutTableId=copy.id;
```

- [ ] **Step 2: Combine measure quota metadata with payout tier data for display**

Add:

```js
function payoutWithMeasureQuota(pt,m){
  return Object.assign({},pt||{},quotaConfigForSnapshot(m));
}
```

where `quotaConfigForSnapshot(m)` reads the four quota fields directly from the proposal measure. Use the combined view in Overview, Overview2, approval/read-only payout rendering, `payoutOverviewKey`, and comparison serialization/cap fields. Do not write the combined fields back into payout tables.

- [ ] **Step 3: Preserve legacy proposal compatibility**

When reading a proposal created before this change, migrate quota fields from its linked payout table onto its measure if the measure fields are absent. This normalization happens in the existing content seeding/normalization path and does not mutate published source data outside the in-memory prototype.

- [ ] **Step 4: Verify seeds and static references**

Run browser evaluation after clearing `compplan_config_sets_v6`:

```js
planVersions.map(v=>({id:v.versionId,errors:validateVersionContent(v.content)}))
```

Expected: every `errors` array is empty.

Search for obsolete symbols:

```powershell
rg -n "payoutSetups|findPayoutSetup|sourceSetupId|sourceSetupName|config-payouts|onPayoutSetupRefChange" comp-plan-prototype.html
```

Expected: zero results.

- [ ] **Step 5: Commit**

```powershell
git add -- comp-plan-prototype.html
git commit -m "fix: preserve payout views with measure quota data"
```

### Task 4: Update documentation and run full regression

**Files:**
- Modify: `CLAUDE.md`
- Modify: `docs/progress.md`
- Modify: `docs/prd-comp-plan.md`
- Modify: `docs/PRD-Compensation-Plan.docx`

**Interfaces:**
- Documents final ownership and verification evidence

- [ ] **Step 1: Update architecture and progress documentation**

Document the final shapes and flow exactly:

```text
Performance Measure configuration owns value, system pay measure, quota band set, pay curve, threshold, and cap.
Proposal measure snapshots those quota fields.
Payout table owns generated bands' attainment tier selections, per-tier xPCR, per-band MCR, and optional note.
```

Remove reusable Payout Table Setup and provenance language.

- [ ] **Step 2: Update and regenerate the PRD artifact**

Update the Markdown PRD requirements and workflows to match the final ownership. Regenerate the tracked `.docx` using the repository document workflow and visually inspect every rendered page before accepting it.

- [ ] **Step 3: Run browser regression**

Verify:

1. No standalone Payout Tables setup screen or navigation.
2. Performance Measure config shows all six fields and persists them.
3. Selecting a measure generates its configured quota bands.
4. Payout editor shows Attainment Tier Set, xPCR, MCR, and optional note only.
5. Tier selection and xPCR/MCR/note survive save and reopen.
6. Existing proposal snapshots do not change when reusable configuration changes.
7. Overview, approval, and comparison matrices render.
8. All seeded versions validate and the browser console has no errors.

- [ ] **Step 4: Run final repository checks**

Run:

```powershell
git diff --check
git status --short
git log --oneline -6
```

Expected: no whitespace errors; only intended documentation changes remain before the final commit.

- [ ] **Step 5: Commit**

```powershell
git add -- CLAUDE.md docs/progress.md docs/prd-comp-plan.md docs/PRD-Compensation-Plan.docx
git commit -m "docs: align payout ownership with measure config"
```

## Self-Review

- Spec coverage: standalone setup removal, Performance Measure quota fields, payout tier-only ownership, optional note, compatibility, and no broken references are each covered.
- Placeholder scan: no incomplete implementation placeholders.
- Type consistency: reusable and proposal measure quota field names are identical; payout table retains `thresholds`, `note`, and `prorateVct`; combined display objects expose the legacy renderer fields without persisting them on payout tables.
