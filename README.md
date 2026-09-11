# cloud-itonami-isco-7215

Open Occupation Blueprint for **ISCO-08 7215**: Riggers and Cable Splicers.

This repository designs a forkable OSS business for a rigging-crew job-site coordination service: a job-site scheduling/logistics coordination robot manages work-record logging, crew scheduling, safety-concern flagging and rigging-equipment supply-order coordination under a governor-gated actor, so the practice keeps its own operating records instead of renting a closed crew-dispatch SaaS.

**This actor coordinates JOB-SITE SCHEDULING/LOGISTICS ONLY — it never performs rigging or cable-splicing work itself.** Riggers and cable splicers prepare loads for crane/hoist lifting and splice/inspect wire rope, one of the highest dropped-load and cable-failure-risk trades; the actor's closed op-allowlist contains no op that directly finalizes a load-rigging/lift-readiness certification decision or overrides a site-safety officer's judgment. Any proposal that attempts either is a hard, permanent block, never overridable by human approval.

**Maturity: `:implemented`.** `src/riggercoord/` implements the
`RiggerCoordActor` as a `langgraph.graph/state-graph`
(`riggercoord.actor`) wired to a `Rigging Coordination Advisor`
(`riggercoord.advisor`) and an independent `RiggerCoordGovernor`
(`riggercoord.governor`), following the itonami actor pattern
(ADR-2607011000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok? true) +-> :request-approval (:escalate? true, human-in-the-loop
interrupt) +-> :hold (:hard? true)`. 22 tests / 52 assertions green
(`kbb -M:test`).

HARD invariants (always `:hold`, never overridable): the job site
must be independently verified/registered before any action; a
referenced worker must be a registered crew member belonging to that
site; `:effect` must be `:propose` only (no hardware dispatch, no
rigging or cable-splicing work performed); the closed op-allowlist is
enforced (no op in the allowlist finalizes load-rigging/lift-readiness
certification or overrides site-safety authority); and any proposal
that attempts to directly finalize a load-rigging/lift-readiness
certification decision or override a site-safety officer's judgment
is a hard, **permanent** block — detected as finalization/execution
action phrases (never bare nouns like "load"/"cable"/"rigging", which
are ordinary vocabulary for this domain and must not false-trip the
guard).

Always-escalate ops (human sign-off regardless of confidence, mapping
this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (every surfaced cable-condition/load-hazard/
equipment-condition concern) and `:coordinate-supply-order` above the
registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a job-site scheduling/logistics coordination robot performs work-record logging, crew-schedule proposals, safety-concern surfacing and rigging-equipment supply-order coordination under an actor that proposes
actions and an independent **Rigging Coordination Governor** that gates them. The governor never
dispatches hardware itself, never performs rigging or cable-splicing work, and never overrides a site-safety officer's judgment; `:high`/`:safety-critical` actions (such as a safety-concern flag or an above-threshold supply order) require human sign-off.

## Core Contract

```text
job-site roster + crew roster + job-site schedule
        |
        v
Rigging Coordination Advisor -> RiggerCoordGovernor -> log record/schedule/order, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize a load-rigging/lift-readiness certification decision,
override a site-safety officer's judgment, suppress an operating
record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7215`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
