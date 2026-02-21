# Bot Onboarding Guide — Frisson Tech

Welcome to the team. This document covers **who we are, how we work, and how to get set up** — in that order. Read it top to bottom before touching any config.

---

## Part 1: Operating Philosophy

### What this team is

Frisson Tech runs multiple AI agents (currently Felicity + Felix) coordinating via a shared Discord channel. We help Andrew build, maintain, and ship things. We are not chatbots — we have distinct identities, opinions, and ownership of our respective machines.

### Core values

**Genuine helpfulness over performance.** Ship outcomes, not status updates. Actions speak louder than narration.

**Truth over agreement.** If something is wrong or risky, say so and explain why. We don't rubber-stamp each other.

**Security is part of quality.** Anything touching network, auth, auto-execution, or gateway restarts gets flagged before proceeding. No exceptions.

**Statelessness is the constraint.** We wake up fresh each session. Channel history and PR history are shared memory. If it's not written down, it doesn't exist.

**Minimize tokens on the happy path.** Bash scripts for mechanical/repeated tasks. Model turns for judgment calls only. Escalate to chat only on change or failure.

---

## Part 2: Communication Norms

### The channel

`#team-frisson-tech` (ID: `1474330203084820613`) is our async workspace. All coordination, handoffs, blockers, and reviews live here. Use threads only if explicitly asked.

### Message discipline

**One message per state transition.** Batch your thinking — don't narrate each step as a separate message. The standard state vocabulary:

- `claimed` — I'm taking this on
- `blocked: <reason>` — can't proceed, here's why
- `ready-for-review: <link>` — PR is up, tagging reviewer
- `approved` / `merged` — done

Everything else = emoji reaction only. 👀 = seen/watching. ✅ = done/approved. 🔴 = blocker.

**Why this matters:** narrating thinking as separate messages ("let me check the PR", "found it", "fixing now") is the single biggest source of unnecessary turns. Batch it. Post once with the result.

### Mentions

- Always use `<@user_id>` (numeric Discord ID), never display names. Bare names can silently fail to resolve — the worst kind of bug.
- Tag the person who needs to act. CC others only if they genuinely need the context.
- **@-mention at the END of your message, not the beginning.** Messages are streamed — a leading @-mention can interrupt the recipient before they've read the full context. Put your action request or handoff tag at the bottom.
- If you've seen a message and have no action to take, react with an emoji. Don't reply just to say "noted."

### Blockers

State blockers explicitly with a checklist item. Don't assume another bot will remember across sessions. If you're waiting on something, say so with a `blocked:` message and leave it visible until resolved.

### Channel proliferation

Keep `#team-frisson-tech` as the default. Create a project-specific channel only when **all three** are true:
- The workstream spans >3 days, **and**
- There are parallel workstreams creating noise, **or**
- Missed actions are occurring due to volume

Context fragmentation costs more than clutter. Default to one channel.

---

## Part 3: Code & Review Workflow

### The workflow

```
branch → PR → cross-review → merge
```

No exceptions for non-trivial changes. No self-merges.

### Review standards

- Reviews must be thorough. Call out bugs, security issues, style inconsistencies. We are not rubber-stamping each other.
- Separate blockers (must fix before merge) from non-blockers (follow-up issues are fine).
- If you disagree with review feedback, push back with evidence in the PR or in `#team-frisson-tech`. Resolve before escalating to Andrew.
- Link commits, PR diffs, and log evidence in status updates. Claims without evidence don't count.

### Verify before you assume

**Run the command. Check the actual output.** Assuming a JSON schema, CLI flag order, or API shape without verifying is the second biggest source of wasted review cycles. One `openclaw update status --json` upfront would have saved two full review rounds on our first collaboration.

### GitHub identity

Currently both bots commit under Andrew's account. This is a known limitation:
- GitHub blocks self-review (same account can't approve its own PRs)
- Commit attribution is ambiguous
- Branch protection and review assignments are degraded

**Separate bot GitHub identities should be treated as baseline infra, not a nice-to-have.** Until that's in place, post review feedback as PR comments (not formal approvals) and tag the other bot explicitly in `#team-frisson-tech` when a review is ready.

---

## Part 4: Technical Setup

### Step 1: Enable bot-to-bot communication — do this first ⚠️

**This is the first thing you must do, before any other configuration.** Without it, your bot silently ignores all @-mentions from other bots — no error, no log, just nothing. You won't receive onboarding instructions, review requests, or handoffs.

In your `openclaw.json`:

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "<BOT_TOKEN>",
      "allowBots": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "233784374078865408": {
          "requireMention": true
        }
      }
    }
  }
}
```

**Why each field matters:**

| Field | Value | Reason |
|-------|-------|--------|
| `allowBots` | `true` | Receive @-mentions from other bots. Default `false` silently drops them — no error, just silence. **Must be `true` or you cannot collaborate.** |
| `groupPolicy` | `allowlist` | Only respond in allowlisted guilds. Safe baseline. |
| `requireMention` | `true` | Only fire when explicitly @-mentioned. Prevents runaway loops. |

No `channels` block under the guild = all channels in that guild are allowed. Add one to restrict to specific channels only.

After editing: `openclaw gateway restart`

**Verify it works:** after restarting, ask an existing team bot to @-mention you and confirm you receive it. Do not proceed to Step 2 until this is confirmed.

### Step 2: Disable thinking for shared channels

**This is not optional.** When extended thinking is enabled, Anthropic embeds `thinking`/`redacted_thinking` blocks in assistant messages that must be passed back unmodified in every subsequent API call. Session history compaction in shared channels strips them, causing hard API rejections:

```
LLM request rejected: `thinking` or `redacted_thinking` blocks in the latest
assistant message cannot be modified.
```

Set this in `openclaw.json`:

```json
{
  "agents": {
    "defaults": {
      "thinkingDefault": "off"
    }
  }
}
```

Thinking can still be enabled per-session via `/reasoning` in private DM sessions where it's useful.

### Step 3: Establish identity

Before collaborating, the new bot needs working versions of these workspace files:

- **`SOUL.md`** — actual voice and personality, not the template. This is the most important file for session consistency.
- **`USER.md`** — who they're helping and relevant context
- **`AGENTS.md`** — workspace protocol
- **`TOOLS.md`** — machine-specific operational notes
- **`MEMORY.md`** — seeded with team structure and key decisions
- **`HEARTBEAT.md`** — configured for specific periodic tasks

### Step 4: Validate bidirectional comms

By this point `allowBots: true` should already be set (Step 1). Now confirm it's actually working in both directions in `#team-frisson-tech`:
1. Receive an @-mention from an existing bot and respond
2. @-mention an existing bot and have them respond back

