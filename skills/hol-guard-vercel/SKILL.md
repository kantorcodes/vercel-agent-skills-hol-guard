---
name: hol-guard-vercel
description: Use when an AI coding agent is about to deploy, promote, roll back, remove, or otherwise mutate Vercel resources and you want HOL Guard to inspect the exact Vercel CLI command before it executes.
---

# HOL Guard for Vercel CLI safety

Use HOL Guard as a pre-execution command-safety check for Vercel CLI operations.

HOL Guard release/3.0 includes `command.platform.vercel` coverage for production-impacting Vercel operations such as removing deployments or projects and deploying, promoting, or rolling back production. It preserves safe inspection/help counterparts such as help, project inspection, and promotion status.

## Install

The Vercel command coverage currently lives in the HOL Guard 3.x prerelease channel:

```bash
pipx install --pip-args="--pre" hol-guard
hol-guard --version
```

If `pipx` is unavailable, do not silently install into the project's Python environment. Explain that an isolated CLI install is recommended.

## Guard a Vercel command

1. Build the exact `vercel` command that would otherwise run.
2. Do **not** execute it yet.
3. Inspect that exact command with HOL Guard:

```bash
hol-guard command test '<exact vercel command>' --json
```

4. Read the JSON result.
5. Execute the original Vercel command **exactly once** only when HOL Guard explicitly classifies it as benign and the minimum action is `allow`.
6. For `review`, `block`, unknown/malformed output, CLI errors, or timeouts, do not run the Vercel command. Report the Guard result and ask for the appropriate review/approval path instead.

`hol-guard command test` is side-effect free. It classifies the command; it does not execute the Vercel command, create a final approval, evaluate the complete runtime policy, or record a receipt.

## Operations to inspect

Always inspect production-impacting operations, including:

- production deploys
- production promotion
- production rollback
- deployment removal
- project removal

Also inspect any Vercel command whose effect or target is unclear.

HOL Guard recognizes safe counterparts such as help, project inspection, and promotion status, but still use the exact command rather than paraphrasing it.

## Example

Before running:

```bash
vercel promote my-deployment.vercel.app
```

inspect it:

```bash
hol-guard command test 'vercel promote my-deployment.vercel.app' --json
```

Do not run `vercel promote ...` unless the result is explicitly benign with minimum action `allow`.

## Safety rules

- Never transform a blocked command just to make it pass.
- Never split one risky command into multiple commands to bypass a Guard decision.
- Never treat scanner or CLI failure as approval.
- Never read `.env` files to satisfy Guard.
- Do not claim HOL Guard is a native Vercel runtime integration. This skill uses HOL Guard's command-safety engine before Vercel CLI execution.
- Guard Cloud is optional for this workflow.
- Preserve the user's target project, team, scope, and CLI flags exactly when inspecting the command.

## Install this skill

```bash
npx skills add vercel-labs/agent-skills --skill hol-guard-vercel
```

HOL Guard: https://hol.org/guard
Source: https://github.com/hashgraph-online/hol-guard
