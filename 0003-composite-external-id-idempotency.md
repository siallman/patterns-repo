# 0003. Composite external ID for ingestion idempotency

- **Date:** YYYY-MM-DD  *(backfill — set to the actual decision date)*
- **Status:** Accepted
- **Engagement:** <client — multi-brand retail/loyalty>
- **Area:** Ingestion / data modelling

## Decision

Key ingested subscription records on a **composite external ID** in
`loyalty|banner|name` format, so re-ingesting the same logical record upserts
in place rather than creating a duplicate.

## Context / driver

In a multi-brand / multi-banner loyalty programme, no single source field is
both stable and unique across the estate — the same loyalty member can appear
under different banners, and a raw name or channel value collides across brands.
Ingestion needs an idempotency key that is deterministic (reproducible from the
source fields on every run) and unique at the grain being written, so that batch
re-runs and re-deliveries don't fan out into duplicate subscription rows.
Composing the key from `loyalty` + `banner` + `name` gives a value that is
stable per logical record and distinct across banners.

## Alternatives considered

- **Single natural key (e.g. loyalty ID alone)** — not unique once the same
  member spans banners; would merge or collide records that should stay
  distinct. Rejected.
- **Generated surrogate key** — breaks idempotency: a fresh surrogate on each
  ingest means re-runs create new rows instead of upserting. Rejected.

## Consequences

- Batch re-runs and redeliveries are safe — the composite key upserts.
- The delimiter (`|`) must not appear in the component values; if any component
  can contain it, it needs escaping or a safer separator to avoid ambiguous
  keys. Worth a validation check at ingest.
- Component fields must be normalized consistently (case, trimming) before
  concatenation, or the "same" record can produce two different keys.
- Pairs directly with the driver-table transform (0002): the composite ID is
  the stable anchor those unpivoted consent rows attach to.

## Generalisable?

Yes — "composite external IDs for idempotent ingestion in a multi-brand estate."
**Pattern doc #2** in the publish plan; running 0002 and 0003 through the
pipeline together proves the system works repeatably.
