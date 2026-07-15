<COVER>
Comp Delivery Experience (CODEX)
Product Requirements Document
Compensation Plan Creation, Management & Comparison
Author: Alice Wang
2026-07-15
</COVER>

<PAGEBREAK>

# 1. Executive Summary

Adobe's Compensation Design and Policy teams author sales compensation plans that define how salespeople are
paid — which performance measures count, how attainment against quota converts into a payout, and what
bonuses and guardrails apply. Today this work lives in spreadsheets and documents.

**Business problem.** Plans are built and maintained manually, so there is no single source of truth, no
reliable version history, no structured reuse of common building blocks, and no easy way to compare one plan
or version against another.

**Goal.** Give the Compensation team a structured, consistent way to author plans and their role variants
("flavors"), manage versions, reuse standardized building blocks, and compare plans, versions, and flavors —
faster and with fewer errors.

**Proposed solution.** **Comp Delivery Experience (CODEX)** is a purpose-built application for creating,
managing, and comparing compensation plans: a plan library with search, cloning, versioning and archiving; a
guided plan builder organized by flavor (with per-measure payout tables, bonuses, and tether guardrails); and
an on-demand comparison tool that highlights differences.

**Benefits.** Faster authoring, fewer errors, consistent plan structures, a dependable version trail, and
confident change management through comparison. (Approval, document generation, and downstream publishing are
future phases and are out of scope for this document.)

<PAGEBREAK>

# 2. Background

## 2.1 Current process
Compensation plans are currently authored and maintained primarily in **Excel and documents**. A designer
copies last year's spreadsheet, edits measures, weightings, and payout schedules by hand, and circulates
files for input. Variants for different roles are separate tabs or separate files.

## 2.2 Pain points
- **Manual maintenance** — every plan and variant is hand-edited; formulas and structures drift.
- **No version management** — no reliable history of what changed, when, or why; prior versions are scattered.
- **Difficulty comparing plans** — comparing this year vs. last year, or one role vs. another, is a manual,
  error-prone eyeball exercise.
- **Inconsistent plan structures** — no enforced shape across plans.
- **Duplicate effort** — the same payout schedules and measure definitions are re-created repeatedly.
- **No reuse of standards** — common quota bands / attainment tiers are re-keyed each time.

## 2.3 Why this project exists
The business needs a **structured system of record** for compensation plan design that enforces a consistent
structure, remembers versions, reuses standard building blocks, and makes comparison trivial.

<PAGEBREAK>

# 3. Scope

## 3.1 In Scope
- **Compensation Plan Creation** — create a plan, define plan information, manage plan metadata.
- **Compensation Plan Management** — edit, save drafts, create versions, publish versions, archive, clone,
  search & filter.
- **Flavor Management** — create, edit, delete, and organize role variants ("flavors").
- **Performance Measure Management** — create measures; configure weight, payout frequency, performance
  period; link payout tables; configure bonuses, tether, and additional plan attributes.
- **Payout Table Management** — define reusable payout-table setups (library); reference a setup from a
  measure and adjust it; one standardized quota-band + attainment-tier structure with xPCR, MCR, pay curve,
  caps, and thresholds.
- **Compensation Plan Comparison** — compare two versions, two plans, or flavors, side by side with
  differences highlighted.

## 3.2 Out of Scope (future phases)
Approval workflow · Review workflow · Comments · Notifications · Delegation · Document generation · Plan
mechanics generation · Publishing to downstream systems · Employee plan distribution · Reporting dashboards ·
Workflow automation · Audit logs · Security implementation · Technical/API design.

<PAGEBREAK>

# 4. User Roles

Described from a business perspective for the in-scope capabilities. (Reviewers and approvers belong to the
future approval phase and are not detailed here.)

