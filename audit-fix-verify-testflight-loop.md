# Audit → Fix → Verify → TestFlight Loop

Reusable loop for taking an iOS app from "feature complete" to a verified TestFlight build
before App Store submission. Distilled from the first full run on
`yishengjiang99/omr-sheet-cam` (AI Camera – Music Reader, Oct 17 2026).

**Result of that run:** ~6 h elapsed · 26 flows / 211 steps audited · 33 issues filed
(27 bugs + 6 enhancements) · 33 PRs opened, 32 merged (1 duplicate closed) · 33/33 issues closed and verified · 0 reopened ·
main CI green (209 tests) · TestFlight build uploaded.

Templates live in [`audit-loop-templates/`](audit-loop-templates/). Kickoff prompt:
[`audit-loop-templates/kickoff-prompt.md`](audit-loop-templates/kickoff-prompt.md).

## When to use

- App has a paywall/IAP, background audio, or any review-sensitive surface.
- You have parallel agents (tracer, product reviewer, fixers, verifier) and a macOS CI queue.
- Not for: screenshot/metadata-only updates (see `PLAYBOOK.md`).

## Roles

| Role | Job |
|---|---|
| Chief of Staff (CoS) | Runs the loop, makes decisions, merges, keeps the traceability table. Never blocks on the human. |
| Auditor / tracer | Writes the flow audit, then traces code → bug issues. |
| Product reviewer | Separate pass: design/ASO/funnel/trust. Internal tracker only. |
| Fixers (streams A–D…) | One stream per file-overlap cluster; one PR per root cause. |
| Verifier / product owner | Re-walks each step per PR, posts verdicts. |
| Status routine | Every 30 min, posts GitHub-state-based status; nudges stalled fixers after 1 h. |

## Phase 0 — Prep (before any agent starts)

- [ ] **Get explicit user approval in the user's own chat** for public/irreversible bot actions
      (filing issues, merging PRs). Bots cannot rely on pre-written approval; give a
      pre-authorization ("merge reviewed PRs, squash, no per-PR CI, one main run at the end").
- [ ] Confirm `main` protection. OMR's `main` was **unprotected** → state "do not merge on red" in
      every agent prompt.
- [ ] Make the simulator CI workflow **`workflow_dispatch`-only** now, with `concurrency`
      (see Pitfalls → CI queue).
- [ ] **One prep PR** that adds every planned test class to the `ios-sim.yml` `-only-testing`
      allowlist (classes absent from main did not fail the run — verified) so fixers never
      touch that file.
- [ ] Pre-assign stream ownership and merge order (Phase 4).
- [ ] Record the decision list and default answers for product questions (Phase 3).

## Phase 1 — User-flow audit (IDs per step)

Output: `flows.md` ([template](audit-loop-templates/flows-template.md)).

1. Static read of every screen/service, plus Info.plist, entitlements, privacy manifest, tests, CI.
   Tag unverifiable claims `(inferred)` / `[RUNTIME]` — a Linux box can't run the app.
2. Number everything: **F#** core flows, **P#** purchase/IAP flows, steps `F3.4`, `P7.4`.
   Per step: *Expected*, *Acceptance* (testable), *Risk/evidence* with `path:line`.
3. Include a **mermaid overview** (also render to PNG), persisted-state table, permissions list.
4. Add a **StoreKit test matrix** (see App Store specifics) and "what could not be verified".
5. IDs are stable forever — bug titles, PRs, and the traceability table all key on them.

## Phase 2 — Code-trace → bug issues

- One issue per **root cause**, from the [bug template](audit-loop-templates/bug-template.md):
  title `[F3.4] user-visible symptom`, step ID, file:line, repro, severity, acceptance test.
- Severity: **P0** data loss/crash/charged-no-Pro/review blocker · **P1** core flow or revenue ·
  **P2** misleading with workaround · **P3** polish/docs. Enhancements are `ENH`, not bugs.
