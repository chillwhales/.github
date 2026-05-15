---
estimated_steps: 5
estimated_files: 1
---

# T01: Create pr-agent workflow with Claude Opus 4.6 and extended thinking

**Slice:** S01 — PR Review Workflow + Org Ruleset Doc
**Milestone:** M001

## Description

Create the GitHub Actions workflow file `ci-pr-review.yml` that runs qodo-ai/pr-agent with Claude Opus 4.6 as the primary model, extended thinking enabled, and Sonnet 4.6 as fallback. The workflow has two jobs: one for automatic PR review on pull_request events, and one for slash commands via issue_comment. This single file addresses requirements R001–R007.

**Skills:** `github-workflows` — load this skill for GitHub Actions syntax guidance.

## Steps

1. Create `.github/workflows/ci-pr-review.yml` with the following structure:

   **Top-level:**
   - `name: CI — PR Review` (follows existing `CI —` convention)
   - `on:` with both `pull_request` and `issue_comment` triggers

   **Job 1: `pr_agent_review`** (automatic review on PR events)
   - Trigger: `pull_request` types `[opened, reopened, ready_for_review, synchronize]`
   - Condition: `if: github.event_name == 'pull_request' && github.event.pull_request.draft == false && github.event.sender.type != 'Bot'`
   - `runs-on: ubuntu-latest`
   - Permissions: `issues: write`, `pull-requests: write`, `contents: read`
   - Steps:
     - `actions/checkout@v4`
     - `qodo-ai/pr-agent@main` with `github_token: ${{ secrets.GITHUB_TOKEN }}`
   - Environment variables (env block on the pr-agent step):
     - `ANTHROPIC.KEY: ${{ secrets.ANTHROPIC_KEY }}` — **CRITICAL: dot notation, not underscore**
     - `config.model: "anthropic/claude-opus-4-6"`
     - `config.model_turbo: "anthropic/claude-sonnet-4-6"`
     - `config.fallback_models: '["anthropic/claude-sonnet-4-6"]'`
     - `config.enable_claude_extended_thinking: "true"`
     - `config.extended_thinking_budget_tokens: "10240"`
     - `config.max_model_tokens: "16384"`
     - `config.auto_review: "true"`
     - `config.auto_describe: "true"`
     - `config.auto_improve: "true"`

   **Job 2: `pr_agent_commands`** (slash commands via comments)
   - Trigger: `issue_comment` (no type filter — pr-agent handles parsing)
   - Condition: `if: github.event_name == 'issue_comment' && github.event.issue.pull_request && github.event.sender.type != 'Bot'`
   - Same runner, permissions, and pr-agent step config as Job 1
   - Same env vars (model config and API key)

2. Validate YAML syntax: `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/ci-pr-review.yml'))"`

3. Verify all requirement markers are present via grep:
   - Model string `anthropic/claude-opus-4-6` (R002)
   - Fallback `anthropic/claude-sonnet-4-6` (R003)
   - `pull_request` with `synchronize` (R004)
   - `auto_review`, `auto_describe`, `auto_improve` (R005)
   - `issue_comment` trigger (R006)
   - `draft == false` condition (R007)
   - `ANTHROPIC.KEY` with dot notation (R002 — API key)

## Must-Haves

- [ ] Workflow name follows `CI —` convention
- [ ] `pull_request` trigger includes `opened`, `reopened`, `ready_for_review`, `synchronize`
- [ ] Draft PRs excluded via `github.event.pull_request.draft == false`
- [ ] Bot senders excluded via `github.event.sender.type != 'Bot'`
- [ ] Primary model is `anthropic/claude-opus-4-6`
- [ ] Fallback model is `anthropic/claude-sonnet-4-6`
- [ ] Extended thinking enabled with 10240 token budget
- [ ] `ANTHROPIC.KEY` uses dot notation (NOT `ANTHROPIC_KEY`)
- [ ] Auto-review, auto-describe, auto-improve all set to `"true"`
- [ ] `issue_comment` job handles slash commands
- [ ] Permissions include `pull-requests: write`, `issues: write`, `contents: read`
- [ ] Uses `actions/checkout@v4` and `qodo-ai/pr-agent@main`

## Verification

- `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/ci-pr-review.yml'))"` exits 0
- `grep -c 'anthropic/claude-opus-4-6' .github/workflows/ci-pr-review.yml` returns ≥1
- `grep -c 'anthropic/claude-sonnet-4-6' .github/workflows/ci-pr-review.yml` returns ≥1
- `grep -c 'ANTHROPIC.KEY' .github/workflows/ci-pr-review.yml` returns ≥1
- `grep -c 'issue_comment' .github/workflows/ci-pr-review.yml` returns ≥1
- `grep -c 'draft.*false' .github/workflows/ci-pr-review.yml` returns ≥1

## Inputs

- `.github/workflows/ci-build-lint-test.yml` — reference for naming convention (`CI — ` prefix, `runs-on: ubuntu-latest`, `actions/checkout@v4`)

## Expected Output

- `.github/workflows/ci-pr-review.yml` — complete, valid workflow file with two jobs covering automatic PR review and slash commands

## Observability Impact

- **New signal:** Workflow runs appear in the Actions tab for every non-draft, non-bot PR event and every issue comment on PRs.
- **Inspection:** `gh run list --workflow=ci-pr-review.yml` lists runs. PR comments from pr-agent (review/describe/improve) are the primary observable output.
- **Failure visibility:** Missing ANTHROPIC_KEY → step failure with auth error in run logs. Invalid model → Anthropic API error in step output. Skipped jobs (draft/bot) show as grey "Skipped" in Actions UI.
- **Future agent inspection:** Verify workflow correctness with `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/ci-pr-review.yml'))"` and grep checks. Verify runtime with `gh run list --workflow=ci-pr-review.yml`.
