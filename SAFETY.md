# Safety model

Walkaway is intentionally bounded. Its controls are part of the design, not
optional decoration.

## Trust boundaries

- **Owner:** chooses scope and reviews external actions.
- **Manifest builder:** removes unrelated and sensitive context.
- **Executor:** writes only approved paths in a disposable worktree.
- **Verifier:** read-only and independent of the executor's self-rating.
- **Future PR broker:** separate credential and separate phase; not yet tested.

## Denied by default

- credentials and interactive authentication;
- browser profiles and keychains;
- unrelated filesystem access;
- command networking;
- Git remotes during local trials;
- repository settings, rules, Actions, secrets, and collaborators;
- merge, deployment, publication, and permission changes; and
- personal, client, regulated, or production data.

## Why “completion” is not enough

An agent can be mistaken about its own work. Walkaway accepts a result only
when deterministic tests, exact diff policy, secret scanning, filesystem
probes, Git checks, and an independent verifier all agree.

## Reporting a security issue

Do not open a public issue containing credentials, private prompts, model event
streams, local paths, or personal data. Use the upstream projects' private
security-reporting process for vulnerabilities in their software.

This repository contains documentation and aggregate synthetic results only;
it does not ship the private pilot runner.