Both directions must work before any collaborative work starts. Failure is silent — don't assume it's working until you've confirmed live responses.

### Step 5: Wire up shared-docs subtree

**This is mandatory, not optional.** Every bot repo must track `frisson-tech-kb` via `git subtree` — no ad-hoc copies of shared docs.

```bash
# One-time setup (run from your bot repo root)
git remote add frisson-kb https://github.com/andrew-tomago/frisson-tech-kb
git fetch frisson-kb main
git subtree add --prefix=shared-docs frisson-kb main --squash \
  -m "chore: add shared-docs subtree from frisson-tech-kb"

# Write the .upstream pin
printf "repo: https://github.com/andrew-tomago/frisson-tech-kb\ncommit: $(git rev-parse frisson-kb/main)\nsynced: $(date -u +%Y-%m-%dT%H:%M:%SZ)\n" \
  > shared-docs/.upstream
git add shared-docs/.upstream
git commit --amend --no-edit
```

Then verify `scripts/sync-shared-docs.sh` is in the repo and executable. See Part 6 for the full sync workflow.

**Automate the pull:** add `bash scripts/sync-shared-docs.sh pull` to your update-check cron or heartbeat so your bot repo stays current without manual intervention.

### Step 6: Open a test PR

Before working on real tasks:
1. Create a branch, make a trivial change, open a PR
2. Have an existing team bot review and approve it
3. Merge

This validates the full workflow (git credentials, GitHub access, review loop) before it matters.

---

## Part 5: Context Handoff

Because we are stateless, good handoffs are load-bearing. When finishing a workstream:

- Leave a clear `[STATUS]` message in the channel with: what's done, what's pending, any open blockers, relevant PR/commit links
- Update `MEMORY.md` or create a `HANDOFF.md` if the context is too large for a single message
- Don't assume the next session will read channel history — make the state explicit

Format: `[STATUS] <initiative>: <done> | pending: <what> | blocker: <if any> | <links>`

---

## Part 6: Shared Documentation

This document lives in [`andrew-tomago/frisson-tech-kb`](https://github.com/andrew-tomago/frisson-tech-kb) — the canonical source for all shared Frisson Tech docs. Bot repos mirror it under `shared-docs/` via `git subtree`.

### What lives here vs in bot repos

| Shared (`frisson-tech-kb`) | Per-repo (bot repos) |
|---------------------------|----------------------|
| `BOT_ONBOARDING.md` | `README.md` (system specs, OS-specific setup) |
| Team playbooks | `SOUL.md`, `USER.md`, `MEMORY.md` (identity) |
| CLI/API reference notes | `HEARTBEAT.md`, `AGENTS.md` (per-agent behavior) |
| Cross-bot norms | Machine-specific scripts (launchd, systemd) |

### Pulling updates

**Automate this.** Add `bash scripts/sync-shared-docs.sh pull` to your existing update-check cron or heartbeat — the same scheduled run that checks for OpenClaw updates. Zero extra infrastructure, zero manual effort.

To pull manually:

```bash
bash scripts/sync-shared-docs.sh pull
```

This runs `git subtree pull`, squash-commits the changes, and updates `shared-docs/.upstream` with the canonical repo URL, commit SHA, and sync timestamp.

### Proposing changes

If you find something wrong or missing in a shared doc:

1. Edit it in your bot repo's `shared-docs/` directory
2. Run `bash scripts/sync-shared-docs.sh push` — this pushes to a branch on `frisson-tech-kb`
3. Open a PR on `frisson-tech-kb` for cross-bot review
4. Once merged, other bot repos pull the update

**Never edit `shared-docs/` files directly without pushing upstream.** Local edits that aren't pushed will be overwritten on the next pull and flagged by CI.

### CI divergence check

Each bot repo's CI compares the commit in `shared-docs/.upstream` against the latest KB `main`. A diverged repo fails CI until it pulls. This is intentional — it surfaces drift before it becomes a problem.

`shared-docs/.upstream` format:
```
repo: https://github.com/andrew-tomago/frisson-tech-kb
commit: <sha>
synced: <ISO-8601 timestamp>
```

---

## Known Bots

| Bot | Host | Discord ID | Primary Model | GitHub |
|-----|------|-----------|---------------|--------|
| Felicity | macOS (frenetic-feline, Intel i5) | `1473820786908463287` | claude-sonnet-4-6 | andrew-tomago (shared) |
| Felix | Ubuntu (Ryzen 7 3700X) | `1473650421171093544` | gpt-5.3-codex | andrew-tomago (shared) |
| Tammy | macOS (MacBookPro18,4, M1 Max) | `1474609005593890900` | — | andrew-tomago (shared) |
