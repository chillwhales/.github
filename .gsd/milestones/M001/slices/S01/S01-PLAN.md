# S01: PR Review Workflow + Org Ruleset Doc

**Goal:** A complete pr-agent workflow exists in the `.github` org repo with Claude Opus 4.6 + extended thinking, auto-review/describe/improve, slash commands, and draft exclusion — plus documentation for org ruleset setup.
**Demo:** `ci-pr-review.yml` passes YAML syntax validation; file inspection confirms all 8 requirements are addressed; `RULESET.md` provides clear org admin instructions.

## Must-Haves

- `ci-pr-review.yml` with `pull_request` trigger (opened, reopened, synchronize, ready_for_review) and `issue_comment` trigger
- Claude Opus 4.6 (`anthropic/claude-opus-4-6`) as primary model with extended thinking enabled
- Sonnet 4.6 (`anthropic/claude-sonnet-4-6`) as fallback model
- Auto-review, auto-describe, auto-improve all enabled
- Draft PR exclusion via `github.event.pull_request.draft == false`
- Bot sender exclusion to prevent infinite loops
- `ANTHROPIC.KEY` env var (dot notation) mapped from `secrets.ANTHROPIC_KEY`
- `RULESET.md` with org ruleset configuration steps and secret setup

## Verification

- `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/ci-pr-review.yml'))"` — valid YAML
- `grep -q 'anthropic/claude-opus-4-6' .github/workflows/ci-pr-review.yml` — primary model present
- `grep -q 'anthropic/claude-sonnet-4-6' .github/workflows/ci-pr-review.yml` — fallback model present
- `grep -q 'ANTHROPIC.KEY' .github/workflows/ci-pr-review.yml` — correct env var name (dot notation)
- `grep -q 'draft' .github/workflows/ci-pr-review.yml` — draft exclusion present
- `grep -q 'issue_comment' .github/workflows/ci-pr-review.yml` — slash command trigger present
- `test -f RULESET.md` — documentation exists
- `grep -q 'sender.type' .github/workflows/ci-pr-review.yml` — bot exclusion present (failure-path: prevents infinite review loops)
- Workflow YAML contains both job IDs (`pr_agent_review`, `pr_agent_commands`) — structural completeness check

## Tasks

- [x] **T01: Create pr-agent workflow with Claude Opus 4.6 and extended thinking** `est:20m`
  - Why: Core deliverable — the workflow file that powers all PR reviews across the org (R001–R007)
  - Files: `.github/workflows/ci-pr-review.yml`
  - Do: Create workflow with two jobs: `pr_agent_review` (pull_request trigger with draft/bot exclusion, qodo-ai/pr-agent@main, Claude Opus 4.6 config with extended thinking and fallback) and `pr_agent_commands` (issue_comment trigger for slash commands). Use `ANTHROPIC.KEY` env var (dot notation). Follow existing `CI —` naming convention. Set extended thinking budget to 10240 tokens, max output 16384.
  - Verify: `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/ci-pr-review.yml'))"` passes; grep confirms model strings, env var name, triggers, and draft condition are all present
  - Done when: Valid YAML file with all R001–R007 requirements addressed

- [x] **T02: Write RULESET.md org ruleset setup guide** `est:15m`
  - Why: The workflow alone does nothing without org ruleset configuration — this doc closes R008
  - Files: `RULESET.md`
  - Do: Write step-by-step guide covering: (1) creating `ANTHROPIC_KEY` org secret, (2) navigating to org Settings → Rulesets, (3) creating a new ruleset targeting all repos, (4) adding the `ci-pr-review.yml` workflow as a required workflow, (5) testing with a real PR. Include prerequisites and troubleshooting tips.
  - Verify: `test -f RULESET.md` and file contains sections for secret setup and ruleset configuration
  - Done when: `RULESET.md` exists with clear, actionable instructions an org admin can follow

## Observability / Diagnostics

- **Workflow run visibility:** PR review results appear as PR comments (review, describe, improve). Failed runs surface in the Actions tab with step-level logs.
- **Inspection surfaces:** `gh run list --workflow=ci-pr-review.yml` shows recent runs; `gh run view <id> --log-failed` shows failure details.
- **Failure modes:** Missing `ANTHROPIC_KEY` secret → pr-agent step fails with auth error in logs. Invalid model string → Anthropic API 400 error visible in step output. Draft PR or bot sender → job skipped (visible in Actions UI as grey "skipped" status).
- **Redaction:** `ANTHROPIC_KEY` is a GitHub secret — never exposed in logs. The workflow references it via `secrets.ANTHROPIC_KEY` which GitHub redacts automatically.
- **Diagnostic verification:** `grep -q 'ANTHROPIC.KEY' .github/workflows/ci-pr-review.yml` confirms the env var uses correct dot notation (a common misconfiguration).

## Files Likely Touched

- `.github/workflows/ci-pr-review.yml`
- `RULESET.md`
