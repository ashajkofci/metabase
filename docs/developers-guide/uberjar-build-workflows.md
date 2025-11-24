# Uberjar Build Workflows

This document describes the GitHub Actions workflows available for building Metabase uberjars.

## Overview

Metabase provides multiple workflows to build uberjars for different purposes:

1. **Automatic Builds** (`uberjar.yml`) - Builds on master and specific branches
2. **Manual Builds** (`build-uberjar-manual.yml`) - On-demand builds for any commit/branch
3. **Release Builds** (`build-uberjar-release-types.yml`) - Builds for tags and releases
4. **Official Releases** (`build-for-release.yml`) - Official release artifacts

## Workflows

### 1. Automatic Uberjar Builds (`uberjar.yml`)

**Purpose:** Automatically builds uberjars for specific branches when changes are pushed.

**Triggers:**
- Push to `master` branch
- Push to `internal-tools` branch
- Manual workflow dispatch

**Editions Built:** Both OSS and EE

**Artifacts:**
- `metabase-oss-{commit-sha}-uberjar`
- `metabase-ee-{commit-sha}-uberjar`

**Usage:**
- Automatic - no action required
- For manual trigger: Go to Actions → Build Uberjar → Run workflow

---

### 2. Manual Uberjar Builds (`build-uberjar-manual.yml`)

**Purpose:** Build uberjars on-demand for development, testing, or custom purposes.

**Triggers:** Manual workflow dispatch only

**Features:**
- Build from any branch, tag, or commit
- Choose OSS, EE, or both editions
- Select build type (development, testing, release-candidate, custom)
- Custom version strings
- Configurable Java and Node.js versions
- Optional health checks

**Parameters:**

| Parameter | Description | Options | Default |
|-----------|-------------|---------|---------|
| `ref` | Git ref to build from | Any branch/tag/commit | `master` |
| `edition` | Edition to build | `oss`, `ee`, `both` | `both` |
| `build-type` | Type of build | `development`, `testing`, `release-candidate`, `custom` | `development` |
| `custom-version` | Custom version string | Any string | (optional) |
| `java-version` | Java version | `11`, `17`, `21` | `21` |
| `node-version` | Node.js version | `18`, `20`, `22` | `22` |
| `skip-tests` | Skip health checks | `true`, `false` | `false` |

**Artifacts:**
- `metabase-{edition}-{build-type}-{commit-sha}-uberjar`
- `metabase-{edition}-{build-type}-{commit-sha}-metadata`

**Usage:**

1. Go to: Actions → Build Uberjar - Manual → Run workflow
2. Fill in parameters:
   - **ref:** `feature-branch` (or any commit SHA)
   - **edition:** `both` (builds OSS and EE)
   - **build-type:** `development`
   - Click "Run workflow"

**Example Scenarios:**

```yaml
# Development build from a feature branch
ref: feature/new-feature
edition: both
build-type: development

# Testing build from a specific commit
ref: abc123def456
edition: ee
build-type: testing

# Custom versioned build
ref: master
edition: oss
build-type: custom
custom-version: v1.50.0-experimental-1
```

---

### 3. Release Type Builds (`build-uberjar-release-types.yml`)

**Purpose:** Build official release artifacts for different release types.

**Triggers:**
- Push tags matching: `v*.*.*`, `v*.*.*-rc*`, `v*.*.*-beta*`, `v*.*.*-alpha*`, `v*.*.*-hotfix*`
- Manual workflow dispatch

**Features:**
- Automatic release type detection from tags
- Creates GitHub releases (draft)
- Generates checksums (SHA256, MD5)
- Comprehensive verification
- Supports pre-releases and stable releases

**Release Types:**

| Type | Tag Pattern | Pre-release | Example |
|------|-------------|-------------|---------|
| Stable | `v*.*.*` | No | `v1.50.0` |
| Release Candidate | `v*.*.*-rc*` | Yes | `v1.50.0-rc1` |
| Beta | `v*.*.*-beta*` | Yes | `v1.50.0-beta1` |
| Alpha | `v*.*.*-alpha*` | Yes | `v1.50.0-alpha1` |
| Hotfix | `v*.*.*-hotfix*` | No | `v1.49.1-hotfix1` |
| Patch | Manual only | No | `v1.49.2` |

**Artifacts:**
- `metabase-oss-{version}-release.jar`
- `metabase-ee-{version}-release.jar`
- `SHA256.sum` and `MD5.sum` for each
- Build metadata

**Usage:**

