# Publish NPM Package on Release

[![github stars](https://badgen.net/github/stars/phucbm/publish-npm-action?icon=github)](https://github.com/phucbm/publish-npm-action/)
[![github license](https://badgen.net/github/license/phucbm/publish-npm-action?icon=github)](https://github.com/phucbm/publish-npm-action/blob/main/LICENSE)
[![Made in Vietnam](https://raw.githubusercontent.com/webuild-community/badge/master/svg/made.svg)](https://webuild.community)

A GitHub Action that automatically builds, tests, and publishes NPM packages when you create a GitHub release.

## Features
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│    SETUP    │───▶│ VALIDATION  │───▶│    BUILD    │───▶│   PUBLISH   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       ▼                   ▼                   ▼                   ▼
   Install deps        Auto-detect         Execute build       Update version
   Configure Node      test scripts        commands or skip    Commit artifacts
                       Run if found                            Publish to NPM
                       Skip if missing                         Success notify
```
## Quick Start

1. **Add NPM Token to Secrets**
   - Go to [npmjs.com](https://www.npmjs.com) → Profile → Access Tokens
   - Create an "Automation" token with "Publish" permission
   - Add it to GitHub: Settings → Secrets → Actions → `NPM_TOKEN`

2. **Create Workflow File**
   Create `.github/workflows/publish.yml`:

```yaml
name: Publish on Release

on:
  release:
    types: [ published ]  # Triggers when you publish a release through GitHub UI

permissions:
  contents: write  # To push updated files back to main
  packages: write  # For GitHub packages (optional)

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          ref: main  # Always checkout main branch, not the tag
          token: ${{ secrets.GITHUB_TOKEN }}
          fetch-depth: 0  # Fetch all history

      - name: Publish NPM Package
        uses: phucbm/publish-npm-action@v1  # Docs https://github.com/phucbm/publish-npm-action
        with:
          npm-token: ${{ secrets.NPM_TOKEN }}
```

3. **Create a Release**
   - Go to your repo → Releases → Create a new release
   - Tag version: `v1.0.0` (or any semantic version)
   - Publish the release
   - Watch the action automatically publish to NPM! 🎉

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `npm-token` | NPM authentication token | ✅ Yes | - |
| `github-token` | GitHub token for repository access | ❌ No | `${{ github.token }}` |
| `node-version` | Node.js version to use | ❌ No | `18` |
| `pnpm-version` | PNPM version to use | ❌ No | `8` |
| `build-command` | Build command to run | ❌ No | `pnpm build` |
| `install-command` | Install command to run | ❌ No | `pnpm install --no-frozen-lockfile` |
| `test-command` | Test command to run | ❌ No | `pnpm test` |
| `publish-access` | NPM publish access level | ❌ No | `public` |
| `skip-tests` | Skip running tests even if available | ❌ No | `false` |
| `output-dir` | Output directory for built files | ❌ No | `dist/` |
| `target-branch` | Branch to push updated files to | ❌ No | `main` |
| `commit-files` | Additional files/patterns to commit | ❌ No | `` |

## Outputs

| Output | Description |
|--------|-------------|
| `package-name` | Name of the published package |
| `version` | Version that was published |
| `npm-url` | NPM package URL |
| `tests-run` | Whether tests were executed |

## Advanced Usage

All available options (enable only what you need):

```yaml
- name: Publish NPM Package
  uses: phucbm/publish-npm-action@v1
  with:
    npm-token: ${{ secrets.NPM_TOKEN }}
    # github-token: ${{ secrets.CUSTOM_TOKEN }}        # Custom GitHub token, default is ${{ github.token }}
    # node-version: '20'                               # Node.js version, default is '18'
    # pnpm-version: '9'                                # PNPM version, default is '8'
    # build-command: 'npm run build:prod'              # Build command, default is 'pnpm build'
    # install-command: 'npm ci'                        # Install command, default is 'pnpm install --no-frozen-lockfile'
    # test-command: 'npm run test:ci'                  # Test command, default is 'pnpm test'
    # publish-access: 'restricted'                     # NPM access level, default is 'public'
    # skip-tests: 'true'                               # Skip tests, default is 'false'
    # output-dir: 'build/'                             # Output directory, default is 'dist/'
    # target-branch: 'develop'                         # Target branch, default is 'main'
    # commit-files: 'CHANGELOG.md docs/ types/'        # Additional files to commit, default is ''
```

### Using Outputs
```yaml
- name: Publish NPM Package
  id: publish
  uses: phucbm/publish-npm-action@v1
  with:
    npm-token: ${{ secrets.NPM_TOKEN }}

- name: Notify Success
  run: |
    echo "Published ${{ steps.publish.outputs.package-name }}@${{ steps.publish.outputs.version }}"
    echo "Available at: ${{ steps.publish.outputs.npm-url }}"
```

## How It Works

```
📋 SETUP PHASE
   └── Install dependencies with your preferred package manager
   └── Configure Node.js environment

🧪 VALIDATION PHASE  
   └── Auto-detect test scripts
   └── Run tests only if they exist
   └── Skip gracefully if no tests found

🏗️ BUILD PHASE
   └── Execute custom build commands
   └── Skip build entirely if not needed
   └── Support any build system

📤 PUBLISH PHASE
   └── Update package.json version
   └── Commit build artifacts  
   └── Publish securely to NPM
   └── Notify success with package URL
```

## Requirements

- Your repository must have a `package.json` file
- NPM token must be added to GitHub Secrets as `NPM_TOKEN`
- Repository must use semantic versioning for releases (e.g., `v1.2.3`)

## Troubleshooting

**NPM Authentication Failed**
- Ensure your NPM token has "Automation" type with "Publish" permission
- Verify the token is correctly added to GitHub Secrets

**Build Fails**
- Check your build command in package.json
- Customize `build-command` input if needed

**Tests Fail**
- Fix your tests or use `skip-tests: 'true'` to skip them temporarily

**Permission Denied**
- Ensure your workflow has `contents: write` permission

## License

MIT License - feel free to use in your projects!

## Contributing

Issues and pull requests welcome!
