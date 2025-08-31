# Generic file sync workflow (template)

This template repository provides a **generic GitHub Actions workflow** that copies (or generates then copies) files from the runner file system into your repository, either:

1. As a downloadable artifact (review-only mode), or
2. As a pull request with committed changes (commit mode).

It is useful for:

- Capturing SDK / tool output headers, schemas, OpenAPI specs, generated code, license inventories, etc.
- Normalizing / refreshing vendored third‑party text files.
- Periodically syncing externally generated metadata into your repo on a schedule.

By default it **does not commit** anything. You must explicitly enable commit mode via an input or repository variable.

> Always ensure you have redistribution rights for any files you commit from a hosted runner.

## What it does

- Accepts a newline‑separated list of source file paths and/or glob patterns (`sources`).
- Expands globs and collects the matching files on the runner.
- Hash‑compares each candidate with the existing copy under a destination directory in your repo (default: `docs/`).
- Stages only changed / new files into a temporary folder.
- Publishes them as an artifact (`synced-files`).
- Optionally (commit mode) copies them into the repo working tree and opens/updates a PR using `peter-evans/create-pull-request`.

## Inputs (workflow_dispatch)

| Input | Purpose | Default |
|-------|---------|---------|
| `sources` | Newline separated file paths or globs evaluated on the runner. | (none) |
| `dest-dir` | Destination directory (relative to repo root). | `docs` |
| `preserve-dirs` | `true` to preserve directory structure relative to `base-dir`. | `false` |
| `base-dir` | Base path stripped when preserving directory structure. | (none) |
| `commit` | `true` to commit via PR. | `false` |
| `pr-branch` | Branch name for PR. | `file-sync` |
| `pr-title` | PR title prefix. | `File sync update` |
| `pr-body-extra` | Extra body text appended to PR. | (none) |

## Repository variables (fallbacks)

You can set these in: Settings → Secrets and variables → Variables. Each maps to the input of the same semantic meaning and is used when the dispatch input is empty:

- `SYNC_SOURCES`
- `FILE_SYNC_DEST_DIR`
- `FILE_SYNC_PRESERVE_DIRS`
- `FILE_SYNC_BASE_DIR`
- `FILE_SYNC_COMMIT`
- `FILE_SYNC_PR_BRANCH`
- `FILE_SYNC_PR_TITLE`
- `FILE_SYNC_PR_BODY_EXTRA`

## Enabling commit mode

Either:

1. Provide the input `commit: true` in a manual dispatch, or
2. Set repository variable `FILE_SYNC_COMMIT=true`.

The workflow (when changes exist) will:

- Copy changed files into the working tree.
- Open or update a PR on the configured branch (default `file-sync`).
- Include only changed file paths in the PR.

## Adding a generation step

If the files need to be generated first (e.g., run a tool that outputs JSON or headers), add a step **before** the "Resolve sources and detect changes" step in your copy of the workflow. Example:

```yaml
		- name: Generate OpenAPI
			run: |
				./scripts/build-openapi.sh > output/openapi.json
			shell: bash

		- name: Sync generated spec
			uses: ./.github/workflows/file-sync.yml  # if extracted into a reusable workflow
```

Point your `sources` at `output/openapi.json` (or a glob like `output/*.json`).

## Usage quick start

1. Copy `file-sync.yml` into your repo. It should be stored in `.github\workflows\`
2. Commit & push.
3. (Optional) Set repository variable `SYNC_SOURCES` to something like:

```
C:\\Program Files (x86)\\Windows Kits\\10\\Include\\**\\ucrt\\*.h
```

4. Manually dispatch the workflow or wait for the scheduled run.
5. Download the `synced-files` artifact to inspect changes.
6. Enable commit mode once you're satisfied.

## Customization notes

- The workflow currently runs on `windows-latest` (PowerShell 7 is used). PowerShell is cross‑platform; you can adapt `runs-on: ubuntu-latest` if your paths are Linux style.
- Globs are expanded with `Get-ChildItem -Recurse`; adjust if you need directory exclusions.
- Only added / changed files are considered. Deletions (files that existed previously but no longer match) are not removed automatically (you can add a cleanup step if required).
- Commit messages & PR title use your supplied title plus a count of changed files.

## Limitations / Caveats

- Very large file sets may increase run time; prefer targeted globs.
- Binary files are supported (hash compare) but line-level diffs in PR will be less readable.
- If both an input and variable are set for the same concept, the input wins.
- Deletions are not yet auto-pruned (future enhancement idea: optional `delete-missing` flag).

## License & attribution

Distributed under the MIT License (see `LICENSE`).

