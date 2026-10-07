# GitHub Workflows Playbook

Reusable GitHub Actions workflows for iOS App Store management.
Each app repo has these (or should have these).

## Core Workflows

### `ios-testflight.yml` — Build & Upload to TestFlight
**What**: Archives the iOS app, uploads to App Store Connect TestFlight.
**When**: After merging features, before App Store submission.
**Inputs**:
- `marketing_version` (e.g. "1.0") — the version string
- Build number auto-increments from the last TestFlight build
**How**:
```bash
gh-api POST "/repos/{owner}/{repo}/actions/workflows/{id}/dispatches" \
  --data '{"ref":"main"}'
```
**Output**: New build in TestFlight, ready for internal/external testing.

---

### `asc-submit-app-store.yml` — Submit to App Store Review
**What**: Syncs metadata + screenshots from repo, attaches build, submits for review.
**When**: When ready for Apple review (after TestFlight validation).
**Inputs**:
- `build_number` (e.g. "18") — must be a VALID uploaded build
- `version_string` (e.g. "1.0") — the marketing version
- `sync_listing` (default true) — sync metadata/screenshots from repo
**How**:
```bash
gh-api POST "/repos/{owner}/{repo}/actions/workflows/{id}/dispatches" \
  --data '{"ref":"main","inputs":{"build_number":"18","version_string":"1.0"}}'
```
**Output**: App version in WAITING_FOR_REVIEW.

---

### `asc-status.yml` — Check App Store Status (read-only)
**What**: Queries App Store Connect for current version/build/submission state.
**When**: Hourly (via cron) or manually to check review status.
**Inputs**: None (workflow_dispatch only)
**Output**: Log lines with `VERSION`, `BUILD`, `SUBMISSION` states.

---

### `asc-music-reader-upload.yml` — Sync Listing (no submit)
**What**: Uploads metadata, screenshots, URLs to ASC without submitting.
**When**: When updating the product page without a new version.
**Inputs**: Same as submit workflow, but `VERIFY_ONLY` mode available.
**Note**: Each app has its own variant (e.g. `asc-music-reader-upload.yml` for OMR).

---

### `ios-screenshots.yml` — Capture Real Screenshots
**What**: Builds for simulator, captures actual app UI at App Store sizes.
**When**: When you need authentic screenshots (vs. generated marketing frames).
**Output**: PNGs at 1320x2868 (iPhone) and 2064x2752 (iPad).

---

### `asc-cancel-review.yml` — Pull from Review
**What**: Cancels the current App Store review submission.
**When**: When you need to fix something and resubmit (e.g. wrong build attached).
**Warning**: This pulls the app OUT of the review queue. Only use when necessary.

---

### `asc-clear-export-compliance.yml` — Clear Export Compliance
**What**: Sets the export compliance declaration (no encryption).
**When**: Run once per version if Apple asks about encryption.
**Note**: Most apps using only Apple APIs can declare "no encryption".

---

## Workflow IDs by App

| App | Repo | TestFlight | Submit | Status |
|-----|------|-----------|--------|--------|
| ProTune | photo-recipes | 364312490 | 364312491 | 364312490 |
| OMR | omr-sheet-cam | 367879548 | 368107917 | 367879547 |
| AI Music Radar | earsheet | 373628785 | TBD | 373628785 |
| FinalCap | finalcut | TBD | 364785276 | 364785276 |

> **Note**: IDs are workflow database IDs, not the same as run IDs.
> Get them via: `gh-api GET "/repos/{owner}/{repo}/actions/workflows"`

## Typical Flow

```
1. Merge feature to main
   ↓
2. Dispatch ios-testflight.yml → Build appears in TestFlight (30-60 min)
   ↓
3. Test on device via TestFlight
   ↓
4. Dispatch asc-submit-app-store.yml with build_number → WAITING_FOR_REVIEW
   ↓
5. Hourly asc-status.yml checks (via cron) → notify on decision
```

## Excluded Workflows

The following are NOT part of this playbook (app-specific or deprecated):
- `remove-testflight-users.yml` — one-off cleanup, not reusable
- `coreml-diag.yml` — OMR-specific diagnostics
- Per-app listing upload variants (consolidate to the standard pattern)
