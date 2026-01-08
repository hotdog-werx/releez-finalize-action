# Releez Finalize Action

Finalize a GitHub release with `releez`, including tags, release notes, and
artifact-friendly version outputs.

## What this action does

- Installs `releez` via `uv`
- Generates release notes and (optionally) tags the release
- Produces release version outputs plus semver/docker/PEP440 variants

## Prerequisites

- `actions/checkout` with full history (for git tags and `git-cliff`)
- `uv` installed in the runner environment
- A `git-cliff` config (such as `cliff.toml`) in your repo

## Usage

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: astral-sh/setup-uv@v2
      - name: Finalize release
        id: finalize
        uses: hotdog-werx/releez-finalize-action@v0
        with:
          releez-version: 0.1.2
          dry-run: false
      - name: Show release notes
        run: |
          echo "${{ steps.finalize.outputs.release-notes }}"
```

## Inputs

| Input | Description | Required | Default |
| --- | --- | --- | --- |
| `releez-version` | Git ref for `releez` (tag, branch, or SHA). | No | `0.1.2` |
| `version-override` | Override for the next version. | No | `""` |
| `alias-versions` | Alias tag strategy: `none`, `major`, or `minor`. | No | `""` |
| `dry-run` | If `true`, preview the release without tagging. | No | `false` |
| `github-token` | Token for GitHub authentication. | No | `${{ github.token }}` |

Notes:
- Inputs are strings; use `true`/`false` for `dry-run`.
- `releez-version` is passed as a git ref to `uv tool install`, so use a valid
  ref for the `releez` repo (for example `0.1.2` or `v0.1.2`, depending on tags).
- Leave `alias-versions` empty to use repo config; set it explicitly to override
  (for example `none`).

## Outputs

| Output | Description |
| --- | --- |
| `release-version` | The released version string. |
| `release-notes` | The markdown release notes for the version. |
| `release-preview` | Preview output when `dry-run` is `true`; includes v-prefixed alias tags (for example `v1`, `v1.2`) when enabled; empty otherwise. |
| `semver-versions` | Semver versions generated for the release. |
| `docker-versions` | Docker-friendly versions generated for the release. |
| `pep440-versions` | PEP440 (Python) versions generated for the release. |

## Dry run example

```yaml
- name: Preview release
  id: finalize
  uses: hotdog-werx/releez-finalize-action@v0
  with:
    dry-run: true
```

## Alias tags example

```yaml
- name: Preview alias tags
  id: finalize
  uses: hotdog-werx/releez-finalize-action@v0
  with:
    dry-run: true
    alias-versions: minor
- name: Show preview
  run: |
    echo "${{ steps.finalize.outputs.release-preview }}"
```

## About releez

This action wraps the `releez` CLI, which uses `git-cliff` for versioning and
changelog generation. Review the `releez` docs for configuration details.
