---
id: T02
parent: S01
milestone: M001
provides:
  - RULESET.md with step-by-step org secret and ruleset configuration guide for org admins
key_files:
  - RULESET.md
key_decisions:
  - Included a troubleshooting table covering the most common failure modes (auth errors, infinite loops, slash commands not firing)
  - Referenced specific GitHub navigation paths for org Settings to reduce ambiguity
patterns_established:
  - Org-level workflow distribution pattern: centralised workflow in .github repo + org ruleset = zero per-repo config
observability_surfaces:
  - Troubleshooting table in RULESET.md maps symptoms to causes and fixes
  - Workflow reference table documents all model/feature settings in one place for admin inspection
duration: 5m
verification_result: passed
completed_at: 2026-03-18
blocker_discovered: false
---

# T02: Write RULESET.md org ruleset setup guide

**Created RULESET.md with step-by-step org secret setup, ruleset configuration, testing steps, and a troubleshooting table**

## What Happened

Created `RULESET.md` at the repo root covering all required sections:
- **Overview:** explains the centralised workflow + org ruleset pattern
- **Prerequisites:** admin access and Anthropic API key
- **Step 1:** org secret creation (`ANTHROPIC_KEY`) with repo access scope guidance
- **Step 2:** org ruleset creation — navigation path, targeting pattern (`~DEFAULT_BRANCH`), required workflow configuration pointing at `chillwhales/.github` → `ci-pr-review.yml`
- **Step 3:** testing instructions (PR comments + slash command verification)
- **Troubleshooting:** table covering auth errors, missing secrets, workflow not triggering, bot loops, slash command failures
- **Workflow reference table:** summarises all model and feature flag settings

## Verification

All checks passed:
- `test -f RULESET.md` ✅
- `grep -q 'ANTHROPIC_KEY' RULESET.md` ✅
- `grep -q 'ci-pr-review.yml' RULESET.md` ✅
- `grep -q 'Ruleset' RULESET.md` ✅
- `grep -q 'Troubleshoot' RULESET.md` ✅

Full slice-level verification — all 8 checks pass:
- YAML valid, primary model, fallback model, ANTHROPIC.KEY dot notation, draft exclusion, issue_comment trigger, bot exclusion, both job IDs ✅
- `test -f RULESET.md` ✅

## Diagnostics

- `test -f RULESET.md && echo ok` — confirms file exists
- The troubleshooting table in RULESET.md is itself the diagnostic surface for runtime failures

## Deviations

None.

## Known Issues

None.

## Files Created/Modified

- `RULESET.md` — New org admin setup guide at repo root
