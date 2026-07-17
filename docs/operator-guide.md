# Operator Guide

## First Deployment

1. Define the operator's job-site roster and crew-registration process.
2. Define consent and purpose categories for logged work records.
3. Run synthetic coordination cases (work-record logging, scheduling, safety-concern flags, supply orders).
4. Enable human-reviewed sign-off for `:high`/`:safety-critical` actions — every safety-concern flag and every above-threshold supply order.
5. Measure coordination outcomes and audit coverage.

## Minimum Production Controls

- consent and disclosure log
- safety-critical escalation path (cable-condition, load-hazard and equipment-condition concerns always reach a human)
- provenance for all job sites and crew members before any coordination action
- human review for high-risk cases
- audit export for all gated actions

## Scope Boundary (Mandatory)

This actor coordinates job-site scheduling and logistics ONLY. Operators
must not wire this actor's output into any system that would let it
directly finalize a load-rigging/lift-readiness certification decision
or override a site-safety officer's judgment — the governor's closed
op-allowlist and scope-exclusion rule are the last line of defense,
not the only one; operator-side integrations must not create a path
around them.

## Certification

Certified operators must prove that the governor gates every
safety-critical robot action, that safety-critical risks escalate to
humans, and that no integration allows this actor to finalize
load-rigging/lift-readiness certification or override site-safety
authority.