**Option 1: Tag Push (Automatic)**
```bash
# Create and push a release tag
git tag -a v1.50.0-rc1 -m "Release Candidate 1"
git push origin v1.50.0-rc1
```

**Option 2: Manual Dispatch**
1. Go to: Actions → Build Uberjar - Release Types → Run workflow
2. Parameters:
   - **version:** `v1.50.0-rc1`
   - **release-type:** `release-candidate`
   - **create-tag:** `true` (optional, creates tag if it doesn't exist)
3. Click "Run workflow"

**GitHub Release:**
- Creates a draft GitHub release
- Attaches JAR files and checksums
- Marks as pre-release if applicable
- Ready for review and publishing

---

## Choosing the Right Workflow

| Scenario | Workflow | Why |
|----------|----------|-----|
| Testing a feature branch | Manual Builds | Flexible, choose any ref |
| Development iteration | Manual Builds | Custom versions, skip tests |
| CI/CD integration | Automatic Builds | Runs on push to master |
| Release candidate | Release Types | Proper versioning, checksums |
| Official release | Official Releases | Full release process |
| Hotfix release | Release Types | Tag-based, verified |
| Custom experiment | Manual Builds | Full control over parameters |

## Artifact Naming Conventions

### Automatic Builds
```
metabase-{edition}-{commit-sha}-uberjar
Example: metabase-oss-abc123def456-uberjar
```

### Manual Builds
```
metabase-{edition}-{build-type}-{commit-sha}-uberjar
Example: metabase-ee-testing-abc123def456-uberjar
```

### Release Builds
```
metabase-{edition}-{version}-release
Example: metabase-oss-v1.50.0-rc1-release
```

## Build Metadata

All builds include metadata files:

- `COMMIT-SHA` - Full commit hash
- `VERSION` - Version string
- `EDITION` - `oss` or `ee`
- `BUILD-TYPE` - Type of build
- `BUILD-TIMESTAMP` - UTC timestamp
- `RELEASE-TYPE` - For release builds
- `IS-PRERELEASE` - For release builds

## Health Checks

All workflows include optional health checks that:
1. Start the uberjar
2. Wait for the `/api/health` endpoint
3. Verify response is `{"status":"ok"}`
4. Timeout after 5 minutes

## Best Practices

### For Development
- Use **Manual Builds** with `skip-tests: true` for faster iteration
- Use descriptive `custom-version` strings to identify builds
- Build only the edition you need (`oss` or `ee`) to save time

### For Testing
- Always run health checks (`skip-tests: false`)
- Use `testing` build type to distinguish from development builds
- Keep artifacts for regression testing

### For Releases
- Use **Release Types** workflow
- Always create proper semantic version tags
- Let the workflow create checksums
- Review the draft GitHub release before publishing
- Verify both OSS and EE editions

### For CI/CD
- Use commit SHAs to ensure reproducible builds
- Store artifacts with sufficient metadata
- Implement proper artifact retention policies

## Troubleshooting

### Build Fails
- Check Java/Node versions match project requirements
- Verify the ref exists and is accessible
- Review build logs for compilation errors

### Health Check Fails
- Increase timeout if needed
- Check for port conflicts (default: 3000)
- Review application startup logs

### Artifact Not Found
- Ensure build completed successfully
- Check artifact retention period
- Verify artifact name matches expected pattern

## Examples

### Example 1: Quick Development Build
```
Workflow: Build Uberjar - Manual
ref: my-feature-branch
edition: oss
build-type: development
skip-tests: true
```

### Example 2: Pre-Release Testing
```
Workflow: Build Uberjar - Manual
ref: release-x.50
edition: both
build-type: release-candidate
custom-version: v1.50.0-rc1
skip-tests: false
```

### Example 3: Official RC Release
```
Workflow: Build Uberjar - Release Types
version: v1.50.0-rc1
release-type: release-candidate
create-tag: true
```

### Example 4: Hotfix Build
```
bash:
git checkout hotfix/critical-fix
git tag -a v1.49.3-hotfix1 -m "Critical security fix"
git push origin v1.49.3-hotfix1
```
This automatically triggers the Release Types workflow.

## Advanced Usage

### Building from Pull Requests
```
ref: refs/pull/12345/head
edition: both
build-type: testing
```

### Custom Java/Node Combinations
```
java-version: 17
node-version: 18
# For testing compatibility with older versions
```

### Parallel Builds
The workflows automatically build OSS and EE in parallel for faster completion.

## See Also

- [Official Release Process](./release-process.md)
- [Metabase Build System](../../bin/build/README.md)
- [Containerization Workflows](./containerization.md)