| Role | Responsibilities | Goals | Primary activities |
|---|---|---|---|
| **Comp Designer** | Author and maintain compensation plans and their flavors | Produce accurate, consistent plans quickly | Create/edit plans, flavors, measures, payout tables, bonuses, tether; save drafts; create versions; compare |
| **Comp Policy** | Govern the standards and building blocks plans must use | Ensure plans conform to policy | Maintain reusable libraries (quota band sets, attainment tier sets, performance-measure list, payout-table setups); review structure via comparison |
| **Read-only Stakeholder** | Business SMEs, sponsors, finance partners | Understand and validate plans | View plans and flavors; run comparisons; provide feedback |
| **Administrator** | Manage reference data and configuration libraries | Keep shared building blocks current | Create/edit reusable sets and the performance-measure catalog |

<PAGEBREAK>

# 5. Proposed Solution Overview

CODEX provides a **plan library**, a **guided plan builder** organized by flavor, reusable **configuration
libraries**, and an on-demand **comparison tool**.

```flow
Create Compensation Plan
Configure Flavors (role variants)
Configure Performance Measures (per flavor)
Reference Payout Tables (from the reusable library)
Configure Bonuses & Tether
Save Draft
Publish Version (becomes the current version)
```

A **Compensation Plan** is the top-level item (identified by a plan number and a fiscal year). It contains
one or more **Flavors** (role variants such as A/B/C). Each flavor has its own **Performance Measures**
(weighted metrics); each measure has a **Payout Table** (attainment-to-payout schedule, referenced from a
reusable library of payout-table setups and copied into the plan), plus **Bonuses**, a **Tether** guardrail,
and policy notes. Plans move through **draft → published version**.

💡 **Comparison is a standalone capability, not a workflow step** — a user can compare any two plans,
versions, or flavors at any time (see 6.8).

<PAGEBREAK>

# 6. Functional Requirements

*Each module below follows the same structure: **Overview → User Workflow → Functional Requirements
(table) → Business Rules (table) → Prototype Screenshot**. Requirement IDs are module-prefixed (e.g.
CPM-001); business-rule IDs restart per module (BR-01…).*

## 6.1 Compensation Plan Management

**Overview** The **Compensation Plan** is the top-level business object, identified by a **Comp Plan ID +
fiscal year** (e.g. A01 · FY27). It holds one or more flavors and evolves over time through proposals and
published versions. This module covers the plan **as a container and how new work is initiated**: finding
plans in the Library, plan identity and creation routing, editing plan information/metadata, **cloning**, and
archiving. A key rule governs creation: **entering a NEW Comp Plan ID creates a new Compensation Plan;
entering an EXISTING Comp Plan ID initiates a new Proposal under that plan** (rather than a second plan). To
avoid rebuilding from scratch, a designer can **clone** an existing plan or a specific version — copying its
full content (flavors, measures, payout tables, bonuses, tether, policies) as the starting point; the same
new-vs-existing Comp Plan ID rule then determines whether the clone lands as a **new plan** or a **new
proposal** under an existing plan.

> **Scope boundary.** This module ends once a proposal has been *initiated*. Everything that happens to an
> individual proposal after that — editing the draft, publishing it to become the current version, and
> version history — is covered in **§6.2 Proposal Management**.

**User Workflow**
```flow
Open Plan Library
Search / filter plans
Start from scratch (enter Comp Plan ID + fiscal year) OR Clone an existing plan / version
Decision: does the target Comp Plan ID already exist?
If NEW ID -> create a new Compensation Plan (enter name + metadata)
If EXISTING ID -> create a new Proposal under that Compensation Plan
(If cloned) the source content is copied in as the starting point
Open in the Plan Builder to configure
Archive the plan when retired
```

