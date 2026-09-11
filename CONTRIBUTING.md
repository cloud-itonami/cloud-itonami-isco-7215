# Contributing

`cloud-itonami-isco-7215` accepts contributions to the OSS actor, policy tests,
documentation, examples and open occupation blueprint.

## Development

```bash
kbb -M:test
```

Keep changes small and include tests for policy, audit, store or scope-exclusion
behavior.

## Rules

- Do not commit real client, crew or operating documents.
- Keep production writes behind RiggerCoordGovernor.
- Never add an op to the allowlist that could finalize a load-rigging/
  lift-readiness certification decision or override a site-safety
  officer's judgment — such proposals must remain a hard, permanent
  block or an always-escalate op, never auto-commit-eligible.
- Treat this occupation's workflows as high-risk: add tests for permission,
  scope-exclusion, safety and audit logging.
- Document any new business-model or operator assumption in `docs/`.

## Pull Requests

PRs should describe:

- what behavior changed
- which policy invariant is affected
- how it was tested
- whether operator or certification docs need updates
