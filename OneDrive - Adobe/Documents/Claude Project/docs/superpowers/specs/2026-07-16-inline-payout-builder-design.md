# Inline Payout Table Builder Design

**Date:** 2026-07-16

**Status:** Approved

## Goal

Remove the standalone Payout Tables configuration library and let a plan designer define each performance measure's payout table directly in the Plan Builder.

## Scope

This change affects the single-file prototype, its seed/configuration model, and its supporting documentation. It does not redesign plan versioning, approvals, comparison, or the standardized payout-table data format.

## User Experience

Each performance measure remains linked one-to-one with a payout table. In the measure's inline builder card, the designer selects:

- quota band set;
- pay curve;
- threshold;
- cap.

Selecting a quota band set creates the table's bands. Within each band, the designer selects an attainment tier set and enters xPCR values per tier plus MCR for the band. An optional note remains on that individual payout table within the proposal flavor.

The Performance Measures configuration screen remains a simple mapping from performance-measure value to system pay measure. It does not own payout structure. The standalone Payout Tables configuration screen and reusable payout profiles are removed.

## Data Model

The existing per-flavor ownership model remains authoritative:

```text
flavor.measures[n].payoutTableId -> flavor.payoutTables[n].id
```

Each table retains the standardized shape:

```text
{
  id, title, bandSetId, payCurveType, threshold, globalVctCap, note,
  prorateVct,
  thresholds[{ id, label, description, tierSetId, mcr,
               tiers[{ id, attainmentFrom, attainmentTo, xPCR, mcr }] }]
}
```

`sourceSetupId` and `sourceSetupName` are removed because tables no longer originate from reusable profiles. `prorateVct` remains preserved because it is completed later during Release Validation.

## Configuration and Persistence

`quotaBandSets`, `attainmentTierSets`, and `performanceMeasures` remain persisted in localStorage. `payoutSetups` is removed from runtime state, normalization, persistence, CRUD, routing, and screen registration.

Previously persisted `payoutSetups` data is ignored. Existing persisted band, tier, and performance-measure identifiers remain stable so references continue to resolve.

## Builder Behavior and Data Flow

Adding a performance measure creates and links one blank payout table. Removing the measure removes its table. The measure and payout controls render as one inline unit.

Before a structural selection causes a re-render, current DOM edits are synchronized into `builderWorkingContent` so sibling edits are not lost.

- Changing the quota band set rebuilds the table's bands from that set and clears each band's tier selection and tiers.
- Changing a band's attainment tier set rebuilds that band's tiers from the selected set and leaves xPCR values blank for entry.
- xPCR, MCR, curve, threshold, cap, title, and note round-trip through the existing collection/synchronization path.
- Read-only Overview, approval, and comparison surfaces continue to render the same matrix representation.

## Seed Data

Demo content no longer copies from a user-facing payout library. Internal seed-only factory functions create fresh standardized payout tables by demo name. Each seeded measure receives its own table and fresh table, band, and tier identifiers.

## Validation

Submission continues to require, per flavor:

- every performance measure links to an existing payout table;
- every payout table is linked by at least one measure;
- all other existing plan and VCT validation rules still pass.

The one-to-one builder behavior normally guarantees these linkage rules, while validation protects migrated or malformed data.

## Documentation

Update `CLAUDE.md`, `docs/progress.md`, and the business PRD source to describe inline payout definition and remove the reusable-library workflow. Regenerate the tracked PRD `.docx` from the updated source.

## Verification

Because this is a self-contained throwaway browser prototype with no automated test framework, verification uses focused static checks plus browser regression testing:

1. Confirm there are no remaining Payout Tables nav/screen/library references.
2. Confirm Performance Measures configuration still contains only value and system-pay-measure fields.
3. Exercise add/remove measure and the one-to-one payout lifecycle.
4. Exercise band-set and tier-set changes, including rebuild behavior and preservation of sibling edits.
5. Save and reopen a proposal; confirm curve, threshold, cap, xPCR, MCR, and note round-trip.
6. Confirm all seeded versions pass `validateVersionContent`.
7. Check Overview, approval, and comparison matrix rendering.
8. Confirm no browser console errors.

## Alternatives Considered

1. **Inline per-measure definition (selected).** Matches the final clarified requirement and removes unnecessary configuration indirection.
2. **Reusable library plus builder overrides.** Supports reuse but contradicts the explicit removal of the separate payout-table configuration.
3. **Payout fields on Performance Measures configuration.** Centralizes defaults but contradicts the clarification that these choices belong in the Plan Builder.

## Non-Goals and Known Limitations

- No unrelated cleanup of legacy builder functions.
- No new backend or persistence layer.
- No change to the stepped/marginal payout calculation.
- `payCurveType` remains descriptive unless separately requested; the current calculation does not branch on Linear versus Stepped.
