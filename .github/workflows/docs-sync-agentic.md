---
name: Docs Sync
description: Daily check for documentation that has drifted out of sync with recent code changes, opening a pull request with the necessary updates.
emoji: 📚
on:
  schedule: daily on weekdays
  skip-if-match: 'is:pr is:open in:title "[docs sync]"'

permissions:
  contents: read

strict: true

network: defaults

tools:
  github:
    mode: gh-proxy
    toolsets: [repos]
  cache-memory: true

safe-outputs:
  create-pull-request:
    title-prefix: "[docs sync] "
    labels: [documentation, automation]
    draft: true
    allowed-files:
      - "**/*.md"
    excluded-files:
      - "CHANGELOG.md"
      - ".github/ISSUE_TEMPLATE/**"
      - ".github/pull_request_template.md"
      - ".github/skills/**"
---

# Documentation Sync

Keep this repository's Markdown documentation (`README.md`, `CONTRIBUTING.md`, and per-component READMEs such as `terraform/README.md` and language-runtime `README.md` files) accurate and in sync with the code.

## 1. Determine what changed

Load `/tmp/gh-aw/cache-memory/watermark.json` if it exists. It contains the commit SHA (`last_sha`) processed by the previous run.

- If the file exists, run `git log <last_sha>..HEAD --name-only --no-merges` to list commits and files changed since the last run.
- If the file does not exist (first run) or `<last_sha>` is no longer reachable (e.g. history rewritten), fall back to the last 7 days: `git log --since="7 days ago" --name-only --no-merges`.
- If there are no new commits to inspect, use the `noop` safe output ("no code changes since last run") and still refresh the watermark (see step 4), then stop.

Exclude documentation-only commits (commits that only touched `**/*.md` files) from the set of "code changes" — the goal is to find docs that fell behind *code*, not docs that already changed.

## 2. Compare changed code against documentation

For each area of the repository with code changes (root scripts and patches, `adot/`, `dotnet/`, `go/`, `java/`, `nodejs/`, `python/`, `sample-apps/`, `terraform/`), read the closest relevant documentation file(s) and check for concrete drift caused by the code changes, such as:

- Installation, build, or usage instructions that no longer match updated scripts, commands, or file paths.
- Version numbers, supported runtimes, or dependency references that changed in code but not in docs.
- Configuration options, environment variables, or CLI flags that were added, renamed, or removed in code but not reflected in docs.
- Directory/module structure or file names referenced in docs that were moved, renamed, or removed.
- Links to files or paths within the repository that are now broken because of the change.

Only flag genuine, concrete mismatches directly tied to an inspected code change. Do not rewrite prose, restyle unrelated sections, or make speculative improvements.

## 3. Apply updates and open a pull request

If drift was found:

- Edit only the affected documentation files with the minimal changes needed to make them accurate again.
- Use the `create-pull-request` safe output. In the PR body, list each documentation file changed, the specific inaccuracy fixed, and the commit(s)/file(s) that caused the drift.
- If no drift is found, use the `noop` safe output and briefly state that all documentation is consistent with recent code changes.

## 4. Update the watermark

Regardless of outcome, write `/tmp/gh-aw/cache-memory/watermark.json` with the current `HEAD` commit SHA (`last_sha`) and a `checked_at` timestamp in `YYYY-MM-DD-HH-MM-SS` format (no colons, no `T`, no `Z`), so the next run starts from here.