**Functional Requirements**
| ID | Requirement |
|---|---|
| CPM-001 | The system shall display a Plan Library listing all compensation plans with key attributes (Comp Plan ID, name, fiscal year, status, current version, creator, last updated). |
| CPM-002 | The system shall allow users to search and filter plans by fiscal year, Comp Plan ID, status, creator, and free text. |
| CPM-003 | The system shall create a **new Compensation Plan** when the entered Comp Plan ID does not already exist for the selected fiscal year. |
| CPM-004 | The system shall **initiate a new Proposal under the existing Compensation Plan** when the entered Comp Plan ID already exists (the proposal is then created and managed per §6.2). |
| CPM-005 | The system shall capture plan metadata: Comp Plan ID, fiscal year, plan name, role group, creator. |
| CPM-006 | The system shall allow users to edit plan information (name and metadata). |
| CPM-007 | The system shall allow users to **clone** an existing Compensation Plan (or a specific version) as the starting point for new work, copying its full content — flavors, performance measures, payout tables, bonuses, tether, and policies — with new identifiers. |
| CPM-008 | The system shall route a clone per the Comp Plan ID rule: a **new** target ID creates a new Compensation Plan; an **existing** target ID initiates a new Proposal under that plan. |
| CPM-009 | The system shall allow users to archive a retired plan while retaining it for viewing and comparison. |
| CPM-010 | The system shall display each plan's status (Active / Archived) and its current published version in the Library. |

**Business Rules**
| Rule ID | Rule |
|---|---|
| BR-01 | A Compensation Plan is uniquely identified by Comp Plan ID + fiscal year. |
| BR-02 | A new Comp Plan ID creates a new Compensation Plan; an existing Comp Plan ID initiates a new Proposal within that plan (managed per §6.2). |
| BR-03 | A plan must have a plan name. |
| BR-04 | A plan may accumulate multiple proposals and published versions over time; their lifecycle is governed in §6.2. |
| BR-05 | Archived plans remain viewable and comparable and cannot be edited. |
| BR-06 | A clone is independent of its source: later edits to one do not affect the other. |

⚠ **To confirm:** whether a hard **Delete** is permitted, or only **Archive** (recommendation: archive only
for any plan that has versions). Also confirm whether a clone may target a **different fiscal year** (e.g.
cloning last year's plan to seed this year's).

**Prototype Screenshot**
```screenshot
Plan Library (Compensation Plans)
The plan list with search + filters and a Create Plan action; columns include Comp Plan ID, Name, Fiscal
Year, Status, Current Version, Creator, Last Updated.
```

## 6.2 Proposal Management

**Overview** A **Proposal** is a first-class object: a candidate version of a plan being authored before it
is published. A plan can have multiple proposals; publishing one makes it the plan's current version. This
module covers what happens to a proposal **once it exists**: editing its draft content, tracking its status,
publishing it, and viewing version history.

> **Scope boundary.** *How* a proposal is initiated — via the Comp Plan ID rule or by cloning existing
> content — belongs to **§6.1 Compensation Plan Management**. This module picks up at the point a proposal
> has been created and takes it through its lifecycle to a published version.

**User Workflow**
```flow
Open a plan with a proposal (created blank or seeded by a clone -> see §6.1)
Edit the proposal's draft content
Save draft (repeat)
Publish -> becomes the current version
View version history
```

**Functional Requirements**
| ID | Requirement |
|---|---|
| PM-001 | The system shall create every new Proposal — whether started blank or seeded by a clone (§6.1) — in **Draft** status. |
| PM-002 | The system shall allow users to edit the content of a Draft Proposal. |
| PM-003 | The system shall display each Proposal's status (Draft / Published). |
| PM-004 | The system shall identify the single current (active published) version for a plan. |
| PM-005 | The system shall allow users to publish a Proposal, making it the plan's current version and superseding the prior current version. |
| PM-006 | The system shall display a plan's version history (proposals and published versions). |
| PM-007 | The system shall prevent editing of a published Proposal/version. |

**Business Rules**
| Rule ID | Rule |
|---|---|
| BR-01 | A plan may contain multiple Proposals; only one is the current published version at a time. |
| BR-02 | Every Proposal starts as a Draft and follows the standard publish lifecycle. |
| BR-03 | Only Draft Proposals can be edited. |
| BR-04 | Publishing a Proposal makes it the current version and locks it (immutable). |
| BR-05 | Published versions are retained for history and comparison. |
| BR-06 | A Proposal must pass validation before it can be published. |

