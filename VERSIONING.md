# Versioning Strategy

This action follows [Semantic Versioning](https://semver.org/) principles.

## Version Format

`MAJOR.MINOR.PATCH` (e.g., `v1.2.3`)

- **MAJOR**: Breaking changes that require users to modify their workflow configurations
- **MINOR**: New features that are backward compatible
- **PATCH**: Bug fixes and small improvements that don't change the API

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

- `main`: Stable, production-ready code
- `develop`: Development branch for new features
- `release/vX.Y.Z`: Release preparation branches

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
