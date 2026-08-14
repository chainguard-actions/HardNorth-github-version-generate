<!-- markdownlint-disable -->

# Hardening Report: HardNorth--github-version-generate/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HardNorth--github-version-generate/v1.4.1** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag/version refs instead of full 40-character SHA commit digests, making the workflow vulnerable to supply-chain attacks if the referenced action tag is moved or compromised.

Failing references in .github/workflows/ci.yml:
- `actions/checkout@v4` (line 14)
- `actions/setup-node@v4` (line 18)

Failing references in .github/workflows/release.yml:
- `actions/checkout@v4` (line 28)
- `actions/setup-node@v4` (line 31)
- `HardNorth/github-version-generate@v1` (line 40)
- `mindsers/changelog-reader-action@v1.3.1` (line 103)
- `actions/create-release@v1` (line 109)
- `actions/checkout@v3` (line 119)

Locations:

- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:18`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:40`
- `.github/workflows/release.yml:103`
- `.github/workflows/release.yml:109`
- `.github/workflows/release.yml:119`

### permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` key, and no job within either file defines a `permissions:` key. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating least-privilege principles.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in release.yml directly interpolate `${{ env.* }}` and `${{ steps.*.outputs.* }}` expressions into shell command strings. Any `${{ ... }}` expression inside a `run:` block is subject to YAML template substitution before the shell ever sees the value, allowing shell metacharacters to be injected. Affected steps and representative offending lines:

- 'Update build' step (sub-rule a): `git commit -m "Compiled version ${{ env.RELEASE_VERSION }}"`
- 'Update README.md' step (sub-rule a): `sed 's/${{ env.README_VERSION_PLACEHOLDER }}/${{ env.RELEASE_VERSION }}/g' ${{ env.README_TEMPLATE_FILE }} > ${{ env.README_FILE }}`; `git add ${{ env.README_FILE }}`
- 'Update CHANGELOG.md' step (sub-rule a): `${{ env.CHANGE_LOG_FILE }}`, `${{ env.TMP_SUFFIX }}`, `${{ env.RELEASE_VERSION }}` used throughout shell commands
- 'Tagging new version' step (sub-rule a): `RELEASE_TAG="v${{ env.RELEASE_VERSION }}"`, `${{ env.RELEASE_VERSION_MAJOR }}`, `${{ env.RELEASE_VERSION_MINOR }}`
- 'Update version file' step (sub-rule a): `echo '${{ env.CURRENT_VERSION }}'`, `${{ env.NEXT_VERSION }}`, `${{ env.VERSION_REPLACE_PATTERN }}`, `${{ env.VERSION_FILE }}`, `${{ env.ADDITIONAL_VERSION_FILE }}`, `${{ env.TMP_SUFFIX }}`

Locations:

- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:67`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:73`
- `.github/workflows/release.yml:74`
- `.github/workflows/release.yml:75`
- `.github/workflows/release.yml:76`
- `.github/workflows/release.yml:77`
- `.github/workflows/release.yml:78`
- `.github/workflows/release.yml:85`
- `.github/workflows/release.yml:86`
- `.github/workflows/release.yml:87`
- `.github/workflows/release.yml:93`
- `.github/workflows/release.yml:118`
- `.github/workflows/release.yml:119`
- `.github/workflows/release.yml:120`
- `.github/workflows/release.yml:121`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings across ci.yml and release.yml:

1. unpinned-uses: Pinned all 8 action references to full SHA digests:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - HardNorth/github-version-generate@v1 → @daf138e2557c0bb49affe49bd7d79671dc8bfb3f
   - mindsers/changelog-reader-action@v1.3.1 → @6624a194a4cad18ad1016631b792753745b0afb5
   - actions/create-release@v1 → @0cb9c9b65d5d1901c1f53e5e66eaf4afd303e70e
   - actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26

2. permissions: Added top-level permissions blocks to both workflows:
   - ci.yml: `contents: read` (only needs to read repo)
   - release.yml: `contents: write` (needs to push commits, tags, and create releases)

3. script-injection: Moved all ${{ env.* }} expressions out of run: shell strings into step-level env: blocks for 5 affected steps in release.yml: 'Update build', 'Update README.md', 'Update CHANGELOG.md', 'Tagging new version', and 'Update version file'. Shell scripts now reference plain environment variables ($VAR_NAME) instead of template expressions.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Setup git credentials' step of .github/workflows/release.yml. Moved `${{ secrets.GITHUB_TOKEN }}` out of the `run:` shell command and into an `env:` block as `GH_TOKEN`. The shell script now references it as `"$GH_TOKEN"` instead of directly interpolating the expression, preventing template substitution from bypassing shell quoting.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection instances in release.yml: (1) readmeUpdate: sed pattern now uses properly double-quoted variable contexts via quote-breaking technique. (2) changelogUpdate: replaced echo with printf and properly double-quoted RELEASE_VERSION as a separate argument. (3) versionFileUpdate: replaced backtick substitution with $() and properly double-quoted VERSION_REPLACE_PATTERN in its own context.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in .github/workflows/release.yml:

1. 'Update README.md' step (line 88): Replaced the unsafe shell quoting concatenation `sed 's/'"${README_VERSION_PLACEHOLDER}"'/'"${RELEASE_VERSION}"'/g'` with a safe approach that first escapes sed special characters in both values using `printf '%s\n' "$VAR" | sed 's/[[\.*^$()+?{|]/\\&/g'` (and additionally escaping `&` in the replacement value), then uses the sanitized variables in a double-quoted sed expression.

2. 'Update version file' step (line 160): Replaced the unsafe shell quoting concatenation with `${VERSION_REPLACE_PATTERN}` unquoted in sed by first escaping the pattern into `SAFE_REPLACE_PATTERN`, using it safely to derive `CURRENT_VERSION_VALUE` and `NEXT_VERSION_VALUE`, then escaping those derived values into `SAFE_CURRENT` and `SAFE_NEXT` before using them in the final sed substitution commands. This prevents any shell metacharacters in env-context-sourced values from being interpreted as shell commands.

