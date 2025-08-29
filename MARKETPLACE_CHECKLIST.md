# GitHub Marketplace Publishing Checklist

Use this checklist to ensure your action is ready for marketplace publication.

## ✅ Pre-Publishing Requirements

### Repository Setup
- [ ] Repository is public
- [ ] Repository contains only one action
- [ ] Repository has a clear, descriptive name
- [ ] 2FA is enabled on your GitHub account

### Required Files
- [x] `action.yaml` or `action.yml` at repository root
- [x] `README.md` with comprehensive documentation
- [x] `LICENSE` file
- [x] Optional: `VERSIONING.md` for version strategy

### Action Metadata (action.yaml)
- [x] Unique `name` field (not conflicting with existing actions)
- [x] Clear, concise `description`
- [x] Proper `branding` section with icon and color
- [x] Well-documented `inputs` with descriptions
- [x] Clear `outputs` with descriptions
- [x] Valid `runs` configuration

### Documentation (README.md)
- [x] Clear description of what the action does
- [x] Usage examples (basic and advanced)
- [x] Input/output documentation
- [x] Setup instructions
- [x] Contribution guidelines
- [x] License information
- [x] Support information

## 🚀 Publishing Steps

### 1. Final Code Review
- [ ] Test the action in a real workflow
- [ ] Verify all links in README work correctly
- [ ] Check that examples in README are accurate
- [ ] Ensure action handles errors gracefully

### 2. Create Initial Release
- [ ] Create and push a version tag (e.g., `v1.0.0`)
- [ ] Go to GitHub → Releases → "Draft a new release"
- [ ] Select the version tag
- [ ] Write release notes describing features
- [ ] ✅ Check "Publish this Action to the GitHub Marketplace"
- [ ] Select appropriate category
- [ ] Accept GitHub Marketplace Developer Agreement
- [ ] Publish the release

### 3. Post-Publication
- [ ] Verify action appears in marketplace
- [ ] Test marketplace installation
- [ ] Update README with marketplace badge
- [ ] Create major version tag (`v1`) for easier user adoption

## 📝 Release Notes Template

```markdown
## Features
- Feature 1 description
- Feature 2 description

## Improvements
- Improvement 1
- Improvement 2

## Bug Fixes
- Fix 1
- Fix 2

## Breaking Changes (if any)
- Breaking change 1
- Migration guide

## Usage
See [README.md](README.md) for detailed usage instructions.
```

## 🏷️ Marketplace Categories

Choose the most appropriate category:
- **CI/CD**: For continuous integration and deployment
- **Code Quality**: For code analysis and quality tools
- **Deployment**: For deployment and infrastructure
- **Monitoring**: For monitoring and observability
- **Productivity**: For workflow productivity tools
- **Publishing**: For publishing and distribution
- **Testing**: For testing frameworks and tools
- **Utilities**: For general-purpose utilities

## 🔍 Final Verification

After publishing, verify:
- [ ] Action is searchable in marketplace
- [ ] Action page displays correctly
- [ ] Installation instructions work
- [ ] Documentation renders properly
- [ ] All links function correctly

## 🆘 Troubleshooting

### Common Issues
1. **Name conflict**: Choose a more unique name
2. **Invalid branding**: Use valid Feather icons and supported colors
3. **Missing 2FA**: Enable two-factor authentication
4. **Invalid action.yaml**: Validate YAML syntax
5. **Repository not public**: Make repository public

### Support Resources
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Marketplace Publishing Guide](https://docs.github.com/en/actions/how-tos/create-and-publish-actions/publish-in-github-marketplace)
- [Action Metadata Syntax](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions)
