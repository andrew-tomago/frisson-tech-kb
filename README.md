# Frisson Tech Knowledge Base

Canonical source for shared team documentation across all Frisson Tech OpenClaw bot repos.

## Structure

- `onboarding/` — new agent/contributor guides
- `playbooks/` — operational runbooks
- `reference/` — API, CLI, and config reference notes

## How this works

Docs here are the **single source of truth**. Bot repos (`felicity-macos-claw`, `felix-ubuntu-claw`, etc.) mirror this content under `shared-docs/` via `git subtree`.

### Pulling updates into a bot repo

```bash
bash scripts/sync-shared-docs-pull.sh
```

### Proposing changes to shared docs

1. Edit the doc in your bot repo's `shared-docs/` directory
2. Run `bash scripts/sync-shared-docs-push.sh` to push the change upstream
3. Open a PR on this repo for cross-review
4. Once merged, other bot repos pull the update

### CI divergence check

Each bot repo's CI verifies `shared-docs/.version` matches the latest canonical commit. A diverged bot repo will fail CI until it pulls.

## Contributing

All changes to shared docs go through a PR on this repo with at least one cross-bot review before merge. Machine-specific docs (system specs, scheduler config, identity files) belong in the individual bot repos — not here.