- Draft locally (`issues/<StepID>-slug.md`), keep an `issue-map.md` (step → issue #), file in
  batches P1 → P2 → P3 → ENH with a script that skips duplicates and scans for secrets/paths.
- Labels: `audit`, `bug|enhancement`, `flow:Fx`, `sev:Px`, `area:*`, `priority:*`.
- Filing issues is a public action → needs the Phase 0 approval.

## Phase 3 — Product / design review (separate pass)

- Output: `product-review.md` + `product-tracker.md` — **internal, not filed on GitHub**.
- Separate from bugs. Constraint line up front ("does not change core recognition/playback").
- Tag evidence: `[CODE path]`, `[S#]` external source, `[INFERENCE]`, `[UNVERIFIED]`.
- Tracker table: item · effort · impact · status; overlaps with audit IDs are cross-referenced.
- Promote only the items that are real work to ENH issues once decided.
- **Decide fast.** Open decisions go to the CoS with defaults; user can revisit. OMR examples:
  no feature flags (ship always-on), analytics anonymous + opt-out toggle, upsell caps
  (frequency cap, suppress for Pro / in-flight purchase / first session), store name keeps brand
  and avoids incumbents' trademarks.

## Phase 4 — Fixers: streams, merge order

- Split by **file overlap**, not by severity. OMR: A = StoreKit/entitlements, B = quota/scan
  gate, C = library/misc, D = upgrade-prompt stack (stacked), plus a CI prep PR.
- Each fixer: branch `fix/<stepid>-slug`, PR title `[F5.5] ...`, **regression test** added to
  the pbxproj in the correct group, body links the issue.
- **Merge order:** CI prep PR → shared-infra stream (A) → dependent streams (B…) → stacked
  stack bottom-up. Never merge a stream before the one it semantically collides with.
- **Closing keywords:** `Fixes #N` only when the PR *fully* fixes it. Partial → `Refs #N`
  (merging a "Fixes" PR auto-closes the issue even if only part landed).
- Squash-merge with `--match-head-commit <sha>` (guards against late pushes).
- Rebase promptly, one PR at a time in merge order; CoS posts a rebase request (once, not per pass).
- Stalled > 1 h → nudge the fixer in the PR.

## Phase 5 — Verify

Per PR, the verifier re-walks the affected step IDs, reads the diff, and posts a verdict comment
([format](audit-loop-templates/verdict-and-traceability.md)): PASS / FAIL / PARTIAL with evidence.

- Check: real regression test exists; scope matches issue; no secrets/docs/agents-file noise;
  body has the right closing keyword; pbxproj file refs are in the right group.
- Re-verify after rebases and after the final main run.
- Keep the **traceability table** (step · issue · severity · state · PR+SHA · verdict) current;
  a step is VERIFIED only after a **real main CI run** covers the merged SHA.
- A merged PR without a passing run on main is *unverified*; if the main run fails, reopen the
  matching issues.

## Phase 6 — CI gate → TestFlight

1. Land everything; stop cancelling/queueing runs. Cancel stale PR runs for merged branches.
2. **One manual `ios-sim.yml` run on main** (`gh workflow run ios-sim.yml --ref main`). Wait for a
   real `xcodebuild test` (15–35 min), not a 6-second status job.
3. Only then dispatch **`ios-testflight.yml`** on the same SHA. Build number = `run_number`;
   `MARKETING_VERSION` is hard-coded in the pbxproj (bump it for a new version).
4. The workflow must fail unless ASC reports the build **VALID** (no false greens).
5. **Never dispatch `asc-submit-app-store.yml` without explicit user approval.**

## Phase 7 — Post-release checklist

- [ ] TestFlight build VALID; assigned to internal testers; export-compliance prompt absent
- [ ] Traceability table final: all issues closed with verification comment, none reopened
- [ ] `TODO.md` / `docs/asc/TESTFLIGHT.md` history row added (build, SHA, run id, result)
- [ ] Review notes, App Privacy answers, privacy manifest, privacy page agree (see below)
- [ ] Privacy/support pages deployed and live-checked
- [ ] Status routine paused; leftover ENH/backlog items listed for the user
- [ ] Learnings appended to this playbook

## Pitfalls hit (and fixes)

| Pitfall | What happened | Fix |
|---|---|---|
| **Merge churn** | Every merge conflicted the others: `ios-sim.yml` allowlist, `project.pbxproj`, `TODO.md` | Prep PR pre-adds all planned test classes; add pbxproj entries to the correct group; avoid TODO.md edits in fix PRs (CoS updates it); rebase promptly in merge order |
| **macOS CI queue** | 3–4 runners, 15–35 min/run; per-PR runs piled up, authors' rebases re-queued, main runs got cancelled by the next merge | `workflow_dispatch`-only sim CI from the start; `concurrency` group per ref; one run on main at the end; cancel runs for merged/superseded branches |
| **Unprotected `main`** | Nothing stops a red merge | Tell every agent "never merge red"; or add branch protection |
| **Stacked PRs + `--delete-branch`** | Deleting the base branch auto-closed dependents (#56); retarget/reopen required | Never delete stacked base branches; retarget dependents to main first; don't use `--delete-branch` on stack bases |
| **`Fixes` on partial PRs** | Auto-closing an issue the PR only partly fixed (#62/#25) | `Refs #N` unless fully fixed; CoS closes after verification |
| **PR bodies "tmp"** | Stacked PRs had placeholder bodies → no closing link | Require real body with issue links on open |
| **pbxproj build break** | New files referenced in the wrong group (`Services` instead of `Paywall`) → "Build input files cannot be found" | Script-verify that each PBXFileReference's group path matches its disk path; reviewer greps the diff |
| **Bots can't approve public actions** | Issue filing/merge blocked on user approval | Approval up front in the user's chat (Phase 0) |
| **Stalls** | Fixers idle after rebase requests | 30-min GitHub-state status routine; nudge after 1 h |
| **Hollow green** | After `ios-sim` went manual-only, a "success" main run was a 6-second status job | Verdict requires the real test run's SHA + test count |
| **Out-of-order merge** | A pass script merged Stream B before A | Encode merge order in the pass script; verify dependencies in PR comments |

## App Store specifics (IAP / review / privacy)

**StoreKit 2** — see [`storekit-test-matrix.md`](audit-loop-templates/storekit-test-matrix.md) (24 cases).
- Attach the `.storekit` config **to the scheme** and the project (else products don't load in Xcode runs).
- Derive Pro from `Transaction.currentEntitlements` (not a cached bool); refresh on foreground and
  before the quota gate; one shared `StoreKitManager`.
- Restore reachable from Settings, not only the paywall; progress + result feedback.
- Ask to Buy / pending ≠ error; UI reflects approval.
- Free-scan quota is charged **only on success**; photo that hit the paywall is kept and resumed.
- Upgrade-prompt policy: shared gate, frequency cap, suppress for Pro / in-flight / first session.
- No raw Swift error strings to users.

**App Review**
- Review notes need a **reachable no-login path** (e.g. "Try sample" button that works on every
  screen the notes mention) and **sandbox purchase + Restore instructions**.
- `UIBackgroundModes: audio` needs a written justification in review notes (what plays, when,
  that no mic is used, how to test).
- Pro-gated feature copy (e.g. Transpose-as-Pro) must match code gating (`ProFeatureGate`) in the
  paywall, ASC description, and review notes.
- Subscription apps: Terms of Use + Privacy links in description (see `learnings-four-apps.md`).

**Privacy** — these four must agree: privacy page, ASC App Privacy answers, `PrivacyInfo.xcprivacy`,
and what the code sends. Decide **Linked vs Not Linked** once (an install UUID tied to purchase
history is conservatively *Linked*). ASC App Privacy must be published by hand in the web UI.

**Hosting** — privacy/support pages for music-reader are deployed by copying into
`yishengjiang99/finalcut` at `public/music-reader/` and pushing to `main`
(the `deploy-grepawk` workflow auto-deploys). Check the live URL afterwards.

**Release hygiene** — TestFlight build number = workflow `run_number`; `MARKETING_VERSION` hard-coded;
never submit for review from CI without explicit approval; no ASC changes from the audit itself.

## Timing expectations (OMR run, all times PT, Oct 17 2026)

| Milestone | Clock time | Notes |
|---|---|---|
| Flow audit done (`flows.md`) | ~09:40 | static read only |
| Product review done | ~10:00 | ran in parallel with fixer start |
| Fixer PRs begin opening | 10:03 | fixers worked from local issue drafts |
| 33 issues filed | 10:22–10:23 | batch script, after user approval |
| Fix PRs all opened (#6–#19, #53–#62; Stream D #64–#69 at ~12:20) | 10:03–10:30 | 4 streams in parallel |
| First merge | 10:28 | |
| Bulk of merges | 12:12–14:12 | after user pre-authorized batch merge; before that ~1 merge/30 min |
| Main CI gate (real run) | ~13:15–14:30 | 209 tests, 0 failures; 18–20 min when not queued |
| TestFlight dispatch → VALID | 14:31–14:38 | ~7 min |
| **Total** | **~6 h** | 33 issues, 33 PRs opened (32 merged, 1 closed duplicate #57) |

Merge/rebase churn (10:30–12:10: only 4 merges in 100 min) was the long pole. Pre-authorized batch
merge + manual-only CI unblocked it; doing that at minute 0 should save ~2 h.

## Second-round notes (Music Radar, `yishengjiang99/earsheet`)

- Reuse this doc + templates; copy the kickoff prompt and fill the app-specific block.
- Reuse the flow ID scheme but expect different F-flows (recording/mic permission instead of camera
  → check `NSMicrophoneUsageDescription`, interruptions, background audio).
- Do Phase 0 first (approval, manual-only CI, allowlist prep PR) — it is the biggest time saver.
