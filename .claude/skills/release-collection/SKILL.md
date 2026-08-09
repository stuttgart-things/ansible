---
name: release-collection
description: Release an Ansible collection from stuttgart-things/ansible — cut a release, re-release without a code change, check why a release did not appear, or add a new collection. Use for "release the container collection", "publish baseos", "why is there no new release", "rebuild the collection", "bump the dagger module".
---

# Release a collection

## The one thing to know first

**Releases are automatic on merge to `main`.** `.github/workflows/main-collection-build.yaml`
fires on any push to `main` touching `collections/**`, works out which collections
changed, and publishes a GitHub release for each.

So the answer to "how do I release X" is almost always **merge a PR that touches
`collections/X/`**. Do not reach for a manual build or a `gh release create` — those
exist for narrow cases covered below, and running one by hand can publish a second
release for the same content.

Never run `task release` without saying what it does first and getting the user's
agreement — it auto-merges the PR. `task build-collection` is safe by default but
publishes when given `PUBLISH=true`; treat that form the same way.

## Normal flow

```bash
git checkout -b <type>/<slug>
# edit collections/<name>/...
git add collections/<name> && git commit
git push -u origin <type>/<slug>
gh pr create --base main --title "<type>(<scope>): <description>" --body "..."
```

Then merge. CI does the rest — no manual step, no version to pick.

PR titles and commits follow **conventional commits with a scope**, matching the
existing history (`fix(baseos): ...`, `feat(machinery): ...`, `chore(deps): ...`).
There is no semantic-release in this repo, so the prefix does not gate the release
— the version comes from the date and commit count either way. It is a readability
convention, but an enforced one by review.

## Version and tag scheme

```
sthings-<collection>-<YY>.<M><DD>.<commit-count>
```

`YY` = 2-digit UTC year, `M``DD` = month with **no leading zero** then zero-padded
day (`date -u +%-m%d`), count = `git rev-list --count HEAD`. Recent examples:

```
sthings-baseos-26.804.1193
sthings-container-26.804.1191
```

