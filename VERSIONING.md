# Versioning Strategy

This action follows [Semantic Versioning](https://semver.org/) principles.

## Version Format

`MAJOR.MINOR.PATCH` (e.g., `v1.2.3`)

- **MAJOR**: Breaking changes that require users to modify their workflow configurations
- **MINOR**: New features that are backward compatible
- **PATCH**: Bug fixes and small improvements that don't change the API

## What counts as breaking (MAJOR) vs. not

For a composite action, "breaking" means a change to the **inputs/outputs contract**, not
internal implementation:

| Change | Bump |
|---|---|
| Remove an input, rename an input, change an input's meaning, remove an output | MAJOR |
| Add a new optional input or a new output | MINOR |
| Change a default value that alters existing callers' behavior | MINOR (call it out loudly in the changelog) or MAJOR if the behavior shift is significant |
| Fix incorrect/broken behavior (retries, error handling, status parsing, etc.) with no input/output contract change | PATCH |
| Internal refactor, comment/doc changes, dependency version bump (e.g. `slackapi/slack-github-action`) with no observable behavior change | PATCH |

Every release, however small, gets an entry in [CHANGELOG.md](CHANGELOG.md) before tagging.

## Release Process

### 1. Initial Release (v1.0.0)
```bash
git tag v1.0.0
git push origin v1.0.0
```

### 2. Create GitHub Release
1. Go to GitHub → Releases → "Draft a new release"
2. Choose the tag (e.g., `v1.0.0`)
3. Title: `v1.0.0 - Initial Release`
4. Description: List features and changes
5. ✅ Check "Publish this Action to the GitHub Marketplace"
6. Select category (e.g., "CI/CD")
7. Click "Publish release"

### 3. Major Version Tags
For easier user adoption, maintain major version tags:

```bash
# After releasing v1.0.0
git tag v1
git push origin v1

# After releasing v1.1.0
git tag -d v1
git tag v1
git push origin :refs/tags/v1
git push origin v1
```

This allows users to reference `@v1` and automatically get the latest v1.x.x version.

## Branch Strategy

Trunk-based: everything ships from `main`, there's no standing `develop` or
`release/*` branch. For a single composite `action.yaml` with no build step, the
overhead of a full git-flow setup isn't worth it — a PR into `main` plus a tag is
the whole release.

## Quick release checklist (patch or minor)

```bash
# 1. Move the CHANGELOG.md [Unreleased] section under a new version heading, commit it.
# 2. Tag the release:
git tag vX.Y.Z
git push origin vX.Y.Z

# 3. Move the floating major tag to point at it (see below), so @v1 users pick it up:
git tag -d v1
git tag v1
git push origin :refs/tags/v1
git push origin v1

# 4. Draft the GitHub release from the tag (gh CLI or the UI) — see "Create GitHub
#    Release" above. Paste the CHANGELOG.md section for that version as the body.
gh release create vX.Y.Z --title "vX.Y.Z" --notes-file <(sed -n '/## \[X.Y.Z\]/,/## \[/p' CHANGELOG.md)
```

## User References

Users can reference the action in multiple ways:

- `@v1` - Latest v1.x.x release (recommended for production)
- `@v1.2.3` - Specific version (recommended for strict environments)
- `@main` - Latest commit (not recommended for production)

## Example Release Workflow

```yaml
name: Release
on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false
```
