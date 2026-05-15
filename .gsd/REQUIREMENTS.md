# Requirements

This file is the explicit capability and coverage contract for the project.

## Active

### R001 — Org-wide automated PR review without per-repo workflow files
- Class: core-capability
- Status: active
- Description: A single workflow in the `.github` org repo runs pr-agent reviews across all org repos without needing workflow files in each repo.
- Why it matters: Eliminates duplication, ensures consistent review quality, single source of truth.
- Source: user
- Primary owning slice: M001/S01
- Supporting slices: none
- Validation: mapped
- Notes: Enforced via GitHub org rulesets pointing at the centralized workflow.

### R002 — Reviews powered by Claude Opus 4.6 with extended thinking
- Class: core-capability
- Status: active
- Description: PR reviews use `anthropic/claude-opus-4-6` as primary model with extended thinking enabled via `secrets.ANTHROPIC_KEY`.
- Why it matters: Best available reasoning for thorough code review — the user explicitly wants the strongest model with deep thinking.
- Source: user
- Primary owning slice: M001/S01
- Supporting slices: none
- Validation: mapped
- Notes: Opus 4.6 defaults to high effort with adaptive thinking, effectively max reasoning depth.

### R003 — Fallback to Claude Sonnet 4.6 on primary failure
- Class: continuity
- Status: active
- Description: If Opus 4.6 fails (rate limit, timeout), falls back to `anthropic/claude-sonnet-4-6`.
- Why it matters: Reviews shouldn't silently fail because the primary model is temporarily unavailable.
- Source: inferred
- Primary owning slice: M001/S01
- Supporting slices: none
- Validation: mapped
- Notes: none

### R004 — Triggers on PR open/sync/reopen on any non-default branch
- Class: primary-user-loop
- Status: active
- Description: Workflow fires on `pull_request` events (opened, reopened, synchronize, ready_for_review) targeting any branch.
- Why it matters: Every push to a feature branch with an open PR gets reviewed automatically.
- Source: user
- Primary owning slice: M001/S01
- Supporting slices: none
- Validation: mapped
- Notes: Uses `pull_request` trigger since pr-agent needs PR context. `synchronize` fires on every push to the PR branch.

### R005 — Auto-review + auto-describe + auto-improve on trigger
- Class: primary-user-loop
- Status: active
- Description: On each trigger event, pr-agent automatically runs review, describe, and improve commands.
- Why it matters: No manual invocation needed — reviews happen automatically on every meaningful PR event.
- Source: user
- Primary owning slice: M001/S01
- Supporting slices: none
- Validation: mapped
- Notes: none

### R006 — Comment-triggered slash commands
- Class: core-capability
- Status: active
- Description: Developers can invoke `/review`, `/improve`, `/describe`, etc. by commenting on the PR.
- Why it matters: On-demand re-review or specific tool invocation without re-pushing.
- Source: inferred
- Primary owning slice: M001/S01
- Supporting slices: none
- Validation: mapped
- Notes: Requires `issue_comment` trigger in the workflow.

### R007 — Draft PRs excluded from automatic review
- Class: quality-attribute
- Status: active
- Description: Draft PRs do not trigger automatic review. Only non-draft PRs are reviewed.
- Why it matters: Avoids noisy reviews on work-in-progress PRs.
- Source: inferred
- Primary owning slice: M001/S01
- Supporting slices: none
- Validation: mapped
- Notes: Uses `github.event.pull_request.draft == false` condition.

### R008 — Ruleset setup instructions documented for org admin
- Class: operability
- Status: active
- Description: A clear doc explains how to configure the GitHub org ruleset to point at this workflow so it runs across all repos.
- Why it matters: The workflow file alone isn't enough — someone needs to wire the ruleset in the org settings UI.
- Source: inferred
- Primary owning slice: M001/S01
- Supporting slices: none
- Validation: mapped
- Notes: Includes ANTHROPIC_KEY org secret setup.

## Validated

(none yet)

## Deferred

(none)

## Out of Scope

### R030 — Per-repo workflow customization or overrides
- Class: constraint
- Status: out-of-scope
- Description: Individual repos cannot override or customize the review workflow behavior.
- Why it matters: Prevents scope creep — single source of truth means uniform behavior.
- Source: user
- Primary owning slice: none
- Supporting slices: none
- Validation: n/a
- Notes: If needed later, could use per-repo `.pr_agent.toml` config files.

### R031 — Non-Anthropic model backends
- Class: anti-feature
- Status: out-of-scope
- Description: No support for OpenAI, Gemini, or other model providers.
- Why it matters: Keeps config simple and focused on the user's chosen provider.
- Source: user
- Primary owning slice: none
- Supporting slices: none
- Validation: n/a
- Notes: none

### R032 — Auto-merge or status-check gating on review outcome
- Class: anti-feature
- Status: out-of-scope
- Description: PR review results do not block merge or trigger auto-merge.
- Why it matters: Review is advisory, not a gate.
- Source: inferred
- Primary owning slice: none
- Supporting slices: none
- Validation: n/a
- Notes: none

## Traceability

| ID | Class | Status | Primary owner | Supporting | Proof |
|---|---|---|---|---|---|
| R001 | core-capability | active | M001/S01 | none | mapped |
| R002 | core-capability | active | M001/S01 | none | mapped |
| R003 | continuity | active | M001/S01 | none | mapped |
| R004 | primary-user-loop | active | M001/S01 | none | mapped |
| R005 | primary-user-loop | active | M001/S01 | none | mapped |
| R006 | core-capability | active | M001/S01 | none | mapped |
| R007 | quality-attribute | active | M001/S01 | none | mapped |
| R008 | operability | active | M001/S01 | none | mapped |
| R030 | constraint | out-of-scope | none | none | n/a |
| R031 | anti-feature | out-of-scope | none | none | n/a |
| R032 | anti-feature | out-of-scope | none | none | n/a |

## Coverage Summary

- Active requirements: 8
- Mapped to slices: 8
- Validated: 0
- Unmapped active requirements: 0