⚠ **To confirm:** publishing governance — in the full product, publishing is the outcome of an approval
workflow (future phase); for this phase an authorized user marks the Proposal as published/current.

**Prototype Screenshot**
```screenshot
Plan detail / proposal list
A plan's proposals and versions (e.g., Proposal 1 draft, Proposal 2, Version v2 current) with their status.
```

## 6.3 Flavor Management

**Overview** A **Flavor** represents a role variant within a compensation plan (e.g., A/B/C). Each proposal
can contain one or more flavors, each with its own performance measures, payout tables, bonuses, tether, and
policy notes. The Flavor Management module allows Compensation Designers to create, configure, and maintain
these role variants in a standardized manner.

**User Workflow**
```flow
Open the proposal in the Plan Builder
Add a Flavor and assign its role
Enter Flavor information (headcount, pay mix, new hire guarantee)
Configure the Flavor's measures / payout tables / bonuses / tether
Repeat per role variant
```

**Functional Requirements**
| ID | Requirement |
|---|---|
| FM-001 | The system shall allow users to create a new Flavor within a Proposal. |
| FM-002 | The system shall allow users to edit Flavor information (role, headcount, pay mix, new hire guarantee). |
| FM-003 | The system shall allow users to duplicate an existing Flavor. |
| FM-004 | The system shall allow users to delete a Flavor (removing its measures and payout tables). |
| FM-005 | The system shall organize/label Flavors in order (A, B, C…). |
| FM-006 | The system shall validate that Flavor names are unique within the Proposal. |
| FM-007 | The system shall display validation errors for incomplete Flavors. |
| FM-008 | The system shall prevent Proposal publication when required Flavor information is missing. |

**Business Rules**
| Rule ID | Rule |
|---|---|
| BR-01 | Every Proposal must contain at least one Flavor. |
| BR-02 | Every Flavor must contain at least one Performance Measure. |
| BR-03 | Flavor names must be unique within the Proposal. |
| BR-04 | Every Flavor must have a role. |
| BR-05 | Headcount and Pay Mix are optional at the Flavor level. |
| BR-06 | New Hire Guarantee is defined at the Flavor level (future flexibility for role-level). |
| BR-07 | Only Draft Proposals can be modified. |

⚠ **To confirm:** whether manual drag-**reordering** of flavors is required (today ordering follows the
A/B/C label order); and whether Base + Variable pay mix must total 100% (recommend a warning).

**Prototype Screenshot**
```screenshot
Plan Builder — flavor tabs
Tab strip (Overview, Overview by Flavor, then Flavor A / B / C + Add flavor) with the active flavor's role,
headcount, and pay mix.
```

## 6.4 Performance Measure Management

**Overview** A **Performance Measure** defines what a person is paid on and how much it counts. Each flavor
has one or more measures (labelled M1/M2/M3…), each drawn from a standardized catalog, weighted, and linked
to a payout table. This module lets designers configure and validate a flavor's measures.

**User Workflow**
```flow
Open a Flavor
Add a measure (M1)
Select the performance measure from the catalog -> system pay measure auto-fills
Enter VCT weight %, performance period, payout frequency
Reference a Payout Table setup for the measure (see 6.5)
Add M2, M3 until weights total 100%
```

**Functional Requirements**
| ID | Requirement |
|---|---|
| PMM-001 | The system shall allow users to create a Performance Measure within a Flavor (labelled M1/M2/M3…). |
| PMM-002 | The system shall allow users to edit a Performance Measure. |
| PMM-003 | The system shall allow users to delete a Performance Measure (removing its linked payout table). |
| PMM-004 | The system shall allow users to select the performance-measure value from a standardized catalog and shall auto-populate the corresponding system pay measure. |
| PMM-005 | The system shall allow users to set the VCT weight, performance period, and payout frequency for each measure. |
| PMM-006 | The system shall link each Performance Measure to exactly one Payout Table. |
| PMM-007 | The system shall validate that the VCT weights across a Flavor's measures total 100%. |
| PMM-008 | The system shall display validation errors when a Flavor's measures are incomplete or do not total 100%. |

