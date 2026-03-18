# M001: Org-Wide PR Review Workflow

**Vision:** Centralized automated PR review using qodo-ai/pr-agent + Claude Opus 4.6 with extended thinking, enforced across all chillwhales org repos via GitHub rulesets — single source of truth, no per-repo workflow files.

## Success Criteria

- Any PR opened in a chillwhales org repo triggers automatic AI review (after ruleset is configured)
- Review uses Claude Opus 4.6 with extended thinking enabled
- Developers can invoke `/review`, `/improve`, `/describe` via PR comments
- Draft PRs are excluded from automatic review
- Workflow is managed from a single file in the `.github` repo

## Key Risks / Unknowns

- pr-agent LiteLLM model string for Opus 4.6 — expected `anthropic/claude-opus-4-6` but needs runtime verification

## Proof Strategy

- Model string validity → retire in S01 by verifying workflow YAML against pr-agent docs and LiteLLM model naming conventions

## Verification Classes

- Contract verification: workflow YAML syntax validation, correct triggers/permissions/env vars present
- Integration verification: none (requires org admin to configure ruleset + real PR)
- Operational verification: none
- UAT / human verification: org admin configures ruleset, pushes a test PR, confirms review appears

## Milestone Definition of Done

This milestone is complete only when all are true:

- `ci-pr-review.yml` exists with correct triggers, permissions, model config, and extended thinking
- Workflow follows naming/style conventions of existing workflows in the repo
- `RULESET.md` documents step-by-step org ruleset configuration
- All 8 active requirements (R001–R008) are addressed
- Workflow YAML passes syntax validation

## Requirement Coverage

- Covers: R001, R002, R003, R004, R005, R006, R007, R008
- Partially covers: none
- Leaves for later: none
- Orphan risks: none

## Slices

- [ ] **S01: PR Review Workflow + Org Ruleset Doc** `risk:low` `depends:[]`
  > After this: `ci-pr-review.yml` exists with Claude Opus 4.6 + extended thinking, auto-review/describe/improve, slash commands, draft exclusion; `RULESET.md` documents org ruleset setup. Proven by workflow syntax validation and file inspection — live PR review requires org admin to configure the ruleset.

## Boundary Map

### S01 (leaf slice)

Produces:
- `.github/workflows/ci-pr-review.yml` — standalone workflow with `pull_request` + `issue_comment` triggers, qodo-ai/pr-agent action, Claude Opus 4.6 config, extended thinking, Sonnet 4.6 fallback
- `RULESET.md` — step-by-step guide for org admin to configure GitHub org ruleset pointing at this workflow + org secret setup

Consumes:
- nothing (first and only slice)
