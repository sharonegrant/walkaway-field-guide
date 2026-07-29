# Sharon's Walkaway Setup Guide

## A tested GSD + Ralph + Codex pattern for bounded unattended work

Last verified: July 29, 2026  
Pilot scope: local macOS, disposable Git worktrees, synthetic code only

## What this is

Walkaway is a setup pattern, not a new AI model or orchestration tool. It
combines:

- [GSD Core](https://github.com/open-gsd/gsd-core) for discuss, plan,
  execute, verify, and ship phases with fresh context; and
- [Ralph Orchestrator](https://github.com/mikeyobrien/ralph-orchestrator)
  for bounded iteration and backpressure.

The tested Codex route adds an external budget controller and a separate
read-only verification turn. Nothing here makes an AI agent inherently safe,
correct, or authorized to merge, deploy, publish, spend money, enter secrets,
or operate outside the job manifest.

## The tested flow

```text
Sharon approves scope
        ↓
sanitized job manifest
        ↓
fresh disposable worktree
        ↓
restricted executor ──→ exact-path diff
        ↓                    ↓
protected verify budget → read-only verifier
        ↓
tests + secret scan + policy checks
        ↓
locally accepted patch
        ↓
human review before any external action
```

## What the July 2026 pilot proved

- Exact prompt and configuration hashes can bind an approval.
- Preflight can reject the wrong repo, wrong branch, dirty tree, remotes,
  changed inputs, or unpinned dependencies.
- A child process can receive a scrubbed environment.
- macOS sandbox controls can deny command networking and writes outside the
  disposable repository.
- An outer watchdog can stop the run independently of the orchestrator.
- Tests, diff policy, secret scanning, and a separate verifier can reject a
  false completion claim.
- Actual per-turn usage can calibrate later job budgets.

It did **not** prove a production repository, container or VM isolation,
provider-side spending limits, GitHub rulesets, a draft-PR broker, merge,
deployment, or AgentHub execution.

## Requirements

- Git
- Node.js 22 or later for the tested harness
- Codex CLI
- an existing deterministic test command
- a disposable repository with no remote for the first run
- a way to delete and recreate the test environment

Ralph is optional in the first real-Codex measurement phase. The accepted
trials called the pinned Codex CLI through a custom restricted adapter rather
than Ralph's built-in Codex backend.

## Step 1 — Create a disposable repository

Use a new repository that contains only synthetic code and a small immutable
test. Do not begin with a client, employer, legal, financial, health, family,
credential-bearing, or production codebase.

Before every run, require:

- a `pilot/*` branch;
- a clean working tree;
- zero Git remotes;
- an explicit changed-path allowlist; and
- no symlink that escapes the repository.

## Step 2 — Install GSD in a contained home

The tested release was GSD Core 1.8.0. Its local installer still wrote a
defaults file under `~/.gsd` when run with the normal home directory, so the
verified pattern redirects `HOME` during installation.

```bash
export WALKAWAY_TEMP_HOME="$(mktemp -d)"
HOME="$WALKAWAY_TEMP_HOME" \
  npx @opengsd/gsd-core@1.8.0 --codex --local
```

After installation:

1. verify the project-local GSD files;
2. confirm the real `~/.codex` and `~/.gsd` were unchanged; and
3. remove the disposable home when it is no longer needed.

Use the upstream current installer for a new experiment only after reviewing
its release notes and repeating the containment check.

## Step 3 — Install the tested Ralph release

The tested Ralph CLI package was:

```bash
npm install -g @ralph-orchestrator/ralph-cli@2.10.1
```

The current upstream project may be newer. Treat 2.10.1 as a reproducible
pilot pin, not a permanent recommendation.

## Step 4 — Do not use Ralph 2.10.1's built-in Codex backend

In the exact 2.10.1 package inspected for this pilot, the built-in Codex
backend constructed `codex exec --yolo`. The pilot therefore excluded that
route and used a custom adapter with a restricted executor and separate
read-only verifier.

Do not replace that control with
`--dangerously-bypass-approvals-and-sandbox`.

Reinspect the current upstream backend before changing this rule. A future
version may differ.

## Step 5 — Configure the loop with current keys

The tested Ralph configuration shape used `event_loop` and disabled
auto-merge:

```yaml
event_loop:
  completion_promise: "LOOP_COMPLETE"
  max_iterations: 3
  max_runtime_seconds: 90
  prompt_file: "PROMPT.md"

cli:
  backend: "custom"
  command: "node"
  args:
    - "path/to/restricted-adapter.mjs"
  default_mode: "autonomous"
  idle_timeout_secs: 15

features:
  parallel: false
  auto_merge: false
```

Important corrections to the original guide:

- use `max_runtime_seconds`, not `max_runtime`;
- do not add `checkpoint_interval` unless the current upstream schema
  documents and enforces it;
- also pass `--no-auto-merge` when using Ralph's CLI; and
- enforce non-merge with credential design and repository rules, not config
  text alone.

## Step 6 — Separate execution from verification

Use two roles:

### Restricted executor

- may write only to approved paths in the disposable worktree;
- has command networking disabled;
- receives a scrubbed environment;
- cannot access browser profiles, credentials, cloud-synced data, or unrelated
  directories; and
- cannot push, open a PR, merge, deploy, publish, or change settings.

### Read-only verifier

- receives the approved manifest, final diff, and deterministic test output;
- cannot modify the worktree;
- checks scope, tests, protected paths, secret patterns, and completion claims;
  and
- may reject the executor's self-assessment.

`LOOP_COMPLETE` is evidence to inspect, never authority to accept the build.

## Step 7 — Budget by job phase

The tested starting allocation was:

| Phase | Allocation |
| --- | ---: |
| Research | 15% |
| Planning | 20% |
| Execution | 40% |
| Verification | 25% |

Rules:

1. protect the verification reserve at dispatch time;
2. do not spend unused tokens merely to exhaust the allocation;
3. record estimates and actual usage separately;
4. stop dispatching new turns when the remaining budget cannot cover the
   protected verifier; and
5. supplement turn-boundary controls with provider spending limits when
   available.

The accepted two-job sample produced:

- execution estimate with 20% headroom: **44,621**;
- verification estimate with 20% headroom: **31,348**;
- observed whole-job headroom floor: **80,000**; and
- retained default whole-job ceiling: **100,000**.

This is a small purposive sample, not a universal benchmark.

## Step 8 — Make acceptance mechanical

An accepted result must pass all of these:

- immutable test command;
- exact changed-path allowlist;
- protected-path denylist;
- outside-write sentinel;
- no unexpected Git metadata or remote;
- canary and secret scan;
- no command-network access;
- independent read-only verification; and
- recorded artifact hashes.

Failure of any gate means `not accepted`, even if the code looks plausible or
the agent says it is finished.

## Step 9 — Keep external actions separate

The tested milestone ends with a locally accepted patch. It does not include
GitHub credentials or a PR.

A future PR phase should use a separate least-privilege broker that can:

- access only one throwaway repository;
- create only a branch and draft PR;
- never merge;
- never alter rules, Actions, secrets, collaborators, or visibility; and
- attach only the allowlisted summary, hashes, and test evidence.

Do not grant the executor the broker credential.

## First-run acceptance checklist

- [ ] Synthetic repository only
- [ ] No remote
- [ ] Clean `pilot/*` branch
- [ ] Pinned versions and recorded hashes
- [ ] GSD installer contained in a disposable `HOME`
- [ ] Built-in Ralph Codex backend not used for 2.10.1
- [ ] Command networking denied
- [ ] Outside reads and writes denied
- [ ] Exact changed-path allowlist
- [ ] Protected verification reserve
- [ ] Deterministic tests
- [ ] Independent read-only verifier
- [ ] Secret and canary scans
- [ ] Failure injections pass
- [ ] Two consecutive accepted runs
- [ ] No push, PR, merge, deployment, or publication

## Best fit

- small, well-scoped features;
- deterministic bug fixes;
- bounded refactors;
- repositories with fast, meaningful tests; and
- work where the product decision is already made.

## Poor fit

- ambiguous product strategy;
- production emergencies;
- unreviewed migrations;
- repositories with secrets or sensitive personal data;
- tasks requiring credentials or external side effects;
- high-stakes legal, financial, medical, employment, or compliance decisions;
  and
- any work you cannot restore from a known-good state.

## Tested versions and hashes

| Component | Tested value |
| --- | --- |
| GSD Core | `@opengsd/gsd-core@1.8.0` |
| GSD npm integrity | `sha512-xgz8Er1/uAaWvIkTXBuNNbVO3d113H9dJUysHSz9yP9wCf57AEs7dLgWdaatPeLVDYTbtJEugkxSUBIscEsCrg==` |
| Ralph CLI | `@ralph-orchestrator/ralph-cli@2.10.1` |
| Ralph npm tarball SHA-256 | `709e15bc72f7212c2b71f61f8cbe207cc0c2989cac78d04726e97c03bfe75f24` |
| Ralph macOS arm64 archive SHA-256 | `fd5d25f72ecc7255e07a4d646e49b7d72fe61a5770efb2e4b5a4d66dc3375843` |
| Extracted Ralph binary SHA-256 | `6da1e3cb62d127bbaa8a3e88c50447340a099c2bb63587ffe19337694b60fa77` |
| Codex CLI | `codex-cli 0.146.0-alpha.3.1` |
| Codex binary SHA-256 | `6d8be49e49751554df16572369e636cbe02c84b208cad3dc35528c846eeca223` |
| Automated pilot suite | 35/35 passing |

Hashes identify the artifacts tested on one machine. They are not an
endorsement or a substitute for checking current upstream releases.

## Credit

GSD Core is maintained by Open GSD. Ralph Orchestrator is maintained by Mike
O'Brien and implements the Ralph Wiggum technique associated with Geoffrey
Huntley. Codex CLI is maintained by OpenAI.

This guide contains Sharon Grant's configuration, test design, evidence, and
operating recommendations. It is independent and is not affiliated with or
endorsed by the upstream projects.

