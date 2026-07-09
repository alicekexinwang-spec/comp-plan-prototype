# Option B — Plan/Version Model Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Flatten the three-tier model (`compPlans + proposals + publishedRevisions`) into a clean two-tier model (`plans + versions`) in `comp-plan-prototype.html`.

**Architecture:** Replace three data arrays with two: `plans` (unique by `planId+fy`, no `versionId`) and `versions` (design alternatives, lifecycle via `status` field). The `releasedVersionId` pointer on each plan replaces `currentPublishedRevisionId`. All UI renaming: Proposals → Versions, Published Revision → Released Version.

**Tech Stack:** Single-file HTML/JS prototype, no build step, no test framework — verification is manual browser testing.

## Global Constraints

- File: `comp-plan-prototype.html` only. No new files.
- Snapshot keys change from `revision|rev-...` / `proposal|prop-...` to `version|ver-...`. Any hardcoded preset keys must be updated.
- New `versions` array replaces BOTH `proposals` AND `publishedRevisions`. No separate revision array survives.
- New `plans` array replaces BOTH `compPlans` AND the old `plans` catalog. Uses `planId` field (not `id`).
- Version lifecycle: `draft` → `in-review` → `released` | `rejected` | `archived`.
- `versionId` at plan level: removed entirely. The "Version" cell in the builder summary bar now shows the version's `versionNum` (1, 2, 3…) not a plan-level V1/V2.
- Do not change: CSS, payout tables, plan mechanics screen, compare engine logic, chart.js usage, or any screen not listed in the tasks.

---

### Task 1: Replace data arrays and global variables

**Files:**
- Modify: `comp-plan-prototype.html:1377-1444` (global vars, data arrays)

**What changes:**
- Delete `compPlans` array and `plans` catalog array
- Delete `publishedRevisions` array and `proposals` array
- Add new `plans` array (merged, `planId` field, `releasedVersionId`, no `versionId`)
- Add new `versions` array (replaces both proposals + publishedRevisions)
- `currentProposalId` → `currentVersionId`
- `approvalProposalId` → `approvalVersionId`
- `PROPOSAL_STATUS_LABELS` → `VERSION_STATUS_LABELS` (new statuses)
- `auditLog` entries: `proposalId`/`revisionId` → `versionId`

- [ ] **Step 1: Replace the global variable block and data arrays**

Replace lines 1377–1444 in the file with the following block:

```js
let currentPlanKey='H01.A|FY27';
let currentVersionId=null;
let approvalVersionId='ver-H01.A-FY27-2';

const VERSION_STATUS_LABELS={
  draft:'Draft','in-review':'In Review',released:'Released',
  rejected:'Rejected',archived:'Archived'
};
const VALID_FYS=['FY26','FY27'];

function planKey(planId,fy){return `${planId}|${fy}`;}
function parsePlanKey(key){
  if(!key)return {planId:'',fy:'FY27'};
  const i=String(key).lastIndexOf('|');
  if(i<0)return {planId:key,fy:'FY27'};
  return {planId:key.slice(0,i),fy:key.slice(i+1)};
}
function planExists(planId,fy){return !!plans.find(p=>p.planId===planId&&p.fy===fy);}

const plans=[
  {planId:'H01.A',name:'Digital Experience AE',fy:'FY27',role:'Digital Experience Account Executive',owner:'Melissa C.',releasedVersionId:null,updated:'Jun 11'},
  {planId:'H02.A',name:'Hybrid CXO & C&P Sales Leader',fy:'FY27',role:'CXO & C&P Hybrid Sales Leader A',owner:'Melissa C.',releasedVersionId:'ver-H02.A-FY27-1',updated:'Jun 10'},
  {planId:'H03.A',name:'Commerce AE',fy:'FY27',role:'Commerce Account Executive',owner:'Val R.',releasedVersionId:'ver-H03.A-FY27-1',updated:'Jun 9'},
  {planId:'H04.A',name:'SMB Account Executive',fy:'FY27',role:'SMB Account Executive',owner:'Val R.',releasedVersionId:null,updated:'Jun 8'},
  {planId:'H05.A',name:'Enterprise AE',fy:'FY27',role:'Enterprise Account Executive',owner:'Melissa C.',releasedVersionId:'ver-H05.A-FY27-1',updated:'Jun 7'},
  {planId:'H06.A',name:'SMB Renewal Specialist',fy:'FY27',role:'SMB Renewal Specialist',owner:'Val R.',releasedVersionId:'ver-H06.A-FY27-1',updated:'Jun 5'},
  {planId:'H07.A',name:'Partner Channel AE',fy:'FY27',role:'Partner Channel AE',owner:'Melissa C.',releasedVersionId:null,updated:'Jun 4'},
  {planId:'H08.A',name:'Strategic Named Account',fy:'FY27',role:'Strategic Named Account AE',owner:'Val R.',releasedVersionId:null,updated:'Jun 3'},
  {planId:'H09.A',name:'EMEA Enterprise AE',fy:'FY27',role:'EMEA Enterprise AE',owner:'Melissa C.',releasedVersionId:null,updated:'Jun 2'},
  {planId:'H10.A',name:'Emerging Markets AE',fy:'FY27',role:'Emerging Markets AE',owner:'Val R.',releasedVersionId:null,updated:'Jun 1'},
  {planId:'H01.A',name:'Digital Experience AE',fy:'FY26',role:'Digital Experience Account Executive',owner:'Melissa C.',releasedVersionId:'ver-H01.A-FY26-1',updated:'Oct 12'},
  {planId:'H02.A',name:'Hybrid CXO & C&P Sales Leader',fy:'FY26',role:'CXO & C&P Hybrid Sales Leader A',owner:'Melissa C.',releasedVersionId:'ver-H02.A-FY26-1',updated:'Oct 15'},
  {planId:'H03.A',name:'Commerce AE',fy:'FY26',role:'Commerce Account Executive',owner:'Val R.',releasedVersionId:'ver-H03.A-FY26-1',updated:'Oct 20'},
  {planId:'H05.A',name:'Enterprise AE',fy:'FY26',role:'Enterprise Account Executive',owner:'Melissa C.',releasedVersionId:'ver-H05.A-FY26-1',updated:'Nov 1'},
  {planId:'H06.A',name:'SMB Renewal Specialist',fy:'FY26',role:'SMB Renewal Specialist',owner:'Val R.',releasedVersionId:'ver-H06.A-FY26-1',updated:'Oct 28'},
];

let versions=[
  {versionId:'ver-H01.A-FY26-1',versionNum:1,planId:'H01.A',fy:'FY26',name:'Version 1',label:'FY26 baseline',status:'released',createdBy:'Susan L.',createdAt:'Oct 12, 2025',updatedAt:'Oct 12, 2025',submittedAt:'Oct 12, 2025',releasedAt:'Oct 12, 2025',releasedBy:'Susan L.',supersededAt:null,parentVersionId:null},
  {versionId:'ver-H02.A-FY26-1',versionNum:1,planId:'H02.A',fy:'FY26',name:'Version 1',label:'FY26 baseline',status:'released',createdBy:'Susan L.',createdAt:'Oct 15, 2025',updatedAt:'Oct 15, 2025',submittedAt:'Oct 15, 2025',releasedAt:'Oct 15, 2025',releasedBy:'Susan L.',supersededAt:null,parentVersionId:null},
  {versionId:'ver-H03.A-FY26-1',versionNum:1,planId:'H03.A',fy:'FY26',name:'Version 1',label:'FY26 baseline',status:'released',createdBy:'Susan L.',createdAt:'Oct 20, 2025',updatedAt:'Oct 20, 2025',submittedAt:'Oct 20, 2025',releasedAt:'Oct 20, 2025',releasedBy:'Susan L.',supersededAt:null,parentVersionId:null},
  {versionId:'ver-H05.A-FY26-1',versionNum:1,planId:'H05.A',fy:'FY26',name:'Version 1',label:'FY26 baseline',status:'released',createdBy:'Susan L.',createdAt:'Nov 1, 2025',updatedAt:'Nov 1, 2025',submittedAt:'Nov 1, 2025',releasedAt:'Nov 1, 2025',releasedBy:'Susan L.',supersededAt:null,parentVersionId:null},
  {versionId:'ver-H06.A-FY26-1',versionNum:1,planId:'H06.A',fy:'FY26',name:'Version 1',label:'FY26 baseline',status:'released',createdBy:'Susan L.',createdAt:'Oct 28, 2025',updatedAt:'Oct 28, 2025',submittedAt:'Oct 28, 2025',releasedAt:'Oct 28, 2025',releasedBy:'Susan L.',supersededAt:null,parentVersionId:null},
  {versionId:'ver-H01.A-FY27-1',versionNum:1,planId:'H01.A',fy:'FY27',name:'Version 1',label:'Option A — FY26 structure carry-forward',status:'draft',createdBy:'Melissa C.',createdAt:'Jun 3, 2026',updatedAt:'Jun 5, 2026',submittedAt:null,releasedAt:null,releasedBy:null,supersededAt:null,parentVersionId:'ver-H01.A-FY26-1'},
  {versionId:'ver-H01.A-FY27-2',versionNum:2,planId:'H01.A',fy:'FY27',name:'Version 2',label:'Option B — Quarterly payout & renewal tether',status:'in-review',createdBy:'Melissa C.',createdAt:'Jun 8, 2026',updatedAt:'Jun 11, 2026',submittedAt:'Jun 11, 2026',releasedAt:null,releasedBy:null,supersededAt:null,parentVersionId:'ver-H01.A-FY27-1'},
  {versionId:'ver-H01.A-FY27-3',versionNum:3,planId:'H01.A',fy:'FY27',name:'Version 3',label:'Option C — Higher 150%+ accelerators',status:'draft',createdBy:'Val R.',createdAt:'Jun 10, 2026',updatedAt:'Jun 10, 2026',submittedAt:null,releasedAt:null,releasedBy:null,supersededAt:null,parentVersionId:null},
  {versionId:'ver-H02.A-FY27-1',versionNum:1,planId:'H02.A',fy:'FY27',name:'Version 1',label:'Hybrid CXO baseline FY27',status:'released',createdBy:'Melissa C.',createdAt:'May 28, 2026',updatedAt:'Jun 10, 2026',submittedAt:'Jun 8, 2026',releasedAt:'Jun 10, 2026',releasedBy:'Susan L.',supersededAt:null,parentVersionId:'ver-H02.A-FY26-1'},
  {versionId:'ver-H02.A-FY27-2',versionNum:2,planId:'H02.A',fy:'FY27',name:'Version 2',label:'VCT adjustment variant',status:'archived',createdBy:'Melissa C.',createdAt:'Jun 1, 2026',updatedAt:'Jun 9, 2026',submittedAt:null,releasedAt:null,releasedBy:null,supersededAt:null,parentVersionId:'ver-H02.A-FY27-1'},
  {versionId:'ver-H03.A-FY27-1',versionNum:1,planId:'H03.A',fy:'FY27',name:'Version 1',label:'Commerce FY27 plan',status:'released',createdBy:'Val R.',createdAt:'May 25, 2026',updatedAt:'Jun 9, 2026',submittedAt:'Jun 6, 2026',releasedAt:'Jun 9, 2026',releasedBy:'Susan L.',supersededAt:null,parentVersionId:'ver-H03.A-FY26-1'},
  {versionId:'ver-H04.A-FY27-1',versionNum:1,planId:'H04.A',fy:'FY27',name:'Version 1',label:'Initial SMB draft',status:'draft',createdBy:'Val R.',createdAt:'Jun 8, 2026',updatedAt:'Jun 8, 2026',submittedAt:null,releasedAt:null,releasedBy:null,supersededAt:null,parentVersionId:null},
  {versionId:'ver-H05.A-FY27-1',versionNum:1,planId:'H05.A',fy:'FY27',name:'Version 1',label:'Enterprise uncapped tier',status:'released',createdBy:'Melissa C.',createdAt:'Jun 1, 2026',updatedAt:'Jun 7, 2026',submittedAt:'Jun 5, 2026',releasedAt:'Jun 7, 2026',releasedBy:'Susan L.',supersededAt:null,parentVersionId:'ver-H05.A-FY26-1'},
  {versionId:'ver-H06.A-FY27-1',versionNum:1,planId:'H06.A',fy:'FY27',name:'Version 1',label:'Renewal-focused plan',status:'released',createdBy:'Val R.',createdAt:'May 20, 2026',updatedAt:'Jun 5, 2026',submittedAt:'Jun 3, 2026',releasedAt:'Jun 5, 2026',releasedBy:'Susan L.',supersededAt:null,parentVersionId:'ver-H06.A-FY26-1'},
  {versionId:'ver-H07.A-FY27-1',versionNum:1,planId:'H07.A',fy:'FY27',name:'Version 1',label:'Channel AE draft',status:'draft',createdBy:'Melissa C.',createdAt:'Jun 4, 2026',updatedAt:'Jun 4, 2026',submittedAt:null,releasedAt:null,releasedBy:null,supersededAt:null,parentVersionId:null},
  {versionId:'ver-H09.A-FY27-1',versionNum:1,planId:'H09.A',fy:'FY27',name:'Version 1',label:'EMEA enterprise revision',status:'rejected',createdBy:'Melissa C.',createdAt:'Jun 1, 2026',updatedAt:'Jun 2, 2026',submittedAt:'Jun 2, 2026',releasedAt:null,releasedBy:null,supersededAt:null,parentVersionId:null},
];

let auditLog=[
  {id:'aud-1',planId:'H01.A',fy:'FY27',versionId:'ver-H01.A-FY27-2',action:'Version submitted for approval',user:'Melissa C.',timestamp:'Jun 11, 2026 11:22 AM',detail:'Version 2 submitted to Susan L.'},
  {id:'aud-2',planId:'H05.A',fy:'FY27',versionId:'ver-H05.A-FY27-1',action:'Version released',user:'Susan L.',timestamp:'Jun 7, 2026 3:40 PM',detail:'Version 1 released — Enterprise uncapped tier.'},
  {id:'aud-4',planId:'H06.A',fy:'FY27',versionId:'ver-H06.A-FY27-1',action:'Version released',user:'Susan L.',timestamp:'Jun 5, 2026 10:15 AM',detail:'Version 1 released — Renewal-focused plan.'},
  {id:'aud-5',planId:'H01.A',fy:'FY27',versionId:'ver-H01.A-FY27-3',action:'Version marked ready for review',user:'Val R.',timestamp:'Jun 10, 2026 4:00 PM',detail:'Version 3 ready for internal design review.'},
];
```

