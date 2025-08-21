# Windows SDK UCRT header sync (template)

This template repository contains a reusable GitHub Actions workflow that detects changes in selected Windows SDK UCRT headers on a Windows runner and proposes a pull request to update them in your repository.

Important: Review the license terms for any SDK headers before committing them to a public repository. By default, this workflow does not commit changes; it only uploads the headers as a build artifact. You must explicitly enable commit mode if you have the rights to redistribute these files.

## What it does

- Locates the latest installed Windows 10 SDK on the `windows-latest` runner.
- Looks for these UCRT headers: `io.h`, `corecrt_io.h`, `corecrt_share.h`, `corecrt_wio.h`.
- Compares their hashes to any copies in `docs/` in your repo.
- If different or missing, uploads the new headers as an artifact.
- Optionally (when enabled) stages the changed headers into `docs/` and opens/updates a PR using `peter-evans/create-pull-request`.

## Repository structure

```
.github/workflows/header-sync.yml   # The workflow
README.md                           # This file
```

## Usage

1. Copy the `.github/workflows/header-sync.yml` file into your repository.
2. Commit and push.
3. By default, the workflow runs weekly and also supports manual dispatch.
4. The workflow will upload an artifact `sdk-ucrt-headers` when changes are detected.

### Enable commit mode (optional)

If you are permitted to commit these headers into your repo, set a repository variable to enable commit mode:

- Repository Settings → Secrets and variables → Variables
- Add variable `HEADER_SYNC_COMMIT` with value `true`

On the next run, the workflow will:
- Copy changed headers into `docs/`
- Open or update a PR on branch `header-sync`

## Customization

- To change which headers are tracked, edit the `$targetNames` list in the PowerShell step.
- To point at a different SDK root, adjust `$kitsRoot` (defaults to `C:\Program Files (x86)\Windows Kits\10\Include`).
- To change the PR branch name or the commit message, update the `create-pull-request` step inputs.

## Why not upload-artifact only?

- `actions/upload-artifact` is used here to make the updated headers available for review without committing. This avoids licensing issues for public repositories.
- If you enable commit mode, the workflow uses `peter-evans/create-pull-request` to commit and open a PR in a clean, idempotent way (no manual `git push` or duplicate PRs when re-run).

## Caveats

- The exact header locations on the runner can change; the workflow searches the installed Windows 10 SDK and chooses the highest version that includes `ucrt/`.
- The workflow requires `contents` and `pull-requests` write permissions only when commit mode is enabled.
