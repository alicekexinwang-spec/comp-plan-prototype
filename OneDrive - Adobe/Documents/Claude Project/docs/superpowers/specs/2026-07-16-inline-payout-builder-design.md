# Performance Measure Quota Configuration Design

**Date:** 2026-07-16

**Status:** Approved

## Goal

Remove the standalone Payout Table Setup configuration and make each reusable Performance Measure configuration own its quota setup. Keep the proposal payout-table editor focused on attainment tiers, xPCR, MCR, and its existing optional note.

## Scope

This change affects the single-file prototype, configuration persistence, seeded data, and supporting documentation. It does not redesign plan versioning, approvals, comparison, or payout calculations.

## User Experience

The Performance Measures configuration screen retains the existing performance-measure value and system-pay-measure fields and adds, inline on the same configuration entry:

- Quota Band Set;
- Pay Curve;
- Threshold;
- Cap.

The standalone Payout Tables configuration screen is removed.

In the Plan Builder, selecting a performance measure applies that measure configuration to the proposal measure and generates the quota bands. The linked payout-table editor does not repeat editable quota-band, curve, threshold, or cap controls. For each generated quota band, the designer selects an Attainment Tier Set, enters xPCR for each tier, and enters MCR for the band. The existing optional payout-table note remains editable.

## Data Ownership

Each reusable Performance Measure configuration entry has this normalized shape:

```text
{
  id,
  value,
  systemPayMeasure,
  bandSetId,
  payCurveType,
  threshold,
  globalVctCap
}
```

When selected in the builder, these quota settings are copied onto the proposal's performance-measure instance. This makes the proposal a stable snapshot: later edits to reusable configuration do not silently change an existing proposal.

The proposal measure therefore extends its existing fields with:

```text
{ bandSetId, payCurveType, threshold, globalVctCap }
```

The linked payout table retains its identity, optional note, Release Validation value, generated band rows, and tier details:

```text
{
  id,
  title,
  note,
  prorateVct,
  thresholds[{
    id,
    label,
    description,
    tierSetId,
    mcr,
    tiers[{ id, attainmentFrom, attainmentTo, xPCR, mcr }]
  }]
}
```

Quota ownership fields are removed from the payout table: `bandSetId`, `payCurveType`, `threshold`, and `globalVctCap`. Library provenance fields `sourceSetupId` and `sourceSetupName` are also removed because payout profiles no longer exist.

## Configuration and Persistence

`quotaBandSets`, `attainmentTierSets`, and the extended `performanceMeasures` remain persisted in localStorage. `payoutSetups` is removed from runtime state, normalization, persistence, CRUD, routing, and screen registration.

Existing persisted Performance Measure entries normalize missing quota fields to blank values, with Pay Curve defaulting to `Linear`. Existing band, tier, and measure identifiers remain stable.

## Builder Data Flow

Each builder performance measure remains linked one-to-one with a payout table.

When a measure value is selected or changed:

1. Resolve its reusable Performance Measure configuration.
2. Copy `systemPayMeasure`, `bandSetId`, `payCurveType`, `threshold`, and `globalVctCap` onto the proposal measure.
3. Rebuild the linked payout table's quota-band rows from the configured Quota Band Set.
4. Initialize each generated band with no Attainment Tier Set and no tiers.
5. Preserve the payout table's identity, title, optional note, and `prorateVct`.

Changing the reusable configuration later does not modify existing proposal measures. Re-selecting or changing the measure in the builder reapplies the current configuration and intentionally rebuilds its band/tier structure.

Within the payout editor, changing a band's Attainment Tier Set rebuilds only that band's tiers. xPCR, MCR, and the optional note round-trip through the existing DOM synchronization path.

Read-only Overview, approval, and comparison rendering resolves quota metadata from the linked performance measure and tier details from the payout table.

## Seed Data

Seeded reusable Performance Measure entries include the quota settings needed by demo measures. Seeded proposal measures carry snapshot quota settings, and their linked payout tables contain the corresponding generated bands and tier details. No user-facing payout-profile library or seed library is retained.

## Validation

Existing plan and VCT rules remain. Submission also continues to require, per flavor:

- every performance measure links to an existing payout table;
- every payout table is linked by at least one performance measure;
- selected performance measures have a Quota Band Set;
- each generated quota band has an Attainment Tier Set and tier details as required by the existing payout validation.

## Documentation

Update `CLAUDE.md`, `docs/progress.md`, and the business PRD source to describe Performance Measure quota ownership and remove the Payout Table Setup workflow. Regenerate the tracked PRD `.docx` from the updated source.

## Verification

This is a self-contained browser prototype without an automated test framework. Verification uses focused static checks and browser regression testing:

1. Confirm no Payout Table Setup navigation, screen, state, CRUD, persistence, or provenance references remain.
2. Confirm each Performance Measure configuration entry exposes value, system pay measure, quota band set, pay curve, threshold, and cap.
3. Confirm legacy persisted Performance Measure entries normalize safely.
4. Select a performance measure in the builder and confirm its configured quota settings are copied and its quota bands are generated.
5. Confirm the payout editor exposes only Attainment Tier Set, xPCR, MCR, and the optional note.
6. Confirm changing a tier set rebuilds only that band's tiers.
7. Save and reopen a proposal; confirm measure quota settings and payout tier details round-trip.
8. Confirm existing proposals are not silently rewritten when reusable configuration changes.
9. Confirm all seeded versions pass validation.
10. Check Overview, approval, and comparison rendering plus browser console output.

## Alternatives Considered

1. **Snapshot quota configuration onto proposal measures (selected).** Preserves existing proposal behavior and avoids runtime coupling to mutable global configuration.
2. **Resolve quota configuration live from reusable entries.** Uses less duplicated data but would silently change existing proposals when administrators edit configuration.
3. **Keep quota settings on payout tables.** Requires fewer data-path changes but fails the explicit ownership requirement and leaves duplicated quota-related fields.

## Constraints and Non-Goals

- Make the smallest changes that satisfy the ownership change.
- Reuse existing selectors, inputs, payout tier editor, and styles.
- Do not refactor unrelated legacy code.
- Preserve existing approval, comparison, seeding, and Release Validation behavior.
- Keep the optional payout-table note editable.
- Do not change xPCR or MCR requirements.
- `payCurveType` remains descriptive unless separately requested; the current payout calculation does not branch on Linear versus Stepped.
