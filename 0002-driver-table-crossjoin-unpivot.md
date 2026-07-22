# 0002. Driver-table cross-join to unpivot wide preference fields

- **Date:** YYYY-MM-DD  *(backfill — set to the actual decision date)*
- **Status:** Accepted
- **Engagement:** <client — multi-brand retail/loyalty>
- **Area:** Batch transforms / consent modelling

## Decision

Unpivot the wide preference source (one row per individual, one column per
preference) into the tall CommSubscriptionConsent shape by cross-joining the
source against a **Preference Definition driver DLO** — one row per preference —
rather than hand-coding a branch per preference.

## Context / driver

Source preference data arrives denormalized: one row per individual with N
opt-in columns (Email_Opt_In, SMS_Opt_In, Push_Opt_In, … per brand). The
canonical consent model needs the inverse shape — one row per
individual-per-subscription. With four brands today and more expected, a
transform that hard-codes each preference as its own branch would need editing
every time a brand or channel is added. Anchoring the preference set in a driver
table instead means **adding a preference is one new row, not a new branch in
the transform** — the whole point of the pattern.

## Alternatives considered

- **One UNION ALL branch per preference** — works, but the transform grows with
  every channel and every brand; brittle and high-maintenance. Rejected.
- **A direct cross/Cartesian join node** — cleanest if the transform editor
  exposes it, but that wasn't confirmed available in-org, so the build uses the
  constant-key method that works regardless (see below).

## Consequences

- The driver DLO (`Preference Definition`) carries `PreferenceCode`,
  `SubscriptionId`, `BrandId`, `SourceFieldName`, `Channel`, plus an
  `ActiveFlag` so preferences can be switched on/off without deleting rows.
  `SourceFieldName` is the link that resolves each definition row back to its
  wide source column.
- Cross join is implemented with the **constant-key trick**: add a Formula
  transform to each source setting `JoinKey = 1`, then an inner join on
  `JoinKey = JoinKey`, producing every individual × every preference definition.
  Swap in a native cross join if the editor later exposes one.
- Definition-side fields are carried forward under clear aliases (e.g.
  `def_SubscriptionId`) so downstream nodes can reference them unambiguously.
- Maintenance cost of onboarding a brand drops to data entry in the driver DLO.

## Generalisable?

Yes — flagship candidate. Abstract shape: "unpivot wide preference/consent
columns into a tall canonical model via a driver table cross-joined with a
constant key." This is **pattern doc #1** in the publish plan.
