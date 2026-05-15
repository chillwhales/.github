# ChillWhales Org Automation

## What This Is

The `chillwhales/.github` org repo — single source of truth for shared GitHub Actions workflows, composite actions, and org-wide automation. Existing reusable workflows handle CI build/lint/test, quality checks, and publish validation. The goal is to expand this with org-wide workflows enforced via GitHub rulesets, starting with automated PR review.

## Core Value

One place to define workflows that run across every repo in the org — no per-repo duplication, no drift.

## Current State

- Three reusable `workflow_call` workflows: `ci-build-lint-test.yml`, `ci-quality.yml`, `ci-publish-validation.yml`
- Two composite actions: `setup-pnpm`, `build-and-upload`
- Used by `lsp-indexer` repo via `workflow_call`
- No org-wide ruleset-enforced workflows yet

## Architecture / Key Patterns

- Workflows use `workflow_call` trigger with configurable inputs for reuse
- Composite actions extract common step sequences
- pnpm-based Node.js ecosystem

## Capability Contract

See `.gsd/REQUIREMENTS.md` for the explicit capability contract, requirement status, and coverage mapping.

## Milestone Sequence

- [ ] M001: Org-Wide PR Review Workflow — Automated code review via qodo-ai/pr-agent + Claude Opus 4.6 with extended thinking, enforced org-wide via GitHub rulesets
