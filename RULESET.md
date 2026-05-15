# PR Review Ruleset Setup Guide

This guide explains how to configure the org-wide PR review workflow for the chillwhales GitHub organisation. The `.github` repo contains a centralised workflow (`ci-pr-review.yml`) that runs [Claude Code Action](https://github.com/anthropics/claude-code-action) with Claude Opus 4.6 on every pull request. An org ruleset makes it a required check across all (or selected) repos without any per-repo configuration.

---

## Prerequisites

- Admin access to the chillwhales GitHub org
- A Claude Max subscription (the workflow uses your Claude Code OAuth token — no separate Anthropic API key needed)

---

## Step 1: Get your Claude Code OAuth token

The workflow authenticates using your Claude Code OAuth token from your Max subscription.

1. On a machine with Claude Code installed, find your credentials:
   - **Linux:** `~/.claude/.credentials.json`
   - **macOS:** Search "claude" in Keychain Access → Show Password
2. Copy the `oauth_token` value

> **Note:** This uses your existing Claude Max subscription budget — no separate API billing. Each PR review costs tokens from your subscription allowance.

---

## Step 2: Create the `CLAUDE_CODE_OAUTH_TOKEN` org secret

1. Go to **github.com/organizations/chillwhales** → **Settings** → **Secrets and variables** → **Actions**
2. Click **New organization secret**
3. Fill in:
   - **Name:** `CLAUDE_CODE_OAUTH_TOKEN`
   - **Value:** your OAuth token from Step 1
   - **Repository access:** _All repositories_ (or select specific repos)
4. Click **Add secret**

---

## Step 3: Configure the org ruleset

The ruleset makes `ci-pr-review.yml` a required workflow for pull requests across the org.

1. Go to **github.com/organizations/chillwhales** → **Settings** → **Rulesets**
2. Click **New ruleset** → **New branch ruleset**
3. Configure the ruleset:
   - **Name:** `PR Review` (or any descriptive name)
   - **Enforcement status:** `Active`
4. Under **Target branches**:
   - **Target repositories:** _All repositories_ (or pick specific repos)
   - **Target branches:** Add a targeting pattern — use `~DEFAULT_BRANCH` to match each repo's default branch, or enter `main` if all repos use `main`
5. Under **Branch protections**, enable **Require status checks to pass**
6. Within that section, enable **Require workflows to pass before merging** and click **Add workflow**:
   - **Repository:** `chillwhales/.github`
   - **Workflow file path:** `.github/workflows/ci-pr-review.yml`
   - **Ref (branch):** `main`
7. Click **Create** to save the ruleset

---

## Step 4: Test

1. Open a PR in any targeted repo
2. Verify a Claude review comment appears within a few minutes with:
   - **Must Fix** — critical issues
   - **Should Consider** — important improvements
   - **Minor** — nits and style suggestions
   - **Summary** — brief overview
3. In a PR comment, type `@claude review this again` — verify Claude responds to the mention
4. In GitHub → **Settings** → **Rulesets**, the ruleset status indicator should show green when the workflow passes

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Workflow fails with auth error | `CLAUDE_CODE_OAUTH_TOKEN` secret missing or expired | Re-check org secret; tokens may expire — regenerate from `~/.claude/.credentials.json` |
| Secret not available in a repo | Repository access scope on the secret | Edit the org secret and change access to "All repositories" or add the specific repo |
| Workflow not triggering on PRs | Ruleset not active or not targeting the repo | Check ruleset enforcement is "Active" and the repo is in scope |
| Claude runs but posts no comments | `GITHUB_TOKEN` permissions issue | The workflow sets `pull-requests: write` and `issues: write` — ensure org settings allow workflows to create PRs/issues |
| Review triggers on draft PRs | Unexpected — workflow has `draft == false` guard | Verify the `if:` condition in `ci-pr-review.yml` job `pr_agent_review` is present |
| `@claude` commands not working | Comment doesn't contain `@claude` or sender is a bot | Check the `pr_agent_commands` job condition |
| Token budget exceeded | Too many reviews consuming Max subscription tokens | Reduce review frequency or switch model to `claude-sonnet-4-6` for lower token usage |

---

## Workflow reference

The workflow is at `.github/workflows/ci-pr-review.yml` in this repo. Key configuration:

| Setting | Value |
|---|---|
| Action | `anthropics/claude-code-action@v1` |
| Model | `claude-opus-4-6` |
| Auth | Claude Code OAuth token (Max subscription) |
| Auto-review | On PR open/reopen/sync/ready-for-review |
| Slash commands | `@claude` in PR comments |
| Draft PR exclusion | Yes (`draft == false`) |
| Bot sender exclusion | Yes (`sender.type != 'Bot'`) |
| Concurrency | One review per PR at a time |

To change model or review behaviour, edit the workflow file directly. The `prompt` field controls what Claude reviews and how it formats output.
