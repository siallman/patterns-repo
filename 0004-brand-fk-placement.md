# 0004. Brand foreign key placed on CommSubscription

- **Date:** YYYY-MM-DD  *(backfill — set to the actual decision date)*
- **Status:** Accepted
- **Engagement:** <client — multi-brand retail/loyalty>
- **Area:** Consent modelling / multi-brand data modelling

## Decision

Carry the brand foreign key on **CommSubscription** — the subscription grain —
rather than on CommSubscriptionConsent or on Contact Point Consent.

## Context / driver

A subscription is inherently brand-scoped: a person subscribes to *Brand A's*
email programme, which is a different thing from *Brand B's* email programme
even on the same email address. That makes CommSubscription the natural home for
brand — it sits at exactly the grain where brand is a defining attribute.
Placing it there keeps consent records anchored to the brand-level source
individual and lets them bypass the identity-unification spine, so cross-brand
identity resolution never dissolves per-brand consent boundaries. Brand is
modelled as *rows* (subscriptions) rather than *columns*, so onboarding a brand
adds data, not schema.

## Alternatives considered

- **Brand FK on CommSubscriptionConsent** — wrong grain: consent is the
  permission *state* on a subscription; duplicating brand down at the consent
  record repeats it below where it's defined and invites drift between a
  subscription and its own consent rows. Rejected.
- **Brand FK on Contact Point Consent** — a contact point (an email address, a
  phone number) can be shared across brands; anchoring brand there conflates the
  channel identifier with the brand and breaks per-brand scoping. Rejected.

## Consequences

- Per-brand consent stays cleanly queryable and maps directly to brand-scoped
  activation (e.g. per-brand MCE business units).
- Consent boundaries survive cross-brand unification because they hang off the
  brand-scoped subscription, not the unified profile.
- Reinforces the separation established in 0001 between permission-state and
  engagement-events, now with brand pinned at the correct grain.

## Generalisable?

Yes — feeds the multi-brand consent pattern doc alongside 0001. Abstract shape:
"brand as a foreign key at the subscription grain, keeping consent brand-scoped
and independent of the identity-unification spine."
