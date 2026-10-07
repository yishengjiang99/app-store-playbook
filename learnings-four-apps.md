# App Store Learnings — Four Apps (Oct 2026)

Real-world lessons from submitting four iOS apps through App Store review.
These are hard-won; don't relearn them.

## Apps covered

| App | Bundle ID | ASC ID | Status (2026-10-07) |
|---|---|---|---|
| AI Music Radar | com.ragnus.pnge | 6818838017 | v1.0 b19 WAITING_FOR_REVIEW (day 3) |
| ProTune AI Camera | com.ragnus.mvp | 6813991381 | v1.1 READY_FOR_SALE; v1.2 b56 WAITING_FOR_REVIEW (resubmitted after rejection) |
| FinalCap | com.ragnus.finalcap | 6815060815 | v1.0 b12 WAITING_FOR_REVIEW (day 7) |
| AI Camera - Music Reader | com.ragnus.vp | 6816476323 | v1.0 b25 WAITING_FOR_REVIEW (resubmitted after 11-day stall) |

---

## 1. ATT (App Tracking Transparency) — Guideline 2.1 rejection

**What happened:** ProTune v1.2 build 53 was rejected. Apple couldn't find the
ATT permission prompt on iPadOS 27.0.

**Root causes found in code:**

1. **ATT prompt was gated behind onboarding completion.** The reviewer does a
   fresh install and may never finish onboarding → prompt never appears.
   Fix: show ATT on first launch, not after onboarding.

2. **Meta's `activateApp()` was called BEFORE the ATT prompt.** This sends data
   to Meta before the user has consented. Apple requires the prompt *before*
   any tracking data is collected.
   Fix: delay `activateApp()` until ATT status is determined. On first launch
   (status `.notDetermined`), request ATT first, then fire the activation event
   in the completion handler.

3. **AppDelegate `applicationDidBecomeActive` may not fire reliably on iOS 27.**
   Fix: add a backup trigger in SwiftUI's `scenePhase` handler:
   ```swift
   .onChange(of: scenePhase) { _, phase in
       if phase == .active {
           MetaEvents.requestTrackingIfNeeded()
       }
   }
   ```

**ASC privacy disclosure:** If the Meta SDK ships, App Privacy must declare
Tracking = Yes with Device ID. The App Store Connect API does NOT expose this —
it must be done manually in the ASC UI. Declaring "not tracking" while the
Meta SDK is in the binary is a separate rejection.

**Resubmission requires a screen recording** attached in App Review Information
→ Notes, showing: fresh install → ATT prompt → user flow. Apple explicitly
requested this.

**Code pattern that works:**
```swift
static func activateApp() {
    guard isConfigured else { return }
    guard ATTrackingManager.trackingAuthorizationStatus != .notDetermined else {
        requestTrackingIfNeeded()  // fires activateApp in completion handler
        return
    }
    AppEvents.shared.activateApp()
}

static func requestTrackingIfNeeded() {
    guard isConfigured else { return }
    guard ATTrackingManager.trackingAuthorizationStatus == .notDetermined else {
        syncAdvertiserTrackingFlag()
        return
    }
    // NOT gated behind onboarding — Apple tests fresh installs
    ATTrackingManager.requestTrackingAuthorization { _ in
        Task { @MainActor in
            syncAdvertiserTrackingFlag()
            AppEvents.shared.activateApp()
        }
    }
}
```

**Testing:** The ATT prompt only appears once per install. To test: delete the
app completely, reinstall from TestFlight, launch. Also check iOS Settings →
Privacy & Security → Tracking → "Allow Apps to Request to Track" is ON —
if off, the prompt never appears for any app.

---

## 2. Can't edit listing while in review

**What happened:** AI Camera - Music Reader v1.0 sat in WAITING_FOR_REVIEW for
11 days with no screenshots uploaded. The listing sync workflow failed with:
`version 1.0 not editable (WAITING_FOR_REVIEW); nothing written`.

**Rule:** Apple locks metadata and screenshots while a version is in review.
You cannot upload screenshots, change description, or edit anything.

**Fix:** Cancel the review → upload screenshots/metadata → resubmit. This resets
the review clock, but a stalled review with missing assets wasn't going anywhere
anyway.

**Prevention:** Always verify screenshots are in ASC *before* submitting. The
upload workflow should run as part of the pre-submit checklist.

---

## 3. TestFlight builds CAN be dispatched via API

