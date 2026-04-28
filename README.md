# Publish NPM Package on Release

[![github stars](https://badgen.net/github/stars/phucbm/publish-npm-action?icon=github)](https://github.com/phucbm/publish-npm-action/)
[![github license](https://badgen.net/github/license/phucbm/publish-npm-action?icon=github)](https://github.com/phucbm/publish-npm-action/blob/main/LICENSE)
[![Made in Vietnam](https://raw.githubusercontent.com/webuild-community/badge/master/svg/made.svg)](https://webuild.community)

A GitHub Action that automatically builds, tests, and publishes your NPM package whenever you create a GitHub release.

---

## Setup

There are two ways to authenticate with NPM. Pick one:

- **OIDC Trusted Publishing** *(recommended)* — no token stored anywhere, uses GitHub's identity directly
- **NPM Token** — classic long-lived token stored as a GitHub secret

---

### Option A: OIDC Trusted Publishing

#### Step 1 — Add the workflow file

Create `.github/workflows/publish.yml` in your repo:

```yaml
name: Build and Publish NPM package on release

on:
  release:
    types: [ published ]  # Triggers when you publish a release through GitHub UI
  workflow_dispatch:      # Enables manual trigger via GitHub web

permissions:
  contents: write  # To push updated files back to main
  packages: write  # For GitHub packages (optional)
  id-token: write  # Required for OIDC trusted publishing

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v6
        with:
          ref: main  # Always checkout main branch, not the tag
          token: ${{ secrets.GITHUB_TOKEN }}
          fetch-depth: 0

      - name: Publish NPM Package
        uses: phucbm/publish-npm-action@v1  # Docs https://github.com/phucbm/publish-npm-action
        with:
          skip-tests: ${{ github.event.inputs.skip-tests || 'false' }}
```

#### Step 2 — Configure a trusted publisher on npmjs.com

1. Go to [npmjs.com](https://www.npmjs.com) and open your package page
2. Click **Settings** → **Trusted Publishers** → **Add a publisher**
3. Select **GitHub Actions** and fill in:
   - **Owner**: your GitHub username or org
   - **Repository**: your repo name
   - **Workflow filename**: `publish.yml`
4. Save

> If you haven't published the package yet, do a manual `npm publish` first so the package exists, then add the trusted publisher.

#### Step 3 — Create a release

1. Go to your repo on GitHub → **Releases** → **Create a new release**
2. Set a tag like `v1.0.0` (semantic version, must start with `v`)
3. Click **Publish release**
4. The action runs automatically and publishes to NPM

---

### Option B: NPM Token

#### Step 1 — Add the workflow file

Create `.github/workflows/publish.yml` in your repo:

```yaml
name: Build and Publish NPM package on release

on:
  release:
    types: [ published ]
  workflow_dispatch:

permissions:
  contents: write
  packages: write

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v6
        with:
          ref: main
          token: ${{ secrets.GITHUB_TOKEN }}
          fetch-depth: 0

      - name: Publish NPM Package
        uses: phucbm/publish-npm-action@v1  # Docs https://github.com/phucbm/publish-npm-action
        with:
          npm-token: ${{ secrets.NPM_TOKEN }}
          skip-tests: ${{ github.event.inputs.skip-tests || 'false' }}
```

#### Step 2 — Create an NPM token

1. Go to [npmjs.com](https://www.npmjs.com) → click your avatar → **Access Tokens**
2. Click **Generate New Token** → choose **Automation**
3. Copy the token

#### Step 3 — Add the token to GitHub Secrets

1. Go to your repo on GitHub → **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `NPM_TOKEN`, Value: paste the token you copied
4. Save

#### Step 4 — Create a release

1. Go to your repo → **Releases** → **Create a new release**
2. Set a tag like `v1.0.0` (semantic version, must start with `v`)
3. Click **Publish release**
4. The action runs automatically and publishes to NPM

---

## How It Works

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

---

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `npm-token` | NPM token. Omit to use OIDC trusted publishing. | — |
| `github-token` | GitHub token for repo access | `${{ github.token }}` |
| `node-version` | Node.js version | `18` |
| `pnpm-version` | PNPM version | `8` |
| `build-command` | Build command (use `''` to skip build) | `pnpm build` |
| `install-command` | Install command | `pnpm install --no-frozen-lockfile` |
| `test-command` | Test command | `pnpm test` |
| `publish-access` | NPM access level | `public` |
| `skip-tests` | Skip running tests | `false` |
| `output-dir` | Output directory for built files | `dist/` |
| `target-branch` | Branch to push updated files to | `main` |
| `commit-files` | Extra files/patterns to commit | — |

## Outputs

| Output | Description |
|--------|-------------|
| `package-name` | Name of the published package |
| `version` | Version that was published |
| `npm-url` | NPM package URL |
| `tests-run` | Whether tests were executed |

---

## Advanced Usage

All available options (enable only what you need):

```yaml
- name: Publish NPM Package
  uses: phucbm/publish-npm-action@v1
  with:
    npm-token: ${{ secrets.NPM_TOKEN }}
    # github-token: ${{ secrets.CUSTOM_TOKEN }}
    # node-version: '20'
    # pnpm-version: '9'
    # build-command: 'npm run build:prod'
    # install-command: 'npm ci'
    # test-command: 'npm run test:ci'
    # publish-access: 'restricted'
    # skip-tests: 'true'
    # output-dir: 'build/'
    # target-branch: 'develop'
    # commit-files: 'CHANGELOG.md docs/ types/'
```

### Using outputs

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

---

## Troubleshooting

**Version already set in package.json** — The action uses `--allow-same-version` so this is fine.

**NPM authentication failed**
- *Token:* Check the token type is "Automation" with publish permission and the secret name is `NPM_TOKEN`
- *OIDC:* Check `id-token: write` is in your workflow permissions and the trusted publisher is configured on npmjs.com with the exact workflow filename

**Build fails** — Check your build script in `package.json`, or set `build-command` to match.

**Tests fail** — Fix the tests or set `skip-tests: 'true'` temporarily.

**Permission denied** — Ensure `contents: write` is set in your workflow permissions.

---

## License

MIT License - feel free to use in your projects!
