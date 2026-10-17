# Bug report template (code tracer → GitHub issue)

One issue per **root cause**. Use the step ID from `flows.md`. Filing issues is a public action:
get the user's explicit approval first (draft locally, file in batches).

## Title
`[<StepID>] <short, user-visible symptom>` — e.g. `[P3.7] Photo that hit the paywall is discarded after purchase`

Labels: `audit`, `bug`, flow (`flow:F3`, `flow:P6`), severity (`sev:P0`…`sev:P3`),
`area:{capture|scan|player|library|settings|iap|paywall|quota|telemetry|audio|docs}`, `priority:{high|medium|low}`.
Enhancements: `audit`, `enhancement`, flow, area, `priority:*` (no `sev:*`; title prefix `[<StepID>] Enhancement (product decision): …`).

## Body

```markdown
**Step ID:** F3.4            <!-- from flows.md; "also P1.4" if it spans flows -->
**Flow:** F3 <flow name>
**Static verdict from audit:** GAP | ? | confirmed-by-run
**Build / env:** version (build) · iOS · device · Simulator|Device · StoreKit env (Xcode config | Sandbox | Prod) · Apple ID type

### Expected
<what flows.md "Expected"/"Acceptance" says>

### Actual
<what the code/device does; quote log lines when relevant>

### Location
`Sources/App/Path/File.swift:LINE` (+ secondary refs)
Root cause (1–2 lines): <why>

### Repro
1. Preconditions (fresh install? `defaults write <bundle> <key> …`? airplane mode? StoreKit config attached?)
2. Steps…
3. Observed result
Repro rate: always | intermittent (n/m) | needs device only
Evidence: screenshot / recording / diagnostics / telemetry events

### Severity
P0 | P1 | P2 | P3 — one-line justification

### Suggested fix
<minimal change, named functions; alternatives; product decision needed?>

### Acceptance test
- [ ] Manual: <exact steps + expected observable result>
- [ ] Automated: <XCTest / SKTestSession (case Sx) / XCUITest name + assertion>
- [ ] Regression: <neighbouring step IDs re-verified>
```

## Severity
- **P0** data loss, crash/hang on core path, charged but Pro not granted, App Review blocker, privacy statement violated
- **P1** core flow broken or revenue-affecting (wrong lock/unlock, restore unreachable), frequent confusing failure
- **P2** wrong/misleading behavior with a workaround, edge-case failures, analytics gaps
- **P3** polish, copy, minor a11y, docs drift

## Worked example (from the OMR run)
**`[P7.4] Pro stays unlocked after subscription expires until the app is relaunched`** — P1
- Expected: after `expirationDate`, next scan shows the paywall without relaunch.
- Actual: Pro flag recomputed only in manager `init` and on restore; `Transaction.updates` has no expiry event.
- Repro: Xcode StoreKit config → buy monthly → Manage Transactions → Expire → background/foreground → 6th scan proceeds.
- Fix: refresh entitlements on `scenePhase == .active` and before the quota gate; derive Pro from `Transaction.currentEntitlements`.
- Acceptance: `SKTestSession` test: buy → `expireSubscription` → refresh → quota check blocks.

## PR conventions for the fixer
- Title `[<StepID>] <summary>`; branch `fix/<stepid>-slug`; body links the issue.
- `Fixes #N` only if the PR fully fixes it, else `Refs #N`.
- Add a regression test; register it in the pbxproj **in the correct group**; do not edit the CI allowlist or TODO.md.
