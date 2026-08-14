<!-- markdownlint-disable -->

# Hardening Report: HardNorth--github-version-generate/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HardNorth--github-version-generate/v1.4.0** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in ci.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs: `actions/checkout@v4`, `actions/setup-node@v4`. These can be silently changed by the upstream maintainer, enabling supply-chain attacks.

Locations:

- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:18`

### unpinned-uses (severity: high)

Multiple `uses:` references in release.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs: `actions/checkout@v4`, `actions/setup-node@v4`, `HardNorth/github-version-generate@v1`, `mindsers/changelog-reader-action@v1.3.1`, `actions/create-release@v1`, `actions/checkout@v3`. These can be silently changed by upstream maintainers, enabling supply-chain attacks.

Locations:

- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:42`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:75`
- `.github/workflows/release.yml:84`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and the single job also has no `permissions:` key. Without explicit permissions, the workflow inherits the default (potentially broad) token permissions.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key and the single job also has no `permissions:` key. Without explicit permissions, the workflow inherits the default (potentially broad) token permissions. This is especially risky given that this workflow pushes commits, tags, and creates releases.

Locations:

- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Rule (a): The 'Update build' step directly interpolates `${{ env.RELEASE_VERSION }}` inside a `run:` shell command string. Any `${{ ... }}` expression inside a run block is subject to template substitution before the shell parses it, allowing injection of shell metacharacters. Offending line: `git commit -m "Compiled version ${{ env.RELEASE_VERSION }}"`

Locations:

- `.github/workflows/release.yml:57`

### script-injection (severity: high)

Rule (a): The 'Update README.md' step directly interpolates multiple `${{ env.* }}` expressions inside a `run:` shell command string, including `${{ env.README_VERSION_PLACEHOLDER }}`, `${{ env.RELEASE_VERSION }}`, `${{ env.README_TEMPLATE_FILE }}`, and `${{ env.README_FILE }}`. These are substituted before shell parsing, enabling injection. Offending line: `sed 's/${{ env.README_VERSION_PLACEHOLDER }}/${{ env.RELEASE_VERSION }}/g' ${{ env.README_TEMPLATE_FILE }} > ${{ env.README_FILE }}`

Locations:

- `.github/workflows/release.yml:62`

### script-injection (severity: high)

Rule (a): The 'Update CHANGELOG.md' step directly interpolates multiple `${{ env.* }}` expressions (`${{ env.CHANGE_LOG_FILE }}`, `${{ env.TMP_SUFFIX }}`, `${{ env.RELEASE_VERSION }}`) inside `run:` shell command strings. These are substituted before shell parsing, enabling injection.

Locations:

- `.github/workflows/release.yml:69`

### script-injection (severity: high)

Rule (a): The 'Tagging new version' step directly interpolates `${{ env.RELEASE_VERSION }}`, `${{ env.RELEASE_VERSION_MAJOR }}`, and `${{ env.RELEASE_VERSION_MINOR }}` inside `run:` shell command strings. These are substituted before shell parsing, enabling injection.

Locations:

- `.github/workflows/release.yml:83`

### script-injection (severity: high)

Rule (a): The 'Update version file' step directly interpolates `${{ env.CURRENT_VERSION }}`, `${{ env.NEXT_VERSION }}`, `${{ env.VERSION_REPLACE_PATTERN }}`, `${{ env.VERSION_FILE }}`, `${{ env.ADDITIONAL_VERSION_FILE }}`, and `${{ env.TMP_SUFFIX }}` inside `run:` shell command strings. These are substituted before shell parsing, enabling injection. Particularly dangerous: `export CURRENT_VERSION_VALUE=\`echo '${{ env.CURRENT_VERSION }}' | sed -E 's/(.*)/${{ env.VERSION_REPLACE_PATTERN }}/'\``

Locations:

- `.github/workflows/release.yml:103`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across ci.yml and release.yml:

**ci.yml:**
- Added `permissions: contents: read` at top level
- Pinned actions/checkout@v4 → SHA 34e114876b0b11c390a56381ad16ebd13914f8d5
- Pinned actions/setup-node@v4 → SHA 49933ea5288caeca8642d1e84afbd3f7d6820020

**release.yml:**
- Added `permissions: contents: write` at top level (required for git push, tagging, and release creation)
- Pinned actions/checkout@v4 → SHA 34e114876b0b11c390a56381ad16ebd13914f8d5
- Pinned actions/setup-node@v4 → SHA 49933ea5288caeca8642d1e84afbd3f7d6820020
- Pinned HardNorth/github-version-generate@v1 → SHA daf138e2557c0bb49affe49bd7d79671dc8bfb3f
- Pinned mindsers/changelog-reader-action@v1.3.1 → SHA 6624a194a4cad18ad1016631b792753745b0afb5
- Pinned actions/create-release@v1 → SHA 0cb9c9b65d5d1901c1f53e5e66eaf4afd303e70e
- Pinned actions/checkout@v3 → SHA f43a0e5ff2bd294095638e18286ca9a3d1956744
- Fixed script injection in 5 steps ('Update build', 'Update README.md', 'Update CHANGELOG.md', 'Tagging new version', 'Update version file') by moving all ${{ env.* }} expressions into each step's env: block and referencing them as plain shell variables in the run: scripts

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Setup git credentials' step of .github/workflows/release.yml. Moved `${{ secrets.GITHUB_TOKEN }}` out of the `run:` shell command and into an `env:` block as `GITHUB_TOKEN`, then updated the shell command to reference it as `"$GITHUB_TOKEN"` instead of directly interpolating the expression.

