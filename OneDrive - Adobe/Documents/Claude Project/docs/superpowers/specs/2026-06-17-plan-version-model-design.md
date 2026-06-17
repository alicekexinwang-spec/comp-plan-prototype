# Comp Plan Version Model — Design Spec
Date: 2026-06-17

## Problem

The current three-tier hierarchy (`compPlans container → proposals → publishedRevisions`) is unnecessarily complex and uses confusing terminology. Users think of a plan as having multiple **versions** (design alternatives), one of which gets picked to release. The current model adds an extra indirection layer between those concepts.

A separate `versionId` was recently added at the plan-container level (V1/V2), but that concept belongs inside the plan as a version number on each design alternative — not at the container level.

## Goal

Flatten to a clean two-tier model:
- **Plan** — the comp plan for a role in a fiscal year
- **Version** — a design alternative within that plan; one gets released

## Identity Rules

- A **Plan** is uniquely identified by `planId + fy`. No two plans may share the same `planId + fy`.
- A **Version** is uniquely identified by `versionId + planId + fy` (auto-numbered V1, V2, V3…).
- At most one version per plan may have status `released` at any time.

## Data Model

### `plans` array (replaces `compPlans`)
```js
{
  planId,       // e.g. "H01.A"
  fy,           // "FY27" | "FY26"
  name,         // "Digital Experience AE"
  role,         // "Digital Experience Account Executive"
  owner,        // "Melissa C."
  releasedVersionId  // null | "ver-H01.A-FY27-2" — points to the current released version
}
```
No `versionId` at this level. The old plan-level `versionId` (V1/V2) is removed.

### `versions` array (replaces both `proposals` and `publishedRevisions`)
```js
{
  versionId,    // "ver-H01.A-FY27-1" (auto-generated)
  versionNum,   // 1, 2, 3 … (display number, auto-incremented per plan)
  planId,
  fy,
  name,         // "Version 1" (default) or user-provided label
  label,        // "Option A — FY26 carry-forward" (optional subtitle)
  status,       // "draft" | "in-review" | "released" | "rejected" | "archived"
  createdBy,
  createdAt,
  updatedAt,
  submittedAt,  // when sent for approval
  releasedAt,   // when approved & released
  releasedBy,
  supersededAt, // set when a newer version is released and this one is no longer current
  parentVersionId  // null | id — which version this was cloned from
}
```

The `releasedVersionId` on the plan points to whichever version is currently released. When a new version is released, the prior released version has `supersededAt` set (timestamp) and is no longer current — it remains queryable for history. Manually closing a version that was never released sets `status: 'archived'`. This distinguishes "was released, now superseded" from "was never released, manually closed."

## Version Lifecycle

```
draft → in-review → released
              ↓
           rejected → (back to draft)
draft/in-review → archived (manually)
```

- **Draft** — being edited in Plan Builder
- **In-review** — submitted for approval, read-only
- **Released** — approved; this is the Current Plan downstream systems see
- **Rejected** — returned by approver; editable again
- **Archived** — superseded or manually closed out

## Screen-by-Screen Changes

### Library (`screen-library`)
- One row per plan (`planId + fy`)
- "Current" column shows which version is released: `V2 · Released` or `Not released`
- No "Version" column at plan level (removed)
- "Versions" count column (was "Proposals")

### Plan Detail (`screen-plan`)
- Title: "Plan — H01.A · FY27"
- Lists all versions with version number, status badge, label
- Released version highlighted with `Current Plan` badge
- Actions per version: Edit, Compare, Submit for approval
- "Create Version" button (was "Create Proposal")
- Release history tab shows previously released versions in order

### Plan Builder (`screen-builder`)
- Summary bar: `Plan # | Version | HC | Paymix` — "Version" cell shows auto-number (V1, V2…)
- Version populated from `versions.versionNum`; no longer from plan-level `versionId`
- Proposal bar renamed to Version selector

### Approval (`screen-approval`)
- Primary action: "Approve & release" — sets version status to `released`, archives prior released version
- Secondary: "Return to draft", "Reject"

### Tracker (`screen-tracker`)
- Rows are versions (was proposals)
- Columns: Plan ID, Version, FY, Status, Submitted, Released, Updated, Actions

### Compare (`screen-compare`)
- Dropdowns show versions by plan: "H01.A · FY27 — V1 · Draft", "H01.A · FY27 — V2 · Released"

## Terminology Swap

| Old | New |
|---|---|
| Proposal | Version |
| Published Revision | Released Version |
| "Create Proposal" | "Create Version" |
| `proposals` array | `versions` array |
| `publishedRevisions` array | removed (status on version handles this) |
| `proposalId` | `versionId` |
| `proposalName` ("Proposal A") | `name` ("Version 1") |
| `compPlans` array | `plans` array |
| Plan-level `versionId` (V1/V2) | removed |

## What Is Removed

- `publishedRevisions` array — replaced by `status: 'released'` on versions + `releasedVersionId` pointer on plan
- `compPlans` array — renamed to `plans` (same fields minus `versionId`)
- Plan-level `versionId` field (the V1/V2 we added in the previous session)
- `currentPublishedRevisionId` on plan — replaced by `releasedVersionId`

## What Stays the Same

- Plan identity: `planId + fy`
- Builder field structure (measures, paymix, bonus, tether, etc.)
- Compare engine
- Audit log
- Plan document / mechanics screens (minor label updates only)
