# PR Review Ruleset Setup Guide

This guide explains how to configure the org-wide PR review workflow for the chillwhales GitHub organisation. The `.github` repo contains a centralised workflow (`ci-pr-review.yml`) that runs [qodo-ai/pr-agent](https://github.com/qodo-ai/pr-agent) with Claude Opus 4.6 on every pull request. An org ruleset makes it a required check across all (or selected) repos without any per-repo configuration.

---

## Prerequisites

- Admin access to the chillwhales GitHub org
- An Anthropic API key with access to Claude Opus 4.6 (and Claude Sonnet 4.6 for fallback)

---

## Step 1: Create the `ANTHROPIC_KEY` org secret

The workflow authenticates to the Anthropic API using an org-level secret so it's available in every repo without per-repo setup.

1. Go to **github.com/organizations/chillwhales** → **Settings** → **Secrets and variables** → **Actions**
2. Click **New organization secret**
3. Fill in:
   - **Name:** `ANTHROPIC_KEY`
   - **Value:** your Anthropic API key
   - **Repository access:** _All repositories_ (or select specific repos if you want to restrict which repos get PR review)
4. Click **Add secret**

> **Tip:** The API key needs access to `claude-opus-4-6` and `claude-sonnet-4-6`. Verify this in your [Anthropic Console](https://console.anthropic.com/) if you see auth errors.

---

## Step 2: Configure the org ruleset

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

> **Note:** The ruleset only enforces the workflow as a required check. The workflow itself also runs as a non-blocking check (for review comments) even without the ruleset — the ruleset just makes it a merge gate.

---

## Step 3: Test

1. Open a PR in any targeted repo
2. Verify three pr-agent comments appear within a few minutes:
   - A **description** comment (PR summary)
   - A **review** comment (code review with findings)
   - An **improvement** comment (suggested code improvements)
3. In a PR comment, type `/review` and submit — verify pr-agent re-runs its review as a slash command
4. In GitHub → **Settings** → **Rulesets**, the ruleset status indicator should show green when the workflow passes

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| "Authentication error" or 401 in workflow logs | `ANTHROPIC_KEY` secret missing or wrong | Re-check org secret name (must be `ANTHROPIC_KEY`, no typo) and value |
| Secret not available in a repo | Repository access scope on the secret | Edit the org secret and change access to "All repositories" or add the specific repo |
| Workflow not triggering on PRs | Ruleset not active or not targeting the repo | Check ruleset enforcement is "Active" and the repo is in scope |
| pr-agent runs but posts no comments | `GITHUB_TOKEN` permissions issue | The workflow sets `pull-requests: write` and `issues: write` — ensure org settings allow workflows to create PRs/issues |
| Review triggers on draft PRs | Unexpected — workflow has `draft == false` guard | Verify the `if:` condition in `ci-pr-review.yml` job `pr_agent_review` is present |
| Infinite comment loop (bot replies trigger review) | Bot exclusion not working | The workflow checks `github.event.sender.type != 'Bot'`; verify the condition is present in both jobs |
| Slash commands not working | `issue_comment` job not running | Check the `pr_agent_commands` job condition: it requires `github.event.issue.pull_request` (only comments on PRs, not issues) |

---

## Workflow reference

The workflow is at `.github/workflows/ci-pr-review.yml` in this repo. Key configuration:

| Setting | Value |
|---|---|
| Primary model | `anthropic/claude-opus-4-6` |
| Fallback model | `anthropic/claude-sonnet-4-6` |
| Extended thinking | Enabled (10240 token budget) |
| Auto-review | Enabled |
| Auto-describe | Enabled |
| Auto-improve | Enabled |
| Draft PR exclusion | Yes (`draft == false`) |
| Bot sender exclusion | Yes (`sender.type != 'Bot'`) |

To change model or feature flags, edit the `env:` block on the `qodo-ai/pr-agent@main` step in `ci-pr-review.yml`.