**Business Rules**
| Rule ID | Rule |
|---|---|
| BR-01 | Every Flavor must contain at least one Performance Measure. |
| BR-02 | The VCT weights across a Flavor's measures must total 100%. |
| BR-03 | The performance-measure value must be chosen from the standardized catalog. |
| BR-04 | Each Performance Measure references exactly one Payout Table. |
| BR-05 | Performance period ∈ {Monthly, Quarterly, Annual}; payout frequency ∈ {Monthly, Quarterly, Annual, Quarterly (YTD)}. |

⚠ **To confirm:** the maximum measures per flavor (prototype allows 4); and whether **Coverage** (the
products/scope a measure applies to, today expressed by the chosen catalog value e.g. "…for Assigned DX
Products") should be captured as a separate structured field.

**Prototype Screenshot**
```screenshot
Measures & Weightings
A flavor's measures table — M1/M2/M3 with performance measure, system pay measure, VCT weight %, performance
period, payout frequency — beside its payout tables.
```

## 6.5 Payout Table Management

**Overview** A **Payout Table** converts attainment against quota into a payout. There is **one standardized
format** — a **quota band set** (quota-size columns) with an **attainment tier set per band** (attainment
rows), a payout multiplier (**xPCR**) per tier and an **MCR** per band, plus a pay curve, cap, and threshold.
(There are no separate table "types": a target-% table is simply a band set whose bands are percentage
intervals, and an attainment-only table is a single 0→∞ band.) Payout tables are defined **once as reusable
setups in the Payout Tables library**, then **referenced** from a measure in a plan: the setup is **copied
in** as a starting point, and the designer names it and adjusts values as needed.

**User Workflow**
```flow
(Library) Define a reusable Payout Table setup — quota band set + per-band attainment tier set + xPCR/MCR + pay curve/cap/threshold
(Plan) For a measure, reference a Payout Table setup — it is copied into the plan
Name the plan's payout table (defaults to the setup name)
Adjust xPCR per tier / MCR per band / pay curve / threshold / cap as needed
Add a per-plan note if useful
```

**Functional Requirements**
| ID | Requirement |
|---|---|
| PT-001 | The system shall maintain a reusable **Payout Tables library** of named payout-table setups (see Appendix A.7). |
| PT-002 | The system shall build every payout table in one standardized format: a quota band set + an attainment tier set per band, with an xPCR per tier and an MCR per band, plus pay curve, cap, and threshold — with **no separate table types**. |
| PT-003 | The system shall let users create a measure's payout table by **referencing** a library setup, copying the setup's content into the plan as a starting point. |
| PT-004 | The system shall keep the plan's copy **independent** of the library setup — local edits do not change the library, and later library edits do not change existing plans. |
| PT-005 | The system shall allow users to **name** a plan's payout table (defaulting to the referenced setup's name). |
| PT-006 | The system shall allow users to edit the copy's payout multipliers (xPCR per tier), MCR per band, pay curve, cap, and threshold; the band set + tier structure stay as inherited from the setup. |
| PT-007 | The system shall display a payout table as a matrix (quota bands = columns, attainment tiers = rows, xPCR in cells, MCR row). |
| PT-008 | The system shall allow an optional **per-plan note** on each payout table (recorded on the plan's table, not on the library setup). |
| PT-009 | The system shall link each Payout Table to its Performance Measure. |

**Business Rules**
| Rule ID | Rule |
|---|---|
| BR-01 | There is a single standardized payout-table format; no separate "table types". |
| BR-02 | Quota bands and attainment tiers are drawn from reusable, centrally-maintained sets. |
| BR-03 | A plan's payout table is a **copy** of a library setup and is independent of it thereafter. |
| BR-04 | Quota bands are ranges [from, to); attainment tiers are ranges (from, to]; the top tier may be open-ended. |
| BR-05 | Payout cap and entry threshold are optional; the note is recorded per plan (not on the library setup). |
| BR-06 | Each Payout Table is linked to a Performance Measure (validated at submit — see 6.9). |

**Prototype Screenshot**
```screenshot
Payout Tables library + a plan's payout table
The Payout Tables config screen (reusable setups); and a plan measure's payout table referencing a setup,
shown as a matrix of quota bands (columns) x attainment tiers (rows) with editable xPCR cells and an MCR row.
```

## 6.6 Bonus & Tether Management

**Overview** **Bonuses** are incentives that sit alongside the core measures; a **Tether** is a guardrail
that caps or gates payout based on another measure's attainment (e.g. "if renewal attainment < 80%, cap all
payouts at 100%"). Both are configured per flavor, along with free-text **Key Policies**.

**User Workflow**
```flow
Open a Flavor
Add a bonus (choose type, enter details and max)
Enter the tether description
Enter key policies
```

**Functional Requirements**
| ID | Requirement |
|---|---|
| BT-001 | The system shall allow users to add, edit, and remove Bonuses per Flavor (up to five). |
| BT-002 | The system shall allow users to select a Bonus type from a standard list and enter payout details and a maximum payout. |
| BT-003 | The system shall allow users to enter a Tether description per Flavor. |
| BT-004 | The system shall allow users to enter Key Policies per Flavor. |

**Business Rules**
| Rule ID | Rule |
|---|---|
| BR-01 | Bonuses and Tether are configured per Flavor. |
| BR-02 | Bonuses are optional; a Flavor may have up to five. |
| BR-03 | Bonus type is chosen from a standard list; payout details and maximum payout are free text (this phase). |
| BR-04 | Tether and Key Policies are free text (this phase). |

⚠ **To confirm:** whether Bonuses/Tether should become **structured** (amounts, %, trigger measure,
threshold) rather than free text in a later phase.

**Prototype Screenshot**
```screenshot
Bonuses, Tether & Key Policies (within a flavor)
The per-flavor Bonuses list (type / details / max) and the Tether and Key Policies free-text sections.
```

## 6.7 Draft & Version Management

**Overview** This module covers how work is saved and how versions are created and retained. Editing happens
on a **draft** proposal; **publishing** turns it into an immutable **version** that becomes the plan's
**current version**; prior versions are kept for history and comparison.

**Version terminology (business language)**
| Term | Plain-language meaning | Example |
|---|---|---|
| Draft / Working Version | An editable proposal being built or revised | "Proposal 3" (draft) |
| Published Version | A finalized, locked version | "Version v2" |
| Current Version | The single active published version of a plan | A01 · FY27 → Version v2 |
| Historical Version | A previously published version, kept for comparison | A01 · FY26 → Version v1 |
| Amendment (future) | A change to an already-in-effect published plan mid-cycle | future consideration |

**User Workflow**
```flow
Edit a proposal (draft)
Save draft
Publish -> current version (locked)
View version history
Create a new proposal from a historical version (to revise / "restore")
```

**Functional Requirements**
| ID | Requirement |
|---|---|
| VM-001 | The system shall allow users to save a Draft (manual save). |
| VM-002 | The system shall allow users to publish a Proposal as the plan's current version. |
| VM-003 | The system shall display a plan's version history (proposals and published versions). |
| VM-004 | The system shall allow users to create a new Proposal seeded from a historical version (to revise/restore). |
| VM-005 | The system shall lock published versions from editing. |

**Business Rules**
| Rule ID | Rule |
|---|---|
| BR-01 | Editing occurs only on Draft proposals. |
| BR-02 | Publishing locks the version (immutable) and makes it the current version. |
| BR-03 | Version numbers increment per plan and fiscal year. |
| BR-04 | Historical versions are retained and available for comparison. |

⚠ **To confirm:** **Auto Save** is not in the current prototype (save is manual) — confirm if desired.
**Restore** is achieved by creating a new proposal from a historical version (no destructive in-place restore).

**Prototype Screenshot**
```screenshot
Builder save/publish + version history
The builder header with Save draft and Publish; a plan's version history list (proposals + published versions).
```

## 6.8 Compensation Plan Comparison

**Overview** Comparison lets users see how one plan/proposal/version/flavor differs from another, **at any
time**, to support review and change management. It is a **standalone capability**, not a step in the
authoring sequence.

**User Workflow**
```flow
Open Compare
Select the left plan / proposal / version
Select the right plan / proposal / version
Review side-by-side, differences highlighted
Read the change summary; add notes
```

**Functional Requirements**
| ID | Requirement |
|---|---|
| CMP-001 | The system shall allow users to compare two Compensation Plans. |
| CMP-002 | The system shall allow users to compare two Proposals. |
| CMP-003 | The system shall allow users to compare two Versions of the same plan (e.g. year-over-year). |
| CMP-004 | The system shall display flavor-level differences within a comparison. |
| CMP-005 | The system shall highlight added / removed / changed values. |
| CMP-006 | The system shall provide difference filtering (show only differences) and a change summary. |
| CMP-007 | The system shall allow users to capture per-plan comparison notes. |

**Business Rules**
| Rule ID | Rule |
|---|---|
| BR-01 | Comparison is available at any time as a standalone feature. |
| BR-02 | Comparison is at the version level; a version bundles all its flavors. |
| BR-03 | Differences are highlighted; unchanged fields are shown for context. |

⚠ **To confirm:** whether a dedicated single-version **flavor-vs-flavor** view (two flavors of the same plan
compared directly) is required.

**Prototype Screenshot**
```screenshot
Compare
Two pickers (left/right); a side-by-side grid grouped into Plan information and per-flavor sections;
differences highlighted; a change-summary panel; per-plan notes.
```

## 6.9 Common Features

**Overview** Cross-cutting capabilities used throughout the application: search, validation, and clear error
messaging.

**User Workflow**
```flow
Search the Plan Library / configuration lists
Edit content in the builder
System validates on save / before publish
Clear error messages guide correction
```

**Functional Requirements**
| ID | Requirement |
|---|---|
| CF-001 | The system shall provide search across the Plan Library (and configuration lists). |
| CF-002 | The system shall validate completeness/consistency (VCT total = 100% per flavor; required plan name and flavor role; measure↔payout linkage). |
| CF-003 | The system shall display clear, actionable error messages identifying what must be corrected. |
| CF-004 | The system shall prevent finalization/publication while validation errors remain. |

**Business Rules**
| Rule ID | Rule |
|---|---|
| BR-01 | Validation must pass before a proposal can be published. |
| BR-02 | Search and filtering are available wherever lists of plans/objects are shown. |

⚠ **To confirm:** **Comments** are **not in Phase 1** (collaboration/approval is a future phase); **Export**
(e.g. a plan PDF/print summary) is **not in the current prototype** — confirm if required this phase.

**Prototype Screenshot**
```screenshot
Validation & search examples
The library search/filter bar; a builder validation message (e.g. a flavor's VCT total not equal to 100%).
```

<PAGEBREAK>

# Appendix A — Data Captured (business view)

This appendix lists the information each object holds, in business terms, so stakeholders can confirm no
important data has been overlooked. It complements §6 (which describes what the system *does*). Field
legend: **Required** = must be entered · **Optional** = may be left blank · **System** = set automatically by
the system · **Auto** = derived from another selection. (Technical data types, keys, and relationships live
in the separate Engineering Design Specification.)

## A.1 Compensation Plan
| Field | Required? | Allowed values / format | Business meaning |
|---|---|---|---|
| Comp Plan ID | Required | e.g. `A01` | Plan identifier (unique with fiscal year) |
| Fiscal Year | Required | `FY26`, `FY27`, … | The plan year |
| Plan Name | Required | text | Business name of the plan |
| Role Group | Optional | text | Segment/group the plan serves (e.g. Commercial AE) |
| Creator | Required | person | Person accountable for the plan (its creator/designer) |
| Status | System | Active / Archived | Lifecycle state |
| Current Version | System | reference to a version | The active published version |

## A.2 Proposal / Version
| Field | Required? | Allowed values / format | Business meaning |
|---|---|---|---|
| Label | System | "Proposal N" (draft) / "Version vN" (published) | Human-readable label |
| Version Number | System | integer | Increments per plan + fiscal year |
| Status | System | Draft / Published | Whether it is editable or locked |
| Source | Optional | reference | The version/proposal it was cloned or derived from |

## A.3 Flavor
| Field | Required? | Allowed values / format | Business meaning |
|---|---|---|---|
| Flavor Label | System | A / B / C… | Role-variant label (in order) |
| Role | Required | text | The role this variant covers |
| Headcount | Optional | number ≥ 0 | Population size |
| Pay Mix — Base / Variable | Optional | 0–100 each (should total 100) | Base vs. variable pay split |
| New Hire Guarantee | Optional | Yes / No | Guarantee arrangement (flavor-level today) |
| Key Policies | Optional | text | Free-text policy notes |

## A.4 Performance Measure
| Field | Required? | Allowed values / format | Business meaning |
|---|---|---|---|
| Sequence | System | M1 / M2 / M3… | Order/label of the measure |
| Performance Measure Value | Required | chosen from the standardized catalog | What the person is measured on (e.g. "Net New ARR for Assigned DX Products") |
| System Pay Measure | Auto | derived from the catalog mapping | The internal pay metric it maps to |
| VCT Weight % | Required | integer %; a flavor's measures must total 100% | How much this measure counts |
| Performance Period | Optional | Monthly / Quarterly / Annual | Measurement window |
| Payout Frequency | Optional | Monthly / Quarterly / Annual / Quarterly (YTD) | How often it pays |
| Linked Payout Table | System (1:1) | reference | The measure's payout schedule |

## A.5 Payout Table (a plan's copy, per measure)
| Field | Required? | Allowed values / format | Business meaning |
|---|---|---|---|
| Name | Auto (editable) | text (defaults to the referenced setup's name) | Table name within the plan |
| Source Setup | System | reference to a library setup | Which reusable setup it was copied from |
| Quota Band Set | Required | reusable set | The quota-size columns |
| Attainment Tier Set (per band) | Required | reusable set | The attainment rows |
| Payout Multiplier (xPCR, per tier) | Required | e.g. `1.50x` | Payout at each tier |
| MCR | Optional | % | Minimum commission rate (per band) |
| Pay Curve | Optional | Linear / Stepped | How payout accrues across tiers |
| Payout Cap | Optional | % | Maximum payout |
| Threshold | Optional | number | Minimum attainment required to earn any payout |
| Note | Optional | text | Per-plan note (not stored on the library setup) |

*One standardized format — no table "types". Ranges: quota bands are `[from, to)`; attainment tiers are
`(from, to]`; the top tier may be open-ended.*

## A.6 Bonus (per flavor, up to 5) / Tether (per flavor)
| Field | Required? | Allowed values / format | Business meaning |
|---|---|---|---|
| Bonus — Type | Optional | standard list (Lead Referral / Linearity / M1 & M2 Achievement) | Kind of bonus |
| Bonus — Payout Details | Optional | text | How the bonus pays |
| Bonus — Max Payout | Optional | text | Bonus cap |
| Tether — Description | Optional | text | Guardrail that caps/gates payout based on another measure |

## A.7 Reusable Libraries (Administrator-maintained)
| Library | Holds | Business meaning |
|---|---|---|
| Quota Band Set | Named list of quota-size upper bounds | Standardized quota-band columns reused across payout tables |
| Attainment Tier Set | Named list of attainment upper bounds | Standardized attainment rows reused across payout tables |
| Performance Measure Catalog | Performance-measure value → system pay measure | The allowed measures and their internal pay-metric mapping |
| Payout Table Setup | A named standardized payout table (quota band set + per-band attainment tier set + xPCR/MCR + pay curve/cap/threshold) | Prebuilt payout tables referenced (copied) into plans |
