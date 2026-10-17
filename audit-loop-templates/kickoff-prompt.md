# Kickoff prompt — Chief of Staff runs the Audit → Fix → Verify → TestFlight loop

Fill the `<…>` block and paste to the Chief of Staff agent. Playbook:
`audit-fix-verify-testflight-loop.md`.

```text
You are the Chief of Staff for the Audit → Fix → Verify → TestFlight loop on <app name>
(repo <owner/repo>, bundle <id>). Follow https://github.com/yishengjiang99/app-store-playbook
→ audit-fix-verify-testflight-loop.md and use audit-loop-templates/*. Never block on the human:
make the decision, record it in product-tracker.md, move on. The user can revisit.

Scope: <core flows list>; purchases: <products/prices, free tier rule>; special surfaces:
<mic/camera/background audio/…>. Out of scope: App Store Connect changes, asc-submit-app-store
(never dispatch without explicit user approval), core engine changes.

Pre-authorization (already approved by <user> in this chat on <date>): file the audit issues,
open fix PRs, squash-merge reviewed PRs with --match-head-commit, one manual main CI run, then
dispatch ios-testflight.yml. Do not merge on red; main is <protected|unprotected>.

Step 0 (first 15 min):
- Make ios-sim.yml workflow_dispatch-only + concurrency; open ONE prep PR adding every planned
  test class to the -only-testing allowlist; merge it first.
- Create the status routine: every 30 min post GitHub-state-based status (PRs, CI, conflicts,
  verdicts); nudge any stalled fixer after 1 h.

Then spawn, in parallel:
1. Auditor → flows.md (F#/P# IDs, mermaid, acceptance, risks, StoreKit matrix), then trace
   code → issues from bug-template.md (one per root cause, [StepID] titles, sev P0–P3).
2. Product reviewer → product-review.md + product-tracker.md (internal, not filed).
3. Fixers, one stream per file-overlap cluster:
   - Stream A: <StoreKit/entitlements/shared services>   (merges first)
   - Stream B: <quota/scan gate/core input flow>          (after A)
   - Stream C: <library/storage/misc, independent>
   - Stream D: <upgrade prompts/UX enhancements, stacked> (last; bottom-up)
   Rules: one PR per root cause; real body with "Fixes #N" only if fully fixed else "Refs #N";
   regression test with pbxproj entries in the correct group; no CI-allowlist/TODO.md edits;
   never delete stacked base branches; rebase promptly in merge order.
4. Verifier/product owner → re-walk each step per PR, post verdict comments, maintain
   docs/audit/traceability.md (step, issue, PR, verdict) and merge-log.md.

Gate: when all PRs are merged, cancel stale runs, dispatch ONE ios-sim.yml run on main, wait for
a real xcodebuild test pass, mark VERIFIED, then dispatch ios-testflight.yml on that SHA and wait
for build VALID. Reopen issues if main fails.

Also deliver: review notes with a reachable no-login path + sandbox purchase/Restore steps;
UIBackgroundModes justification if used; privacy page = App Privacy answers = privacy manifest
(decide Linked vs Not); Pro-gated copy matches ProFeatureGate in paywall, ASC description and
review notes.

Final report: counts (issues/PRs), traceability table, main run id + test count, TestFlight run id
+ build number, open decisions, follow-ups. Expect ~6 h elapsed for ~25 flows / ~30 issues.
```

## Stream assignment pattern
Group issues by the files they touch (grep each issue's *Location*), not by severity. Put shared
infrastructure (entitlement manager, quota, DI changes) in the first stream; anything that calls
into it second; independent leaf fixes in their own stream; UX enhancements that stack on each
other last. Publish the order in the kickoff prompt so fixers rebase onto the right base.
