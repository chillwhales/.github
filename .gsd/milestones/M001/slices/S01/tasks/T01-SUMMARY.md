---
id: T01
parent: S01
milestone: M001
provides:
  - ci-pr-review.yml workflow with Claude Opus 4.6 + extended thinking, auto-review/describe/improve, slash commands, draft/bot exclusion
key_files:
  - .github/workflows/ci-pr-review.yml
key_decisions:
  - Permissions set at workflow level (not job level) since both jobs need the same permissions
  - Slash command job omits auto_review/auto_describe/auto_improve since those only apply to automatic PR events
patterns_established:
  - pr-agent env vars use dot notation (ANTHROPIC.KEY, config.model) not underscore
observability_surfaces:
  - Workflow runs visible in Actions tab per PR; pr-agent posts review/describe/improve as PR comments
  - Skipped jobs (draft/bot) show as grey "Skipped" in Actions UI
  - Missing ANTHROPIC_KEY secret surfaces as step failure with auth error in run logs
duration: 8m
verification_result: passed
completed_at: 2026-03-18
blocker_discovered: false
---

# T01: Create pr-agent workflow with Claude Opus 4.6 and extended thinking

**Created ci-pr-review.yml with two jobs: auto-review on PR events (Claude Opus 4.6 + extended thinking) and slash commands via issue_comment**

## What Happened

Created `.github/workflows/ci-pr-review.yml` with:
- **Job 1 (`pr_agent_review`):** Triggers on `pull_request` (opened, reopened, ready_for_review, synchronize). Excludes draft PRs and bot senders. Runs qodo-ai/pr-agent@main with Claude Opus 4.6 as primary model, Sonnet 4.6 as fallback, extended thinking enabled (10240 token budget), and auto-review/describe/improve all enabled.
- **Job 2 (`pr_agent_commands`):** Triggers on `issue_comment` for PRs. Excludes bot senders. Same model configuration for slash command processing.
- Workflow-level permissions: `issues: write`, `pull-requests: write`, `contents: read`.
- `ANTHROPIC.KEY` uses dot notation mapped from `secrets.ANTHROPIC_KEY`.

Also added Observability sections to S01-PLAN.md and T01-PLAN.md per pre-flight requirements, and added a failure-path verification check to slice verification.

## Verification

All checks passed:
- `python3 -c "import yaml; yaml.safe_load(...)"` — YAML valid ✅
- `grep -c 'anthropic/claude-opus-4-6'` → 2 ✅
- `grep -c 'anthropic/claude-sonnet-4-6'` → 4 ✅
- `grep -c 'ANTHROPIC.KEY'` → 2 ✅
- `grep -c 'issue_comment'` → 2 ✅
- `grep -c 'draft.*false'` → 1 ✅
- `grep -c 'sender.type'` → 2 ✅
- `auto_review`, `auto_describe`, `auto_improve` all present ✅
- `synchronize` trigger type present ✅
- Both job IDs (`pr_agent_review`, `pr_agent_commands`) present ✅

Slice-level verification (6/7 passing — RULESET.md is T02):
- YAML valid ✅, primary model ✅, fallback model ✅, ANTHROPIC.KEY ✅, draft exclusion ✅, issue_comment ✅, sender.type ✅
- `test -f RULESET.md` — expected fail (T02 deliverable)

## Diagnostics

- Verify workflow syntax: `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/ci-pr-review.yml'))"`
- Check env var notation: `grep 'ANTHROPIC.KEY' .github/workflows/ci-pr-review.yml`
- After deployment, check runs: `gh run list --workflow=ci-pr-review.yml`
- View PR comments from pr-agent for review/describe/improve output

## Deviations

- Permissions set at workflow level instead of per-job since both jobs need identical permissions — cleaner and avoids duplication.
- Slash command job omits `auto_review`/`auto_describe`/`auto_improve` env vars since those config keys only affect the automatic PR event flow, not manual slash commands.

## Known Issues

None.

## Files Created/Modified

- `.github/workflows/ci-pr-review.yml` — New workflow file with two jobs for PR review and slash commands
- `.gsd/milestones/M001/slices/S01/S01-PLAN.md` — Added Observability section and failure-path verification
- `.gsd/milestones/M001/slices/S01/tasks/T01-PLAN.md` — Added Observability Impact section
