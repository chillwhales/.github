# Decisions Register

<!-- Append-only. Never edit or remove existing rows.
     To reverse a decision, add a new row that supersedes it.
     Read this file at the start of any planning or research phase. -->

| # | When | Scope | Decision | Choice | Rationale | Revisable? |
|---|------|-------|----------|--------|-----------|------------|
| D001 | M001 | arch | Workflow trigger strategy | `pull_request` + `issue_comment` (not `workflow_call`) | pr-agent needs PR context; org rulesets require `pull_request` trigger directly in the workflow file | No |
| D002 | M001 | library | AI model for PR review | `anthropic/claude-opus-4-6` primary, `anthropic/claude-sonnet-4-6` fallback | User wants best reasoning model with extended thinking; Opus 4.6 is current top model | Yes — if newer model released |
| D003 | M001 | arch | Org-wide distribution mechanism | GitHub org rulesets (not per-repo `workflow_call`) | Single source of truth without per-repo workflow files; rulesets enforce automatically | No |
| D004 | M001 | convention | Workflow naming | `ci-pr-review.yml` | Consistent with existing `ci-*.yml` naming convention in the repo | No |
| D005 | M001 | arch | Extended thinking | Enabled via `config.enable_claude_extended_thinking: "true"` | User explicitly wants max reasoning depth for reviews; Opus 4.6 adaptive thinking defaults to high effort | Yes — if cost becomes prohibitive |