The commit count makes versions monotonic and collision-free: two different commits
can never produce the same tag, which is why the workflow deliberately runs with **no
concurrency group**. Do not add one — see the long comment at the top of
`main-collection-build.yaml` for the merge-burst incident that caused (#1073).

Releases up to `sthings-*-26.2.675` were tagged with a trailing `.tar.gz` in the tag
name itself, so links to them repeat it in the tag segment — correct, not malformed.
Current tags do not carry it, so a pin URL is `.../download/<tag>/<tag>.tar.gz`.

The `sthings.*` pins in the root `requirements.yaml` are consumer-facing **and** the
dependency baseline CI installs before overwriting the collection under test. A
stale pin fails nothing — it quietly tests new code against old siblings
(`molecule/container` imports `sthings.baseos.users`). Refresh them when they drift;
the deliberate reason `molecule/requirements.yaml` omits `sthings.*` is documented
in that file.

## How CI decides what to release

```bash
git diff --name-only "${BEFORE}" "${SHA}" -- collections \
  | awk -F/ 'NF>=3 && $1=="collections" {print $1"/"$2}' | sort -u
```

Each changed path is reduced to its collection root, so a nested file still maps to
the collection that owns it. Two consequences worth knowing:

- **`NF>=3` means a file directly under `collections/` releases nothing.** Editing
  `collections/README.md` matches the `collections/**` path filter, triggers the
  workflow, and resolves to zero buildable collections. The workflow writes a
  "No collection released" block to the job summary rather than passing silently —
  that reporting exists because this failure hid twice (#1048, #11). If a user says
  "I merged but nothing released", check this first.
- Detection is a **per-push diff**. A run that never completes takes its collections
  with it.

## Verifying a release

```bash
gh release list --limit 10
gh run list --workflow=main-collection-build.yaml --limit 5
```

A release should exist within a few minutes of the merge, tagged as above. If the
run succeeded but no release appeared, read the run's step summary — the
"No collection released" case reports there and nowhere else.

## The manual paths, and when each is right

### Re-release without a code change → `dispatch-collection-build.yml`

This is the **preferred** manual path. Use it after a Dagger module bump changes the
built artifact, or when a release needs rebuilding from an unchanged tree.

```bash
gh workflow run dispatch-collection-build.yml \
  -f collection=container \
  -f branch-name=main \
  -f publish-release=true
```

`publish-release` defaults to `false` — leave it false to just check the build.

### Local build → `task build-collection`

```bash
task build-collection                 # builds, installs locally, prints the tag it WOULD cut
PUBLISH=true task build-collection    # builds AND publishes a GitHub release
```

Publishing is opt-in and the guard fails closed — anything that is not exactly
`true` (including a `PUBLISH=TRUE` typo) builds only. Before #1095 this task always
published, so older notes and habits may assume otherwise.

Requirements: an activated venv, `gum`, `dagger`, `GITHUB_TOKEN` exported, and a
**non-shallow clone** — the task hard-fails on a shallow repo because the commit
count would be wrong.

It uses the same Dagger entrypoint as CI (`run-collection-build-pipeline`) so the
local and CI build paths cannot drift.

### `task release`

Runs `git:commit`, closes any existing PR for the branch, opens a new one, waits for
checks, then `gh pr merge --auto --rebase --delete-branch`. **It auto-merges.**

It also titles the PR after the branch name, which does not match the repo's
conventional-commit history. For anything that wants a real title or a review, open
the PR by hand instead.

## What the PR build does

`pr-collection-build.yaml` builds with `publish-release: false` — the artifact is
uploaded for review, never published. It also runs Molecule, but only for scenarios
in the hardcoded list `^collections/(baseos|container|rke)$`; `awx` needs a kind
cluster plus Vault credentials so it runs nightly instead
(`nightly-molecule-awx.yaml`).

## Adding a new collection

1. `collections/<name>/collection.yaml` — `namespace: sthings`, plus `name`,
   `authors`, `description`, `license`, `tags`, repo URLs, and `requirements:` (a
   literal block holding a galaxy `roles:` list; use `roles: []` if it pulls none).
2. One or more playbook YAML files using the repo's DSL (`play:`, `templates:`,
   `vars:` — see `CLAUDE.md`; template names get `.yaml` appended by the build, so
   `src:` references must include the suffix).
3. A `README.md`.

`task build-collection` discovers it automatically — `ALL_COLLECTIONS` is
`ls ./collections`. But **two places hardcode collection names** and will silently
skip a new one:

- `pr-collection-build.yaml` → the `MOLECULE_FOLDERS` grep. No molecule gating until
  the name is added.
- `Taskfile.yaml` → the `run-molecule` task's `gum choose "baseos" "container" "awx"`.

Before adding a collection, ask whether the plays actually need a separate one. A
collection buys nothing operationally here — all collections build from the same
pipeline off the same commit count, and versioning is repo-wide — so the only real
gain is naming. Duplicating a playbook across two collections to get a nicer name
costs permanent drift and is not worth it.

## Troubleshooting

| Symptom | Cause |
|---|---|
| Merged, workflow ran green, no release | Every changed path was directly under `collections/` (`NF>=3` filter). Check the run's step summary. |
| No workflow run at all | Nothing matched the `collections/**` path filter. |
| A collection in a merge burst never released | A run was cancelled and took its diff with it. Re-cut via `dispatch-collection-build.yml`. Do not re-add a concurrency group. |
| `task build-collection` fails on shallow repo | Deliberate guard — the commit count would be wrong. `git fetch --unshallow`. |
| Molecule did not run on a PR | Collection is not in the `baseos|container|rke` allowlist. |
| Version looks like it went backwards | It cannot — the count is monotonic. Compare the count field, not the date. |
