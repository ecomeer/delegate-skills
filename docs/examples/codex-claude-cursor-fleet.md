# Codex + Claude + Cursor fleet preset

This preset is optimized for a workflow where Codex is the primary implementer, Cursor owns visual/UI implementation, and Claude provides read-only review and audit lanes.

## Lane map

| Lane | Implementer | Access | Best fit |
| --- | --- | --- | --- |
| `feature` | Codex | write | Product features, backend/admin work, refactors, integrations, migrations |
| `tests` | Codex | write | Test additions, regression fixes, CI-focused implementation |
| `ui` | Cursor | write | Frontend, responsive/mobile UI, visual polish, interaction work |
| `review` | Claude | read-only | Diff review, architecture checks, correctness/security second opinion |
| `audit` | Claude | read-only | Broader repository audits, missing-flow checks, risk discovery |

The JSON intentionally omits model and effort dials. That keeps the preset portable, inherits each CLI's configured defaults, and avoids hard-coding model identifiers that may become stale. The only explicit safety dial is `readOnly: true` for Claude review/audit lanes.

Preset file: [`codex-claude-cursor-fleet.json`](codex-claude-cursor-fleet.json)

## Validate and install

Use `delegate-setup` to review the lane table and full JSON before writing it. The setup skill must still get explicit approval before it writes configuration.

To validate the file manually from the repository:

```bash
node skills/delegate-setup/scripts/config.mjs validate docs/examples/codex-claude-cursor-fleet.json
```

To install globally after approval:

```bash
node skills/delegate-setup/scripts/config.mjs write --scope global docs/examples/codex-claude-cursor-fleet.json
```

To install for only the current repository after approval:

```bash
node skills/delegate-setup/scripts/config.mjs write --scope project --cwd "$PWD" docs/examples/codex-claude-cursor-fleet.json
```

## Recommended operating loop

1. Route normal implementation to `feature` or `tests` with `codex-delegate`.
2. Route visual/mobile/frontend implementation to `ui` with `cursor-delegate`.
3. After implementation, route the resulting diff to `review` with `claude-delegate`.
4. For larger releases or risky changes, use the `audit` lane before landing.
5. The orchestrator re-runs the project's real gates and commits only verified work.

This keeps implementation and judgment separate: Codex/Cursor produce the diff, Claude challenges it, and the orchestrator owns the final merge decision.