Earlier assumption was wrong: TestFlight `workflow_dispatch` workflows CAN be
triggered via the GitHub API. The user corrected this.

```bash
gh-api POST "/repos/{owner}/{repo}/actions/workflows/{workflow_id}/dispatches" \
  --data '{"ref":"main","inputs":{"marketing_version":"1.2"}}'
```

The new run appears within seconds. Build takes 20-30 min, plus 15-30 min for
Apple's TestFlight processing before it appears on device.

**Gotcha:** If you dispatch immediately after a merge, verify the workflow
picked up the right commit (`head_sha`). Check run history before telling the
user to install.

---

## 4. Don't extract scripts from old commits in workflows

**What happened:** The photo-recipes `asc-submit-app-store.yml` workflow tried
to extract a Python submit script from a pinned old commit via regex:
```python
raw = subprocess.check_output(["git", "show", "f69eecf...:..."])
m = re.search(r"python3 <<'PY'\n(.*)\n          PY\s*$", raw, re.S)
```

This broke when the file format changed (the regex anchored to end-of-file but
the workflow continued after the PY block).

**Fix:** Save the script as a real file in the repo
(`scripts/asc/submit_app_store.py`) and have the workflow run it directly.
Never use regex extraction from git history in CI — it's fragile and breaks
silently.

---

## 5. Stuck submissions after rejection

**What happened:** After ProTune v1.2 build 53 was rejected, resubmitting
failed with:
```
STATE_ERROR.ITEM_PART_OF_ANOTHER_SUBMISSION
"appStoreVersions with id ... was already added to another reviewSubmission"
```

The rejected version was still attached to the old (rejected) submission.

**Fix:** Run the "cancel stuck submissions" workflow before resubmitting. This
clears the dead submission so the version can be attached to a new one.

**Note:** The submit may still report a workflow failure *after* `SUBMIT_OK`
appears in logs. Check for `SUBMIT_OK submission_id=... state=WAITING_FOR_REVIEW`
in the log — if present, the submission went through despite the exit code.

---

## 6. Meta SDK + Events Manager setup

**Install attribution** works via Apple's SKAdNetwork, not the Meta SDK. The
live App Store build (without SDK) still attributes installs. The Meta SDK is
only needed for in-app events (trials, purchases) to switch campaign
optimization from installs to subscribers.

**Events Manager:** The Facebook App must be connected as a dataset in Events
Manager, linked to the ad account. If `ads_get_datasets` returns empty, no
events can flow — it's a setup issue, not a code issue. This is manual in Meta
Business Settings → Data Sources.

**New ad accounts get throttled:** Meta imposes a $20/day account-level spend
cap on new advertisers, regardless of campaign budget. It lifts gradually with
clean spend history. A full disable → restricted → restored cycle is common for
new accounts.

---

## 7. Review watch operations

Hourly status checks via `workflow_dispatch` on each repo's read-only ASC status
workflow. Key implementation notes:

- GitHub Actions log download returns 302 — follow WITHOUT the Authorization
  header or it 401s.
- Log lines are timestamp-prefixed (`2026-10-07T08:15:59Z VERSION ...`), so
  `grep ^VERSION` never matches. Match ` VERSION ` instead.
- `gh-api` correct syntax: `--data '{"ref":"main"}'` with `POST` as positional.
  The `-X -d` flags don't work with this wrapper.
- Track state in `state.json`; silent when unchanged, notify on decision.
- Never release, appeal, or resubmit from the watch — read-only.

---

## 8. OMR-specific: telemetry + IAP before submission

If a build contains telemetry or IAP:
- Update App Store privacy answers, privacy policy, and description BEFORE
  submitting. "Collects no data" + telemetry = rejection.
- IAP product IDs in code must match products created in ASC.
- Don't market unimplemented features ("priority processing").
- The Xcode project file (`project.pbxproj`) must include all Swift files in
  the target — manual edits break it; use Xcode's "Add Files" UI.

---

## Quick reference: resubmit checklist

- [ ] Screenshots uploaded and verified in ASC *before* submitting
- [ ] ATT prompt appears on fresh install (if Meta SDK or any tracking)
- [ ] ASC privacy disclosure matches reality (tracking Yes/No, data types)
- [ ] Screen recording attached in review notes (if Apple requested it)
- [ ] Stuck/rejected submissions cancelled
- [ ] Build number > previous submitted build
- [ ] IAP products created in ASC (if applicable)
- [ ] Privacy policy URL live and accurate