- [ ] **Step 2: Also update the `screens` object and `trackerData` (lines ~1363–1375 and ~3246–3256)**

In the `screens` object replace:
```js
tracker:{title:'Proposal Tracker'},
```
with:
```js
tracker:{title:'Version Tracker'},
```

And update `trackerData` static entries that reference `status:'review'` (these are OK as-is since they're display-only static data, no action needed).

- [ ] **Step 3: Open the file in a browser. Expect JS errors since helper functions still reference old arrays — that's expected. Proceed to Task 2.**

---

### Task 2: Replace helper functions

**Files:**
- Modify: `comp-plan-prototype.html:1446–1741` (all data access and action functions)

**Interfaces:**
- Produces: `getPlan(planId,fy)`, `getVersion(versionId)`, `getVersionsForPlan(pk)`, `getActiveVersionsForPlan(pk)`, `getReleasedVersion(pk)`, `versionBadgeHtml(status)`, `addAuditEntry(pk,action,detail,opts)`, `nextVersionId(pk)`, `nextVersionNum(pk)`, `nextVersionName(pk)`, `releaseVersion(versionId)`, `submitVersionById(versionId)`, `openPlanDetail(planKey)`, `renderPlanDetail(planKey)`, `openCreateVersionModal()`, `closeCreateVersionModal()`, `populateCopyFromSelect(planKey)`, `confirmCreateVersion()`, `openVersionInBuilder(versionId)`, `switchBuilderVersion(versionId)`, `updateBuilderVersionBar(pk,versionId)`, `submitCurrentVersion()`, `approveCurrentVersion()`, `rejectCurrentVersion()`, `compareReleasedToVersion(planKey,versionId)`, `openCompareCurrentVsVersion()`, `filterVersionTracker()`, `buildVersionTracker()`

- [ ] **Step 1: Replace lines 1446–1741 in the file with these functions:**

```js
function getPlan(planId,fy){
  if(fy===undefined){
    if(String(planId).includes('|')){const parsed=parsePlanKey(planId);return getPlan(parsed.planId,parsed.fy);}
    return plans.find(p=>p.planId===planId&&p.fy==='FY27')||plans.find(p=>p.planId===planId);
  }
  return plans.find(p=>p.planId===planId&&p.fy===fy);
}
function getVersion(versionId){return versions.find(v=>v.versionId===versionId);}
function getVersionsForPlan(pk){
  const {planId,fy}=parsePlanKey(pk);
  return versions.filter(v=>v.planId===planId&&v.fy===fy);
}
function getActiveVersionsForPlan(pk){
  const {planId,fy}=parsePlanKey(pk);
  return versions.filter(v=>v.planId===planId&&v.fy===fy&&v.status!=='archived');
}
function getReleasedVersion(pk){
  const {planId,fy}=parsePlanKey(pk);
  const plan=getPlan(planId,fy);
  if(plan?.releasedVersionId)return getVersion(plan.releasedVersionId);
  return versions.find(v=>v.planId===planId&&v.fy===fy&&v.status==='released');
}
function getAuditForPlan(pk){
  const {planId,fy}=parsePlanKey(pk);
  return auditLog.filter(a=>a.planId===planId&&a.fy===fy);
}

function versionBadgeHtml(status){
  const m={draft:'badge-draft','in-review':'badge-review',released:'badge-published',rejected:'badge-rejected',archived:'badge-draft'};
  const label=VERSION_STATUS_LABELS[status]||status;
  return`<span class="badge ${m[status]||'badge-draft'}">${label}</span>`;
}

function addAuditEntry(pk,action,detail,opts={}){
  const {planId,fy}=parsePlanKey(pk);
  auditLog.unshift({
    id:'aud-'+Date.now(),planId,fy,versionId:opts.versionId||null,
    action,user:'Melissa C.',
    timestamp:new Date().toLocaleString('en-US',{month:'short',day:'numeric',year:'numeric',hour:'numeric',minute:'2-digit'}),
    detail,
  });
}

function nextVersionId(pk){
  const {planId,fy}=parsePlanKey(pk);
  const n=getVersionsForPlan(pk).length+1;
  return`ver-${planId}-${fy.replace('FY','')}-${n}`;
}
function nextVersionNum(pk){return getVersionsForPlan(pk).length+1;}
function nextVersionName(pk){return'Version '+nextVersionNum(pk);}

function openPlanDetail(pk){
  currentPlanKey=pk;
  renderPlanDetail(pk);
  showScreen('plan');
}

function renderPlanDetail(pk){
  const {planId,fy}=parsePlanKey(pk);
  const plan=getPlan(planId,fy);
  if(!plan)return;
  const relVer=getReleasedVersion(pk);
  const activeVers=getActiveVersionsForPlan(pk);
  document.getElementById('plan-detail-title').textContent=`Plan — ${planId} · ${fy}`;
  document.getElementById('plan-detail-subtitle').textContent=`${plan.name} · ${activeVers.length} version${activeVers.length===1?'':'s'}`;
  document.getElementById('plan-downstream-id').textContent=`${planId} · ${fy}`;
  document.getElementById('plan-container-name').textContent=plan.name;
  document.getElementById('plan-container-meta').innerHTML=`
    <span class="plan-fy-chip">${fy}</span>
    <span style="font-size:11px;color:var(--muted)">${plan.role||''}</span>
    <span style="font-size:11px;color:var(--muted)">Owner: ${plan.owner||'—'}</span>`;
  document.getElementById('plan-current-rev-label').textContent=relVer?relVer.name:'Not yet released';
  document.getElementById('plan-proposal-count').textContent=activeVers.length+' version'+(activeVers.length===1?'':'s');
  const banner=document.getElementById('plan-published-banner');
  if(relVer&&relVer.status==='released'){
    banner.innerHTML=`<div class="published-rev-banner"><i class="ti ti-circle-check" style="color:#639922;font-size:18px"></i><div><strong>Current Plan</strong> — ${relVer.name}${relVer.label?' · '+relVer.label:''} · ${relVer.fy} · Released ${relVer.releasedAt} by ${relVer.releasedBy}<div style="font-size:11px;color:var(--muted);margin-top:2px">This is what downstream systems reference for Plan ID ${planId} in ${fy}.</div></div><span class="badge badge-current-plan">Current Plan</span></div>`;
  }else{
    banner.innerHTML=`<div class="published-rev-banner" style="background:#FAEEDA;border-left-color:#BA7517"><i class="ti ti-clock" style="color:#BA7517;font-size:18px"></i><div><strong>No released version yet</strong> — downstream consumers have no Current Plan for ${planId} in ${fy}. Submit and approve a version to release.</div></div>`;
  }
  document.getElementById('plan-versions-list').innerHTML=activeVers.length?activeVers.map(v=>{
    const isReleased=relVer&&relVer.versionId===v.versionId;
    return`<div class="proposal-card ${v.status==='in-review'?'is-submitted':''} ${isReleased?'is-current-published':''}">
      <div class="proposal-card-main">
        <div class="proposal-card-title">${v.name} ${versionBadgeHtml(v.status)} ${isReleased?'<span class="badge badge-current-plan">Current Plan</span>':''}</div>
        <div class="proposal-card-sub">${v.label||'—'} · Updated ${v.updatedAt} · ${v.createdBy}${v.submittedAt?' · Submitted '+v.submittedAt:''}</div>
        ${v.parentVersionId?`<div style="font-size:10px;color:var(--muted);margin-top:4px">Based on ${getVersion(v.parentVersionId)?.name||'prior version'}</div>`:''}
      </div>
      <div class="proposal-card-actions">
        <button class="btn btn-sm" onclick="openVersionInBuilder('${v.versionId}')"><i class="ti ti-edit"></i> Edit</button>
        <button class="btn btn-sm" onclick="compareReleasedToVersion('${pk}','${v.versionId}')"><i class="ti ti-columns"></i> Compare</button>
        ${v.status==='draft'?`<button class="btn btn-sm btn-primary" onclick="submitVersionById('${v.versionId}')"><i class="ti ti-send"></i> Submit</button>`:''}
      </div>
    </div>`;
  }).join(''):'<div style="padding:20px;text-align:center;color:var(--muted);font-size:12px">No versions yet. Create Version 1 to begin design options.</div>';
  document.getElementById('plan-version-history').innerHTML=getVersionsForPlan(pk).filter(v=>v.status==='released'||v.supersededAt).map(v=>`
    <div class="revision-history-row">
      <span style="font-weight:500;min-width:80px">${v.name}</span>
      <span style="font-size:11px;color:var(--muted)">${v.fy}${v.releasedAt?' · '+v.releasedAt:''}</span>
      ${!v.supersededAt&&v.status==='released'?'<span class="badge badge-current-plan">Current Plan</span>':''}
      ${v.supersededAt?'<span class="badge badge-draft">Superseded</span>':''}
    </div>`).join('')||'<div style="font-size:12px;color:var(--muted)">No released versions yet.</div>';
  document.getElementById('plan-change-log').innerHTML=getAuditForPlan(pk).map(a=>`
    <div class="change-log-item"><strong>${a.action}</strong> · ${a.user} · ${a.timestamp}<div style="font-size:11px;color:var(--muted);margin-top:2px">${a.detail}</div></div>`).join('')||'<div style="font-size:12px;color:var(--muted)">No audit entries yet.</div>';
  screens.plan={title:`Plan — ${planId} · ${fy}`};
}

function openCreateVersionModal(){
  document.getElementById('version-modal-title').textContent='Create Version';
  const {planId,fy}=parsePlanKey(currentPlanKey);
  document.getElementById('version-plan-id').value=`${planId} · ${fy}`;
  document.getElementById('version-plan-key').value=currentPlanKey;
  document.getElementById('version-name').value=nextVersionName(currentPlanKey);
  document.getElementById('version-draft-label').value='';
  populateCopyFromSelect(currentPlanKey);
  document.getElementById('create-version-modal').classList.add('open');
}

function closeCreateVersionModal(){
  document.getElementById('create-version-modal').classList.remove('open');
}

function populateCopyFromSelect(pk){
  const relVer=getReleasedVersion(pk);
  const sel=document.getElementById('version-copy-from');
  let opts='<option value="">Blank version</option>';
  if(relVer)opts+=`<option value="version:${relVer.versionId}">Current Plan · ${relVer.name}</option>`;
  getActiveVersionsForPlan(pk).forEach(v=>{opts+=`<option value="version:${v.versionId}">${v.name} · ${v.label||VERSION_STATUS_LABELS[v.status]}</option>`;});
  sel.innerHTML=opts;
}

function confirmCreateVersion(){
  const pkVal=document.getElementById('version-plan-key').value||currentPlanKey;
  const {planId,fy}=parsePlanKey(pkVal);
  const name=document.getElementById('version-name').value.trim()||nextVersionName(pkVal);
  const label=document.getElementById('version-draft-label').value.trim();
  const dup=document.getElementById('version-copy-from').value;
  let parentVersionId=null;
  if(dup.startsWith('version:'))parentVersionId=dup.split(':')[1];
  const id=nextVersionId(pkVal);
  const num=nextVersionNum(pkVal);
  const ver={
    versionId:id,versionNum:num,planId,fy,
    name:name||'Version '+num,label:label||name||'',status:'draft',
    createdBy:'Melissa C.',
    createdAt:new Date().toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'}),
    updatedAt:new Date().toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'}),
    submittedAt:null,releasedAt:null,releasedBy:null,supersededAt:null,parentVersionId,
  };
  versions.push(ver);
  addAuditEntry(pkVal,'Version created',`${ver.name} created${dup?' from copy source':''}.`,{versionId:id});
  closeCreateVersionModal();
  buildPlanSnapshots();buildCompareSelects();
  openVersionInBuilder(id);
}

function openVersionInBuilder(versionId){
  const ver=getVersion(versionId);
  if(!ver)return;
  currentPlanKey=planKey(ver.planId,ver.fy);
  currentVersionId=versionId;
  currentDraftPlan={id:ver.planId,name:getPlan(ver.planId,ver.fy)?.name,versionId,fy:ver.fy,status:ver.status,designerNote:''};
  showBuilderWorkspace();
  openBuilder(ver.planId,false,false,versionId);
}

function switchBuilderVersion(versionId){
  if(versionId)openVersionInBuilder(versionId);
}

function updateBuilderVersionBar(pk,versionId){
  const {planId,fy}=parsePlanKey(pk);
  const ver=getVersion(versionId);
  const sel=document.getElementById('builder-version-select');
  if(!ver||!sel)return;
  const vList=getActiveVersionsForPlan(pk);
  sel.innerHTML=vList.map(v=>`<option value="${v.versionId}" ${v.versionId===versionId?'selected':''}>${v.name} · ${VERSION_STATUS_LABELS[v.status]}</option>`).join('');
  document.getElementById('builder-version-label').textContent=ver.name;
  document.getElementById('builder-version-meta').textContent=`${VERSION_STATUS_LABELS[ver.status]} · ${planId} · ${fy}`;
}

function submitVersionById(versionId){
  const ver=getVersion(versionId);
  if(!ver)return;
  const pk=planKey(ver.planId,ver.fy);
  ver.status='in-review';
  ver.submittedAt=new Date().toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'});
  ver.updatedAt=ver.submittedAt;
  approvalVersionId=versionId;
  addAuditEntry(pk,'Version submitted for approval',`${ver.name} submitted.`,{versionId});
  buildPlanSnapshots();buildCompareSelects();buildVersionTracker();
  if(currentPlanKey===pk)renderPlanDetail(pk);
  alert(`${ver.name} submitted for approval.`);
}

function submitCurrentVersion(){
  if(currentVersionId)submitVersionById(currentVersionId);
}

function approveCurrentVersion(){
  const ver=getVersion(approvalVersionId)||getVersion(currentVersionId);
  if(!ver){alert('No version selected.');return;}
  releaseVersion(ver.versionId);
  showScreen('plan');
  openPlanDetail(planKey(ver.planId,ver.fy));
}

function rejectCurrentVersion(){
  const ver=getVersion(approvalVersionId);
  if(!ver)return;
  const pk=planKey(ver.planId,ver.fy);
  ver.status='rejected';
  ver.updatedAt=new Date().toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'});
  addAuditEntry(pk,'Version rejected',`${ver.name} rejected by approver.`,{versionId:ver.versionId});
  buildPlanSnapshots();buildVersionTracker();renderPlanDetail(pk);
}

function releaseVersion(versionId){
  const ver=getVersion(versionId);
  if(!ver)return;
  const pk=planKey(ver.planId,ver.fy);
  const plan=getPlan(ver.planId,ver.fy);
  if(plan?.releasedVersionId){
    const prev=getVersion(plan.releasedVersionId);
    if(prev&&!prev.supersededAt){
      prev.supersededAt=new Date().toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'});
      addAuditEntry(pk,'Version superseded',`${prev.name} superseded when ${ver.name} released.`,{versionId:prev.versionId});
    }
  }
  ver.status='released';
  ver.releasedAt=new Date().toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'});
  ver.releasedBy='Susan L.';
  ver.updatedAt=ver.releasedAt;
  if(plan)plan.releasedVersionId=versionId;
  addAuditEntry(pk,'Version released',`${ver.name} released.`,{versionId});
  buildPlanSnapshots();buildCompareSelects();buildLibrary();buildVersionTracker();
}

function compareReleasedToVersion(pk,versionId){
  const relVer=getReleasedVersion(pk);
  if(!relVer){alert('No Current Plan released yet for this fiscal year.');return;}
  compareReturnScreen='plan';
  currentPlanKey=pk;
  setCompareSelection(`version|${relVer.versionId}`,`version|${versionId}`);
  applyCompareContext(comparePresets.standalone);
  showScreen('compare');
  syncCompareSelects();
  runComparison();
}

function openCompareCurrentVsVersion(){
  const vers=getActiveVersionsForPlan(currentPlanKey).filter(v=>v.status==='in-review');
  const ver=vers[0]||getActiveVersionsForPlan(currentPlanKey)[0];
  if(ver)compareReleasedToVersion(currentPlanKey,ver.versionId);
  else alert('Create a version first to compare against the Current Plan.');
}

function filterVersionTracker(){buildVersionTracker();}

function buildVersionTracker(){
  const tbody=document.getElementById('tracker-tbody');
  if(!tbody)return;
  const q=(document.getElementById('tracker-search')?.value||'').toLowerCase();
  const fyF=document.getElementById('tracker-fy-filter')?.value||'';
  const stF=document.getElementById('tracker-status-filter')?.value||'';
  const apF=document.getElementById('tracker-approval-filter')?.value||'';
  let rows=versions.filter(v=>{
    if(fyF&&v.fy!==fyF)return false;
    if(stF&&v.status!==stF)return false;
    if(apF==='pending'&&v.status!=='in-review')return false;
    if(apF==='approved'&&v.status!=='released')return false;
    if(apF==='released'){const plan=getPlan(v.planId,v.fy);if(!plan||plan.releasedVersionId!==v.versionId)return false;}
    if(q&&!(`${v.planId} ${v.fy} ${v.name} ${v.label}`.toLowerCase().includes(q)))return false;
    return true;
  });
  tbody.innerHTML=rows.map(v=>{
    const pk=planKey(v.planId,v.fy);
    const plan=getPlan(v.planId,v.fy);
    const isCurrentPlan=plan?.releasedVersionId===v.versionId;
    return`<tr>
      <td style="font-weight:500;color:#185FA5;cursor:pointer" onclick="openPlanDetail('${pk}')">${v.planId}</td>
      <td><strong>${v.name}</strong><div style="font-size:10px;color:var(--muted)">${v.label||'—'}</div></td>
      <td><span class="plan-fy-chip" style="font-size:10px;padding:2px 8px;background:#E6F1FB;color:#0C447C;border-radius:4px;font-weight:600">${v.fy}</span></td>
      <td>${versionBadgeHtml(v.status)}</td>
      <td style="font-size:11px">${v.submittedAt||'—'}</td>
      <td>${isCurrentPlan?'<span class="badge badge-current-plan">Current Plan</span>':'<span style="color:var(--muted);font-size:11px">—</span>'}</td>
      <td style="font-size:11px;color:var(--muted)">${v.updatedAt}</td>
      <td><div style="display:flex;gap:4px">
        <button class="btn btn-sm" onclick="openVersionInBuilder('${v.versionId}')" title="Edit"><i class="ti ti-edit"></i></button>
        <button class="btn btn-sm" onclick="compareReleasedToVersion('${pk}','${v.versionId}')" title="Compare"><i class="ti ti-columns"></i></button>
        ${v.status==='in-review'?`<button class="btn btn-sm" onclick="approvalVersionId='${v.versionId}';showScreen('approval')" title="Review"><i class="ti ti-circle-check"></i></button>`:''}
      </div></td>
    </tr>`;
  }).join('')||'<tr><td colspan="8" style="padding:20px;text-align:center;color:var(--muted)">No versions match filters</td></tr>';
}
```

- [ ] **Step 2: Open browser and confirm no undefined variable errors for the helper functions. The library, plan detail, and builder screens will still have some issues — those are fixed in later tasks.**

---

### Task 3: Update `buildPlanSnapshots`, snapshot keys, and compare presets

**Files:**
- Modify: `comp-plan-prototype.html:2502–2585` (`planSnapshots`, `compareQuickPresets`, `buildPlanSnapshots`, `makeSnapshot`)
- Modify: `comp-plan-prototype.html:3032–3060` (`comparePresets`, `openComparePlatform`)

**Interfaces:**
- Consumes: `getPlan(planId,fy)`, `getVersion(versionId)`, `versions`, `plans`, `fy26Baselines`, `fy27CurrentFields`
- Produces: `planSnapshots[]` where each entry has `key: 'version|ver-...'` or `key: 'legacy|H01.A|FY26'`

- [ ] **Step 1: Replace `compareQuickPresets` (around line 2507) with updated keys:**

```js
const compareQuickPresets=[
  {label:'H01.A · Current Plan vs Version 2',left:'version|ver-H01.A-FY26-1',right:'version|ver-H01.A-FY27-2'},
  {label:'H05.A · FY26 vs FY27 Released',left:'version|ver-H05.A-FY26-1',right:'version|ver-H05.A-FY27-1'},
  {label:'H01.A · Version 1 vs Version 3',left:'version|ver-H01.A-FY27-1',right:'version|ver-H01.A-FY27-3'},
  {label:'H02.A versions',left:'version|ver-H02.A-FY27-1',right:'version|ver-H02.A-FY27-2'},
];
```

- [ ] **Step 2: Replace `buildPlanSnapshots` and `makeSnapshot` (lines 2514–2585):**

```js
function buildPlanSnapshots(){
  planSnapshots=[];
  // FY26 legacy baselines (compare-engine field data for plans not yet in versions)
  Object.keys(fy26Baselines).forEach(pid=>{
    const plan=plans.find(p=>p.planId===pid)||{planId:pid,name:pid,role:'',fy:'FY26'};
    const pObj={id:pid,name:plan.name,role:plan.role,fy:'FY26',status:'approved',ver:'1.0'};
    const base=fy26Baselines[pid]||{};
    planSnapshots.push(makeSnapshot(pObj,'FY26','1.0','approved',base,{entityType:'legacy'}));
  });
  // All versions
  versions.forEach(ver=>{
    const plan=getPlan(ver.planId,ver.fy)||{planId:ver.planId,name:ver.planId,role:''};
    const isCurrentPlan=plan.releasedVersionId===ver.versionId;
    const raw=(ver.fy==='FY27'?fy27CurrentFields:fy26Baselines)[ver.planId]||fy26Baselines[ver.planId]||{};
    const pObj={id:ver.planId,name:plan.name||ver.planId,role:plan.role||'',fy:ver.fy,status:ver.status,ver:String(ver.versionNum)};
    const snap=makeSnapshot(pObj,ver.fy,String(ver.versionNum),ver.status,raw,{
      entityType:'version',
      versionId:ver.versionId,
      versionNum:ver.versionNum,
      isCurrentPlan,
    });
    snap.key=`version|${ver.versionId}`;
    snap.label=isCurrentPlan
      ?`${ver.planId} · ${ver.fy} — Current Plan · ${ver.name}`
      :`${ver.planId} · ${ver.fy} — ${ver.name} · ${ver.label||VERSION_STATUS_LABELS[ver.status]}`;
    snap.statusLabel=isCurrentPlan?'Current Plan':VERSION_STATUS_LABELS[ver.status];
    snap.name=ver.name+(ver.label?' · '+ver.label:'');
    planSnapshots.push(snap);
  });
}

function makeSnapshot(p,fy,ver,status,raw,meta={}){
  const statusLabel={draft:'Draft','in-review':'In Review',released:'Released',rejected:'Rejected',approved:'Approved',archived:'Archived'}[status]||status;
  const key=meta.versionId?`version|${meta.versionId}`:`${p.id}|${fy}|${ver}|${status}`;
  const built=buildPlanFieldData(p,fy,raw||{});
  const data={...built,designerNote:getSnapshotNote(key,p,fy)};
  return{
    key,id:p.id,name:p.name,fy,version:ver,
    versionNum:meta.versionNum||parseInt(ver)||1,
    status,statusLabel,
    label:`${p.id} — ${p.name} · ${fy} · v${ver} · ${statusLabel}`,
    searchText:`${p.id} ${p.name} ${p.role||''} ${fy} v${ver} ${statusLabel}`.toLowerCase(),
    fields:data,
    entityType:meta.entityType||'legacy',
    versionId:meta.versionId||null,
    isCurrentPlan:!!meta.isCurrentPlan,
  };
}
```

- [ ] **Step 3: Update `comparePresets` and `openComparePlatform` (around lines 3032–3060):**

Replace the `comparePresets` object:
```js
const comparePresets={
  standalone:{left:null,right:null,preset:'Custom comparison',ctx:'standalone',ctxDesc:'Compare Current Plan, versions, or prior releases — designer view only'},
  builder:{left:'version|ver-H01.A-FY26-1',right:'version|ver-H01.A-FY27-2',preset:'Current Plan vs version',ctx:'builder',ctxDesc:'Compare your version against the Current Plan for this Plan ID'},
  approval:{left:'version|ver-H01.A-FY26-1',right:'version|ver-H01.A-FY27-2',preset:'Current Plan vs submitted version',ctx:'approval',ctxDesc:'Approver view — Current Plan vs version submitted for approval'},
  duplicate:{left:'version|ver-H02.A-FY27-1',right:'version|ver-H07.A-FY27-1',preset:'Duplicate comparison',ctx:'duplicate',ctxDesc:'New version compared to source after duplication'}
};
```

In `openComparePlatform`, replace the block that references `getCurrentRevision`, `proposals.find`, `currentProposalId`, `approvalProposalId`:
```js
function openComparePlatform(context,planKeyArg,returnScreen){
  compareContext=context||'standalone';
  compareReturnScreen=returnScreen||(context==='builder'?'builder':context==='approval'?'approval':context==='plan'?'plan':'dashboard');
  const preset={...comparePresets[context]||comparePresets.standalone};
  if(planKeyArg){
    const {planId,fy}=parsePlanKey(planKeyArg);
    const relVer=getReleasedVersion(planKeyArg);
    const submitted=versions.find(v=>v.planId===planId&&v.fy===fy&&v.status==='in-review');
    if(relVer)preset.left=`version|${relVer.versionId}`;
    if(submitted)preset.right=`version|${submitted.versionId}`;
    else if(currentVersionId){
      const cv=getVersion(currentVersionId);
      if(cv&&cv.planId===planId&&cv.fy===fy)preset.right=`version|${currentVersionId}`;
    }
  }
  if(context==='approval'){
    const ver=getVersion(approvalVersionId);
    if(ver){
      const relVer=getReleasedVersion(planKey(ver.planId,ver.fy));
      if(relVer)preset.left=`version|${relVer.versionId}`;
      preset.right=`version|${ver.versionId}`;
    }
  }
  applyCompareContext(preset);
  if(context==='standalone'&&!planKeyArg&&!preset.left&&!preset.right){
    clearCompareSelection();
  }else{
    setCompareSelection(preset.left,preset.right);
    runComparison();
  }
  showScreen('compare');
}
```

- [ ] **Step 4: Update `refreshNotesDiff` (around line 2474) — replace two references to old proposal terminology:**

Change:
```js
document.getElementById('compare-approval-details').innerHTML=`
  <div><strong>Proposal:</strong> ${rightSnap.proposalId?getProposal(rightSnap.proposalId)?.proposalName:rightSnap.statusLabel} · ${rightSnap.statusLabel}</div>
  <div><strong>Current Plan:</strong> ${leftSnap.isCurrentPlan?`Published Revision ${leftSnap.revisionNumber}`:leftSnap.label} · ${leftSnap.fy}</div>
```
To:
```js
document.getElementById('compare-approval-details').innerHTML=`
  <div><strong>Version:</strong> ${rightSnap.name||rightSnap.statusLabel} · ${rightSnap.statusLabel}</div>
  <div><strong>Current Plan:</strong> ${leftSnap.isCurrentPlan?'Released Version':leftSnap.label} · ${leftSnap.fy}</div>
```

- [ ] **Step 5: Update `updateCompareSlot` (around line 2885) to use version terminology:**

Change `entityType==='proposal'` references to `entityType==='version'`:
```js
function updateCompareSlot(side,snap){
  const slot=document.getElementById('compare-slot-'+side);
  const summary=document.getElementById('compare-summary-'+side);
  const clearBtn=document.getElementById('compare-clear-'+side);
  if(!slot||!summary)return;
  if(!snap){slot.classList.remove('filled');summary.innerHTML='';if(clearBtn)clearBtn.style.display='none';return;}
  slot.classList.add('filled');
  if(clearBtn)clearBtn.style.display='inline';
  summary.innerHTML=`
    <div class="compare-slot-id">${snap.id}</div>
    <div class="compare-slot-name">${snap.isCurrentPlan?`Current Plan · ${snap.name}`:snap.name}</div>
    <div class="compare-slot-meta">
      <span class="plan-fy-chip">${snap.fy}</span>
      ${snap.isCurrentPlan?'<span class="badge badge-current-plan">Current Plan</span>':''}
      ${versionBadgeHtml(snap.status)}
    </div>`;
}
```

- [ ] **Step 6: Update `buildCompareSelects` (around line 2826) — change references to `entityType==='revision'`/`'proposal'` for sort ordering:**

```js
function buildCompareSelects(){
  const byPlan={};
  planSnapshots.forEach(s=>{if(!byPlan[s.id])byPlan[s.id]=[];byPlan[s.id].push(s);});
  const planOrder=Object.keys(byPlan).sort();
  const options=planOrder.map(pid=>{
    const snaps=byPlan[pid].sort((a,b)=>{
      if(a.isCurrentPlan&&!b.isCurrentPlan)return-1;
      if(b.isCurrentPlan&&!a.isCurrentPlan)return 1;
      if(a.entityType==='version'&&b.entityType!=='version')return-1;
      return(a.label||'').localeCompare(b.label||'');
    });
    const opts=snaps.map(s=>`<option value="${s.key}">${s.label||s.id}</option>`).join('');
    return`<optgroup label="Plan ${pid}">${opts}</optgroup>`;
  }).join('');
  ['left','right'].forEach(side=>{
    const sel=document.getElementById('compare-'+side+'-select');
    if(!sel)return;
    const placeholder=side==='left'?'Select Current Plan or baseline…':'Select version or option…';
    sel.innerHTML=`<option value="">${placeholder}</option>${options}`;
  });
}
```

- [ ] **Step 7: In `openBuilder` (around line 2134), update lines that use `currentProposalId`, `getProposal`, and `getActiveProposalsForPlan`:**

```js
// In openBuilder, change:
if(proposalId){
  currentProposalId=proposalId;
  const prop=getProposal(proposalId);
  if(prop)currentPlanKey=planKey(prop.planId,prop.fy);
}
// To:
if(proposalId){
  currentVersionId=proposalId;  // proposalId param is now a versionId
  const ver=getVersion(proposalId);
  if(ver)currentPlanKey=planKey(ver.planId,ver.fy);
}

// Change:
const snap=planSnapshots.find(s=>s.key===`proposal|${proposalId}`);
// To:
const snap=planSnapshots.find(s=>s.key===`version|${proposalId}`);

// Change:
const activeProposal=getActiveProposalsForPlan(pk).find(p=>p.status==='draft')||getActiveProposalsForPlan(pk)[0];
if(activeProposal){
  currentProposalId=activeProposal.proposalId;
  const snap=planSnapshots.find(s=>s.key===`proposal|${activeProposal.proposalId}`);
  if(snap){populateBuilderFromSnapshot(snap);updateBuilderProposalBar(pk,activeProposal.proposalId);}
}else{
  const snap=planSnapshots.find(s=>s.id===pid&&s.fy===fy&&s.entityType==='revision'&&s.isCurrentPlan)||planSnapshots.find(s=>s.id===pid&&s.fy===fy);
// To:
const activeVersion=getActiveVersionsForPlan(pk).find(v=>v.status==='draft')||getActiveVersionsForPlan(pk)[0];
if(activeVersion){
  currentVersionId=activeVersion.versionId;
  const snap=planSnapshots.find(s=>s.key===`version|${activeVersion.versionId}`);
  if(snap){populateBuilderFromSnapshot(snap);updateBuilderVersionBar(pk,activeVersion.versionId);}
}else{
  const snap=planSnapshots.find(s=>s.id===pid&&s.fy===fy&&s.isCurrentPlan)||planSnapshots.find(s=>s.id===pid&&s.fy===fy);
```

- [ ] **Step 8: Update `populateBuilderFromSnapshot` (around line 2695) — change the version ID cell source:**

```js
// Change:
setVal('builder-version-id',snap.versionId||'V1');
// To:
setVal('builder-version-id','V'+(snap.versionNum||1));
```

Also update the `updateBuilderProposalBar` call to `updateBuilderVersionBar`:
```js
// Change:
if(snap){populateBuilderFromSnapshot(snap);updateBuilderProposalBar(pk,proposalId);}
// To:
if(snap){populateBuilderFromSnapshot(snap);updateBuilderVersionBar(pk,proposalId);}
```

- [ ] **Step 9: Update `populateBuilderFromDraft` (around line 2717) — change the version ID cell and proposal bar call:**

```js
// Change:
setVal('builder-version-id',currentDraftPlan.versionId||'V1');
// To:
setVal('builder-version-id','V1');

// Change (if present):
updateBuilderProposalBar(pk, ...)
// To:
updateBuilderVersionBar(pk, ...)
```

- [ ] **Step 10: Update `applyBlankBuilderDefaults` (around line 2737) — the version ID default:**

```js
// Change:
setVal('builder-version-id','V1');
// No change needed — 'V1' is correct default display
```

- [ ] **Step 11: Open browser and verify: Compare screen dropdowns populate, compare quick-presets work.**

---

### Task 4: Update Library screen

**Files:**
- Modify: `comp-plan-prototype.html:582` (table header HTML)
- Modify: `comp-plan-prototype.html:3176–3213` (`buildLibrary` function)

**Interfaces:**
- Consumes: `plans`, `getReleasedVersion(pk)`, `getActiveVersionsForPlan(pk)`, `versions`, `getPlan`
- Produces: library table rows, one row per `plans` entry

- [ ] **Step 1: Update the library table header (line 582) — remove "Version" column, rename "Proposals" to "Versions":**

Change:
```html
<th>Plan ID</th><th>Version</th><th>Plan Name</th><th>Role</th><th>FY</th><th>Current Plan</th><th>Proposals</th><th>Last Updated</th><th>Owner</th><th>Actions</th>
```
To:
```html
<th>Plan ID</th><th>Plan Name</th><th>Role</th><th>FY</th><th>Current Plan</th><th>Versions</th><th>Last Updated</th><th>Owner</th><th>Actions</th>
```

- [ ] **Step 2: Replace `buildLibrary` function (lines 3176–3213):**

```js
function buildLibrary(){
  const tbody=document.getElementById('lib-tbody');
  if(!tbody)return;
  const fyF=document.getElementById('library-fy-filter')?.value||'';
  const q=(document.getElementById('library-search')?.value||'').toLowerCase();
  let rows=plans.filter(p=>{
    if(fyF&&p.fy!==fyF)return false;
    if(q&&!(`${p.planId} ${p.name} ${p.role} ${p.fy}`.toLowerCase().includes(q)))return false;
    return true;
  });
  const sub=document.getElementById('library-subtitle');
  if(sub)sub.textContent=`${rows.length} plan${rows.length===1?'':'s'} · ${fyF||'FY26 & FY27'}`;
  tbody.innerHTML=rows.map(p=>{
    const pk=planKey(p.planId,p.fy);
    const relVer=getReleasedVersion(pk);
    const verCount=getActiveVersionsForPlan(pk).length;
    const inReview=versions.find(v=>v.planId===p.planId&&v.fy===p.fy&&v.status==='in-review');
    return`<tr>
    <td style="font-weight:500;color:#185FA5;cursor:pointer" onclick="openPlanDetail('${pk}')">${p.planId}</td>
    <td>${p.name||''}</td><td style="font-size:11px;color:var(--muted)">${p.role||''}</td>
    <td><span class="plan-fy-chip" style="font-size:10px;padding:2px 8px;background:#E6F1FB;color:#0C447C;border-radius:4px;font-weight:600">${p.fy}</span></td>
    <td>${relVer?`<span class="badge badge-current-plan">${relVer.name}</span>`:'<span style="font-size:11px;color:var(--muted)">Not released</span>'}</td>
    <td><span class="badge badge-proposal">${verCount} version${verCount===1?'':'s'}</span>${inReview?' <span class="badge badge-review">1 in review</span>':''}</td>
    <td style="font-size:11px;color:var(--muted)">${p.updated||'—'}</td>
    <td style="font-size:11px">${p.owner||'—'}</td>
    <td><div style="display:flex;gap:4px">
      <button class="btn btn-sm" onclick="openPlanDetail('${pk}')" title="Versions"><i class="ti ti-versions"></i></button>
      <button class="btn btn-sm" onclick="openComparePlatform('standalone','${pk}')" title="Compare"><i class="ti ti-columns"></i></button>
      <button class="btn btn-sm" onclick="currentPlanKey='${pk}';openCreateVersionModal()" title="New version"><i class="ti ti-plus"></i></button>
    </div></td>
  </tr>`;
  }).join('');
}
```

- [ ] **Step 3: Update `buildSourcePlanList` (around line 2035) to use new `plans` schema:**

```js
function buildSourcePlanList(){
  const list=document.getElementById('source-plan-list');
  if(!list)return;
  list.innerHTML=plans.filter(p=>p.fy==='FY27').map(p=>{
    const relVer=getReleasedVersion(planKey(p.planId,p.fy));
    const statusStr=relVer?'Released':'Draft';
    return`<div class="source-picker-item" onclick="selectSourcePlan('${p.planId}')"><strong>${p.planId}</strong> — ${p.name} <span style="color:var(--muted)">· ${p.role}</span><span class="source-status"><span class="badge ${relVer?'badge-published':'badge-draft'}">${statusStr}</span></span></div>`;
  }).join('');
}
```

- [ ] **Step 4: Update `selectSourcePlan`, `getPlanFromList`, `nextPlanId` (around lines 2053–2033):**

```js
function selectSourcePlan(id){
  createPlanState.sourceId=id;
  const p=plans.find(x=>x.planId===id&&x.fy==='FY27')||plans.find(x=>x.planId===id);
  document.getElementById('source-plan-search').value=`${id} — ${p?.name||''}`;
  document.getElementById('source-plan-list').classList.remove('open');
  document.getElementById('selected-source-card').style.display='block';
  document.getElementById('selected-source-title').textContent=`${id} — ${p?.name||''}`;
  document.getElementById('selected-source-meta').innerHTML=`${p?.role||''} · ${p?.fy||'FY27'}`;
  document.getElementById('source-unchanged-ref').textContent=id;
  document.getElementById('audit-source-id').textContent=id;
  if(!document.getElementById('dup-plan-name').value)document.getElementById('dup-plan-name').value=(p?.name||'')+' (Copy)';
  if(!document.getElementById('dup-plan-id').value)document.getElementById('dup-plan-id').value=nextPlanId();
}

function getPlanFromList(id){
  const p=plans.find(x=>x.planId===id);
  return p?{id:p.planId,name:p.name+' FY27 Plan',role:p.role,payMix:'70 / 30',measures:'—',payoutTables:'—',bonusTether:'—',notes:'—'}:null;
}

function nextPlanId(){
  const nums=plans.map(p=>parseInt(p.planId.replace(/\D/g,''))||0);
  const max=Math.max(...nums,10);
  return 'H'+String(max+1).padStart(2,'0')+'.A';
}
```

- [ ] **Step 5: Open browser. Navigate to Library screen. Verify: rows show, Version column is gone, Versions column shows count, "Not released" / "Version 1" for current plan.**

---

### Task 5: Update Plan Detail screen HTML

**Files:**
- Modify: `comp-plan-prototype.html:590–637` (plan detail HTML)

**Interfaces:**
- Consumes: `renderPlanDetail` (already updated in Task 2) which writes to DOM elements

- [ ] **Step 1: Update plan detail HTML (lines 590–637):**

Change:
```html
<div class="page-subtitle" id="plan-detail-subtitle">Designer view · proposals &amp; published revisions</div>
```
To:
```html
<div class="page-subtitle" id="plan-detail-subtitle">Designer view · versions &amp; release history</div>
```

Change the button group (lines 597–601):
```html
<button class="btn btn-sm" onclick="openCreateProposalModal()"><i class="ti ti-plus"></i> Create Proposal</button>
<button class="btn btn-sm" onclick="openDuplicateProposalModal()"><i class="ti ti-copy"></i> Duplicate from Existing</button>
<button class="btn btn-primary btn-sm" onclick="openCompareCurrentVsProposal()"><i class="ti ti-columns"></i> Compare to Current Plan</button>
```
To:
```html
<button class="btn btn-sm" onclick="openCreateVersionModal()"><i class="ti ti-plus"></i> Create Version</button>
<button class="btn btn-primary btn-sm" onclick="openCompareCurrentVsVersion()"><i class="ti ti-columns"></i> Compare to Current Plan</button>
```

Change line 612:
```html
<div>Current published revision</div>
```
To:
```html
<div>Current released version</div>
```

Change line 619–622 (card title + count badge):
```html
<div class="card-title"><i class="ti ti-versions" style="color:#534AB7;margin-right:6px"></i>Proposals</div>
<span class="badge badge-proposal" id="plan-proposal-count">0 proposals</span>
```
To:
```html
<div class="card-title"><i class="ti ti-versions" style="color:#534AB7;margin-right:6px"></i>Versions</div>
<span class="badge badge-proposal" id="plan-proposal-count">0 versions</span>
```

Change `id="plan-proposals-list"` (line 623) to `id="plan-versions-list"`.

Change line 628 (revision history card title):
```html
<div class="card-title"><i class="ti ti-history" style="color:var(--adobe-red);margin-right:6px"></i>Published revision history</div>
```
To:
```html
<div class="card-title"><i class="ti ti-history" style="color:var(--adobe-red);margin-right:6px"></i>Release history</div>
```

Change `id="plan-revision-history"` (line 629) to `id="plan-version-history"`.

Also update the downstream note (line 603):
```html
the <strong>Current Plan</strong> (latest published revision for that fiscal year). Internal proposals are not distributed.
```
To:
```html
the <strong>Current Plan</strong> (latest released version for that fiscal year). Internal versions are not distributed.
```

- [ ] **Step 2: Open browser → click a plan in Library → Plan Detail screen. Verify: "Create Version" button shows, Versions card lists versions, Release history card shows released versions.**

---

### Task 6: Update Plan Builder HTML and version bar

**Files:**
- Modify: `comp-plan-prototype.html:641–675` (builder entry description + version bar HTML)

- [ ] **Step 1: Update builder entry text (lines 643–648):**

Change:
```html
<div class="builder-entry-sub">Each Plan ID + Version ID is a unique plan container with multiple internal proposals. The same Plan ID can exist with different Version IDs, but the same Plan ID + Version ID combination must be unique.</div>
```
To:
```html
<div class="builder-entry-sub">Each Plan ID + Fiscal Year is a unique plan. Create multiple versions within a plan to explore design options — one version gets released as the Current Plan.</div>
```

Change:
```html
<div class="builder-entry-desc">Start a new plan container with Proposal A — add up to four design options per plan.</div>
```
To:
```html
<div class="builder-entry-desc">Start a new plan — Version 1 is created automatically. Add up to four design versions per plan.</div>
```

Change:
```html
<div class="builder-entry-desc">Copy from Current Plan, a proposal, or another Plan ID into a new proposal or plan container.</div>
```
To:
```html
<div class="builder-entry-desc">Copy from a released version or another plan into a new version or plan.</div>
```

- [ ] **Step 2: Update the builder version bar HTML (lines 664–675):**

Change:
```html
<div class="builder-proposal-bar" id="builder-proposal-bar">
  <div class="builder-proposal-switch">
    <span class="badge badge-proposal">Proposal</span>
    <strong id="builder-proposal-label">Proposal A</strong>
    <span style="color:var(--muted);font-size:11px" id="builder-proposal-meta">Draft · H01.A</span>
    <select class="form-select" id="builder-proposal-select" style="width:auto;min-width:180px" onchange="switchBuilderProposal(this.value)"></select>
  </div>
  <div class="btn-group">
    <button class="btn btn-sm" onclick="openPlanDetail(currentPlanKey)"><i class="ti ti-layout-list"></i> All proposals</button>
    <button class="btn btn-sm" onclick="submitCurrentProposal()"><i class="ti ti-send"></i> Submit proposal</button>
  </div>
</div>
```
To:
```html
<div class="builder-proposal-bar" id="builder-version-bar">
  <div class="builder-proposal-switch">
    <span class="badge badge-proposal">Version</span>
    <strong id="builder-version-label">Version 1</strong>
    <span style="color:var(--muted);font-size:11px" id="builder-version-meta">Draft · H01.A</span>
    <select class="form-select" id="builder-version-select" style="width:auto;min-width:180px" onchange="switchBuilderVersion(this.value)"></select>
  </div>
  <div class="btn-group">
    <button class="btn btn-sm" onclick="openPlanDetail(currentPlanKey)"><i class="ti ti-layout-list"></i> All versions</button>
    <button class="btn btn-sm" onclick="submitCurrentVersion()"><i class="ti ti-send"></i> Submit version</button>
  </div>
</div>
```

- [ ] **Step 3: Open browser → Plan Builder. Click a plan to open it in builder. Verify: Version bar shows "Version 1" (or "Version 2"), select dropdown shows version options, Version cell in summary bar shows "V1"/"V2".**

---

### Task 7: Update Create Plan Modal (remove Version ID fields) and Create Version Modal

**Files:**
- Modify: `comp-plan-prototype.html:1230–1358` (Create Plan modal HTML)
- Modify: `comp-plan-prototype.html:1334–1359` (Create Version/Proposal modal HTML — rename)
- Modify: `comp-plan-prototype.html:1880–2132` (`createPlanNext`, `finalizeDuplicatePlan`)

- [ ] **Step 1: In the Create Plan modal (blank path, around line 1251–1252), remove the Version ID field. Replace the two-column form-row with just Plan ID:**

Change:
```html
<div class="form-row">
  <div class="form-group"><label class="form-label">Comp Plan ID *</label><input class="form-input" id="new-plan-id" placeholder="e.g. H11.A"><p style="font-size:10px;color:var(--muted);margin-top:4px">Reusable across versions. Must be unique per Version ID.</p></div>
  <div class="form-group"><label class="form-label">Version ID *</label><input class="form-input" id="new-plan-version" placeholder="e.g. V1"><p style="font-size:10px;color:var(--muted);margin-top:4px">Unique per Comp Plan ID. The same ID + Version cannot exist twice.</p></div>
</div>
```
To:
```html
<div class="form-row">
  <div class="form-group"><label class="form-label">Comp Plan ID *</label><input class="form-input" id="new-plan-id" placeholder="e.g. H11.A"><p style="font-size:10px;color:var(--muted);margin-top:4px">Unique per fiscal year. The same Plan ID + FY cannot exist twice.</p></div>
  <div class="form-group"><label class="form-label">Fiscal Year *</label><select class="form-select" id="new-plan-fy"><option>FY27</option><option>FY26</option></select></div>
</div>
```

(Note: the `new-plan-fy` select might already exist elsewhere in the form — if it does, remove the duplicate and keep only one.)

- [ ] **Step 2: In the clone/duplicate path (around line 1276), remove the Version ID field:**

Change:
```html
<div class="form-group"><label class="form-label">Version ID *</label><input class="form-input" id="dup-plan-version" placeholder="e.g. V1"><p style="font-size:10px;color:var(--muted);margin-top:4px">Must be unique per Comp Plan ID.</p></div>
```
To: delete this entire form-group (no replacement needed — version numbering is automatic).

- [ ] **Step 3: Update `createPlanNext` blank path (around line 1941–1972):**

```js
// Replace the blank plan creation block (s===2):
if(s===2){
  const name=document.getElementById('new-plan-name').value.trim();
  const id=document.getElementById('new-plan-id').value.trim();
  const fy=document.getElementById('new-plan-fy').value;
  if(!name||!id){alert('Please enter Comp Plan ID and Comp Plan Name.');return;}
  if(planExists(id,fy)){alert(`A plan with ID "${id}" already exists for ${fy}. Use a different Plan ID.`);return;}
  plans.push({planId:id,name,fy,role:document.getElementById('new-plan-role').value,owner:'Melissa C.',releasedVersionId:null,updated:new Date().toLocaleDateString('en-US',{month:'short',day:'numeric'})});
  const pk=planKey(id,fy);
  const verId=nextVersionId(pk);
  const verNum=nextVersionNum(pk);
  const verObj={
    versionId:verId,versionNum:verNum,planId:id,fy,name:'Version 1',label:'Initial design option',status:'draft',
    createdBy:'Melissa C.',
    createdAt:new Date().toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'}),
    updatedAt:new Date().toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'}),
    submittedAt:null,releasedAt:null,releasedBy:null,supersededAt:null,parentVersionId:null,
  };
  versions.push(verObj);
  addAuditEntry(pk,'Plan created',`Plan ${id} · ${fy} created with Version 1.`,{versionId:verId});
  currentPlanKey=pk;
  currentVersionId=verId;
  currentDraftPlan={id,name,role:document.getElementById('new-plan-role').value,fy,clonedFrom:null,versionId:verId,status:'draft',designerNote:''};
  buildPlanSnapshots();buildCompareSelects();buildLibrary();
  closeCreatePlanModal();
  openBuilder(id,true,false,verId);
  return;
}
```

- [ ] **Step 4: Update `createPlanNext` clone path (around line 1974–1990) — remove `newVersionId` references:**

```js
if(s===2){
  if(!createPlanState.sourceId){alert('Please select a source plan.');return;}
  const src=planDetails[createPlanState.sourceId]||getPlanFromList(createPlanState.sourceId);
  const newId=document.getElementById('dup-plan-id').value.trim()||nextPlanId();
  const newName=document.getElementById('dup-plan-name').value.trim()||(src?.name||'')+' (Copy)';
  const fy='FY27';
  if(planExists(newId,fy)){alert(`Plan "${newId}" already exists for ${fy}. Use a different Plan ID.`);return;}
  plans.push({planId:newId,name:newName,fy,role:src?.role||'',owner:'Melissa C.',releasedVersionId:null,updated:new Date().toLocaleDateString('en-US',{month:'short',day:'numeric'})});
  currentDraftPlan={...src,id:newId,name:newName,designerNote:src?.notes||src?.designerNote||'',clonedFrom:createPlanState.sourceId,fy,status:'draft',carriedFields:['planName','planId','role','payMix','measures','payoutTables','bonusTether','notes']};
  closeCreatePlanModal();
  openBuilder(newId,true,true);
  return;
}
```

- [ ] **Step 5: Update `finalizeDuplicatePlan` (around line 2105–2131) — remove `versionId` references, use `planExists`:**

```js
function finalizeDuplicatePlan(){
  const src=planDetails[createPlanState.sourceId]||getPlanFromList(createPlanState.sourceId);
  const fields=getCarriedFields();
  const newId=document.getElementById('dup-plan-id').value.trim();
  const newName=document.getElementById('dup-plan-name').value.trim();
  const fy='FY27';
  if(planExists(newId,fy)){alert(`Plan "${newId}" already exists for FY27. Use a different Plan ID.`);return;}
  plans.push({planId:newId,name:fields.planName?newName:'',fy,role:fields.role?src?.role:'',owner:'Melissa C.',releasedVersionId:null,updated:new Date().toLocaleDateString('en-US',{month:'short',day:'numeric'})});
  currentDraftPlan={
    id:newId,name:fields.planName?newName:'',role:fields.role?src?.role:'',
    payMix:fields.payMix?src?.payMix:'',fy,clonedFrom:createPlanState.sourceId,
    status:'draft',carriedFields:Object.keys(fields).filter(k=>fields[k]),
    measures:fields.measures?src?.measures:'',payoutTables:fields.payoutTables?src?.payoutTables:'',
    bonusTether:fields.bonusTether?src?.bonusTether:'',notes:fields.notes?src?.notes:'',designerNote:''
  };
  closeCreatePlanModal();
  openBuilder(newId,true,true);
}
```

- [ ] **Step 6: Rename the Create Proposal modal to Create Version modal (lines 1334–1359):**

Change:
```html
<div class="modal-overlay" id="create-proposal-modal" role="dialog">
  <div class="modal">
    <div class="modal-header">
      <div class="modal-title"><i class="ti ti-versions" style="color:#534AB7;margin-right:6px"></i> <span id="proposal-modal-title">Create Proposal</span></div>
      <button class="modal-close" onclick="closeCreateProposalModal()"><i class="ti ti-x"></i></button>
    </div>
    <div class="modal-body">
      <div class="form-group"><label class="form-label">Plan ID · Fiscal year</label><input class="form-input" id="proposal-plan-id" readonly><input type="hidden" id="proposal-plan-key"></div>
      <div class="form-row">
        <div class="form-group"><label class="form-label">Proposal name *</label><input class="form-input" id="proposal-name" placeholder="e.g. Proposal C"></div>
        <div class="form-group"><label class="form-label">Draft label</label><input class="form-input" id="proposal-draft-label" placeholder="e.g. Option C — Higher accelerators"></div>
      </div>
      <div class="form-group" id="proposal-duplicate-from-group">
        <label class="form-label">Duplicate from</label>
        <select class="form-select" id="proposal-duplicate-from">
          <option value="">Blank proposal</option>
        </select>
        <p style="font-size:11px;color:var(--muted);margin-top:6px">Copy from Current Plan, another proposal, or a prior published revision.</p>
      </div>
    </div>
    <div class="modal-footer">
      <button class="btn btn-sm" onclick="closeCreateProposalModal()">Cancel</button>
      <button class="btn btn-primary btn-sm" onclick="confirmCreateProposal()"><i class="ti ti-check"></i> Create proposal</button>
    </div>
  </div>
</div>
```
To:
```html
<div class="modal-overlay" id="create-version-modal" role="dialog">
  <div class="modal">
    <div class="modal-header">
      <div class="modal-title"><i class="ti ti-versions" style="color:#534AB7;margin-right:6px"></i> <span id="version-modal-title">Create Version</span></div>
      <button class="modal-close" onclick="closeCreateVersionModal()"><i class="ti ti-x"></i></button>
    </div>
    <div class="modal-body">
      <div class="form-group"><label class="form-label">Plan ID · Fiscal year</label><input class="form-input" id="version-plan-id" readonly><input type="hidden" id="version-plan-key"></div>
      <div class="form-row">
        <div class="form-group"><label class="form-label">Version name</label><input class="form-input" id="version-name" placeholder="e.g. Version 3"></div>
        <div class="form-group"><label class="form-label">Label</label><input class="form-input" id="version-draft-label" placeholder="e.g. Option C — Higher accelerators"></div>
      </div>
      <div class="form-group">
        <label class="form-label">Copy from</label>
        <select class="form-select" id="version-copy-from">
          <option value="">Blank version</option>
        </select>
        <p style="font-size:11px;color:var(--muted);margin-top:6px">Copy from Current Plan or another version as a starting point.</p>
      </div>
    </div>
    <div class="modal-footer">
      <button class="btn btn-sm" onclick="closeCreateVersionModal()">Cancel</button>
      <button class="btn btn-primary btn-sm" onclick="confirmCreateVersion()"><i class="ti ti-check"></i> Create version</button>
    </div>
  </div>
</div>
```

- [ ] **Step 7: Open browser → Plan Detail → "Create Version" button. Verify modal opens with plan ID shown, version name auto-populated as "Version N", "Copy from" dropdown populated.**

---

### Task 8: Update Approval screen, Tracker screen, and nav labels

**Files:**
- Modify: `comp-plan-prototype.html:446` (nav label)
- Modify: `comp-plan-prototype.html:875–890` (approval screen text)
- Modify: `comp-plan-prototype.html:967–989` (tracker HTML)
- Modify: `comp-plan-prototype.html:1836–1845` (`showScreen` approval block)

- [ ] **Step 1: Update nav item (line 446):**

Change:
```html
<div class="nav-item" onclick="showScreen('tracker')"><i class="ti ti-list-check"></i> Proposal Tracker</div>
```
To:
```html
<div class="nav-item" onclick="showScreen('tracker')"><i class="ti ti-list-check"></i> Version Tracker</div>
```

- [ ] **Step 2: Update approval screen text (around line 885):**

Change:
```html
<div>Reviewing <strong id="approval-proposal-name">Proposal B</strong> against the <strong>Current Plan</strong> (Published Revision <span id="approval-current-rev-num">1</span>). On approval, a new published revision is created — the prior revision is preserved in history.</div>
```
To:
```html
<div>Reviewing <strong id="approval-version-name">Version 2</strong> against the <strong>Current Plan</strong>. On approval, this version is released as the Current Plan — the prior released version is preserved in history.</div>
```

- [ ] **Step 3: Update `showScreen` for approval case (around line 1836–1845):**

Change:
```js
if(id==='approval'){
  const prop=getProposal(approvalProposalId);
  if(prop){
    document.getElementById('approval-page-title').textContent=`Approval Review — ${prop.proposalName}`;
    document.getElementById('approval-page-subtitle').textContent=`${prop.planId} · ${prop.fy} · ${prop.draftLabel||''} · Submitted ${prop.submittedAt||'—'}`;
    document.getElementById('approval-proposal-name').textContent=prop.proposalName;
    const rev=getCurrentRevision(planKey(prop.planId,prop.fy));
    if(rev)document.getElementById('approval-current-rev-num').textContent=rev.revisionNumber;
  }
}
```
To:
```js
if(id==='approval'){
  const ver=getVersion(approvalVersionId);
  if(ver){
    document.getElementById('approval-page-title').textContent=`Approval Review — ${ver.name}`;
    document.getElementById('approval-page-subtitle').textContent=`${ver.planId} · ${ver.fy} · ${ver.label||''} · Submitted ${ver.submittedAt||'—'}`;
    document.getElementById('approval-version-name').textContent=ver.name;
  }
}
```

- [ ] **Step 4: Update tracker screen HTML (lines 967–989):**

Change `page-title` and `page-subtitle`:
```html
<div class="page-title">Proposal Tracker</div><div class="page-subtitle">All proposals by Plan ID · designer &amp; approval view</div>
```
To:
```html
<div class="page-title">Version Tracker</div><div class="page-subtitle">All versions by Plan ID · designer &amp; approval view</div>
```

Change the search placeholder:
```html
placeholder="Search Plan ID or proposal…"
```
To:
```html
placeholder="Search Plan ID or version…"
```

Change the status filter options to match new statuses:
```html
<select class="filter-select" id="tracker-status-filter" onchange="filterVersionTracker()"><option value="">All version statuses</option><option value="draft">Draft</option><option value="in-review">In Review</option><option value="released">Released</option><option value="rejected">Rejected</option><option value="archived">Archived</option></select>
```

Change `oninput="filterProposalTracker()"` on the search input to `oninput="filterVersionTracker()"` and `onchange="filterProposalTracker()"` on other selects to `onchange="filterVersionTracker()"`.

Change the table headers:
```html
<th>Plan ID</th><th>Proposal</th><th>FY</th><th>Proposal status</th><th>Submitted</th><th>Current Plan</th><th>Published rev.</th><th>Updated</th><th>Actions</th>
```
To:
```html
<th>Plan ID</th><th>Version</th><th>FY</th><th>Status</th><th>Submitted</th><th>Current Plan</span></th><th>Updated</th><th>Actions</th>
```

- [ ] **Step 5: Update the approval button text in the approval screen footer (find "Approve & release" or similar):**

Find and update any button text that says "Publish" or "Publish new revision" to "Approve & release":
```html
<!-- Find the approval action buttons and ensure they call the correct functions -->
<!-- approveCurrentProposal() → approveCurrentVersion() -->
<!-- rejectCurrentProposal() → rejectCurrentVersion() -->
```

Search for `approveCurrentProposal` and `rejectCurrentProposal` in the HTML onclick attributes (not just JS) and change to `approveCurrentVersion` and `rejectCurrentVersion`.

- [ ] **Step 6: Remove `syncCompPlansFromLegacy` function call if it exists at init time (search for `syncCompPlansFromLegacy()` near line 3327 and remove it).**

- [ ] **Step 7: Open browser → Version Tracker. Verify: page title is "Version Tracker", rows show versions with correct statuses, filter dropdowns work.**

- [ ] **Step 8: Open browser → Approval screen. Verify: title shows "Approval Review — Version 2", approve/reject buttons work.**

---

### Task 9: Fix remaining references and init-time calls

**Files:**
- Modify: `comp-plan-prototype.html` — final sweep for any remaining old references

- [ ] **Step 1: Search the file for `compPlans` and replace any remaining references. There should be none after Tasks 1–8, but verify:**

Search for: `compPlans` → should return 0 results.
Search for: `publishedRevisions` → should return 0 results.
Search for: `proposals.` → should return 0 results (except inside comments if any).
Search for: `getProposal(` → should return 0 results.
Search for: `getCurrentRevision(` → should return 0 results.
Search for: `approvalProposalId` → should return 0 results.
Search for: `currentProposalId` → should return 0 results.
Search for: `proposalBadgeHtml` → should return 0 results.
Search for: `openCreateProposalModal` → should return 0 results (HTML + JS both updated).
Search for: `closeCreateProposalModal` → should return 0 results.
Search for: `confirmCreateProposal` → should return 0 results.
Search for: `buildProposalTracker` → should return 0 results.
Search for: `filterProposalTracker` → should return 0 results.
Search for: `submitCurrentProposal` → should return 0 results.
Search for: `approveCurrentProposal` → should return 0 results.
Search for: `rejectCurrentProposal` → should return 0 results.
Search for: `rescindCurrentPublished` → should return 0 results.

For any found, fix them individually.

- [ ] **Step 2: Check `saveBuilderPlanNote` (around line 2431) — remove `compPlans`-based note key logic if any:**

The function uses a snapshot key to store notes. Ensure it still works by referencing `currentVersionId` if needed.

- [ ] **Step 3: Update the bottom initialization block (around line 3320–3330):**

Find the initialization calls at the bottom of the `<script>` tag. Change:
- `buildProposalTracker()` → `buildVersionTracker()`
- Remove `syncCompPlansFromLegacy()` if present
- Ensure `buildPlanSnapshots()`, `buildCompareSelects()`, `buildLibrary()`, `buildVersionTracker()` are all called

- [ ] **Step 4: Final browser test — full regression:**
  - Open library: 15 plans visible, no Version column, Versions count correct
  - Click H01.A plan: Plan detail shows 3 versions (V1 draft, V2 in-review, V3 draft)
  - "No released version yet" banner shows for H01.A
  - Click H06.A plan: "Current Plan — Version 1" banner shows (released)
  - Builder entry: updated description
  - Open H01.A V2 in builder: version bar shows "Version 2", Version cell shows "V2"
  - Submit V2 for approval → status becomes "in-review" in plan detail
  - Approve V2 → status becomes "released", H01.A plan detail shows "Current Plan — Version 2"
  - Create Version modal: opens, auto-names "Version 4", no Version ID field
  - Create Plan: no Version ID field, requires only Plan ID and FY
  - Version Tracker: shows all versions, filters work
  - Compare: quick presets load, dropdowns show "version|ver-..." keys, comparison renders

---

## Self-Review

### Spec coverage check

| Spec requirement | Task that covers it |
|---|---|
| `plans` unique by `planId+fy`, no `versionId` | Task 1 (data), Task 7 (modal) |
| `versions` replaces `proposals + publishedRevisions` | Task 1 (data), Task 2 (functions) |
| `releasedVersionId` on plan | Task 1 (data), Task 2 (`releaseVersion`) |
| Version lifecycle: `draft→in-review→released|rejected|archived` | Task 2 (`submitVersionById`, `releaseVersion`, `rejectCurrentVersion`) |
| `supersededAt` set on prior released version | Task 2 (`releaseVersion`) |
| Library: no Version column, Versions count | Task 4 |
| Plan detail: Versions tab, Create Version button | Tasks 5, 2 |
| Builder: Version cell shows versionNum | Task 3 (makeSnapshot) |
| Builder version bar renamed | Task 6 |
| Approval: releases version | Task 8 (`approveCurrentVersion`) |
| Tracker: Version Tracker, new columns | Tasks 8, 2 |
| Compare: version keys, updated labels | Task 3 |
| Create Plan: no Version ID field | Task 7 |
| Terminology: Proposal → Version throughout | Tasks 5, 6, 7, 8 |

### Placeholder scan — none found.

### Type consistency
- All functions in Task 2 use `versionId` (not `proposalId`) as parameter name for version lookups.
- All snapshots in Task 3 use `key: 'version|ver-...'` format.
- `openVersionInBuilder(versionId)` is called from: `confirmCreateVersion`, `openPlanDetail` buttons, `buildVersionTracker` — all pass `ver.versionId`.
- `approvalVersionId` (global) is set in `submitVersionById` and used in `approveCurrentVersion`, `rejectCurrentVersion`, `showScreen('approval')`.

---

Plan complete and saved to `docs/superpowers/plans/2026-06-17-option-b-plan-version-model.md`.

**Two execution options:**

**1. Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?**
