# M001: Org-Wide PR Review Workflow — Context

**Gathered:** 2026-03-18
**Status:** Ready for planning

## Project Description

Centralized automated PR review workflow using `qodo-ai/pr-agent` with Claude Opus 4.6 (extended thinking enabled), stored in the `chillwhales/.github` org repo and enforced across all org repos via GitHub org rulesets.

## Why This Milestone

Every repo in the org should get consistent, high-quality AI code review without maintaining per-repo workflow files. This is the first org-wide workflow — it establishes the pattern for future centralized automation.

## User-Visible Outcome

### When this milestone is complete, the user can:

- Open a PR in any chillwhales org repo and get an automatic AI review from Claude Opus 4.6 with extended thinking
- Comment `/review`, `/improve`, or `/describe` on any PR to trigger on-demand review actions
- Manage the review workflow from a single file in the `.github` repo

### Entry point / environment

- Entry point: Push to non-default branch with open PR in any org repo
- Environment: GitHub Actions runner (ubuntu-latest)
- Live dependencies involved: Anthropic API via `secrets.ANTHROPIC_KEY`

## Completion Class

- Contract complete means: workflow file exists with correct triggers, model config, permissions, and extended thinking enabled
- Integration complete means: org ruleset documentation explains exact steps to wire this org-wide
- Operational complete means: none (operational proof requires org admin to configure ruleset + real PR push)

## Final Integrated Acceptance

To call this milestone complete, we must prove:

- The workflow file is syntactically valid and follows existing repo conventions
- Model config uses `anthropic/claude-opus-4-6` primary with `anthropic/claude-sonnet-4-6` fallback
- Extended thinking is enabled
- Draft PRs are excluded
- Both `pull_request` and `issue_comment` triggers are present
- Ruleset setup doc is clear and complete

## Risks and Unknowns

- pr-agent model string for Opus 4.6 may not be `anthropic/claude-opus-4-6` — pr-agent uses LiteLLM model strings, recent releases added Sonnet 4.6 mappings but Opus 4.6 support needs verification at runtime
- Extended thinking config (`enable_claude_extended_thinking`) may increase token costs significantly — user explicitly wants this, so it's accepted cost

## Existing Codebase / Prior Art

- `.github/workflows/ci-build-lint-test.yml` — existing reusable workflow, reference for naming/style conventions
- `.github/workflows/ci-quality.yml` — existing reusable workflow
- `.github/workflows/ci-publish-validation.yml` — existing reusable workflow
- `.github/actions/setup-pnpm/action.yml` — composite action pattern

> See `.gsd/DECISIONS.md` for all architectural and pattern decisions — it is an append-only register; read it during planning, append to it during execution.

## Relevant Requirements

- R001 — Org-wide review without per-repo files
- R002 — Claude Opus 4.6 with extended thinking
- R003 — Sonnet 4.6 fallback
- R004 — Triggers on non-default branch PRs
- R005 — Auto-review/describe/improve
- R006 — Slash commands via comments
- R007 — Draft PR exclusion
- R008 — Ruleset setup documentation

## Scope

### In Scope

- `ci-pr-review.yml` workflow file with pr-agent + Claude Opus 4.6 + extended thinking
- Ruleset setup documentation (`RULESET.md`)
- Org secret setup instructions for `ANTHROPIC_KEY`

### Out of Scope / Non-Goals

- Per-repo customization or overrides
- Non-Anthropic model backends
- Auto-merge or status-check gating
- Actually configuring the org ruleset (requires org admin UI access)

## Technical Constraints

- Workflow must use `pull_request` trigger (not `workflow_call`) for ruleset compatibility
- `issue_comment` trigger needed for slash commands
- pr-agent uses LiteLLM model strings prefixed with `anthropic/`
- Org-level `ANTHROPIC_KEY` secret must be accessible to the workflow

## Integration Points

- Anthropic API — model inference for review, describe, improve
- GitHub API — PR comments, reviews, descriptions via `GITHUB_TOKEN`
- GitHub Org Rulesets — enforces workflow execution across repos

## Open Questions

- Exact LiteLLM model string for Opus 4.6 (`anthropic/claude-opus-4-6` is the expected format based on API model ID `claude-opus-4-6`) — will be verified during implementation
