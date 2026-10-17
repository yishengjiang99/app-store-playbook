# StoreKit 2 test matrix (S1–S24)

## Environments
| Env | How | Good for |
|---|---|---|
| **E1 Xcode StoreKit Testing** | `.storekit` attached to the scheme (Run → Options → StoreKit Configuration); Debug → StoreKit → Manage Transactions | Deterministic: Ask to Buy, fail/interrupt, expire, refund, grace, renewal rate, storefront |
| **E2 XCTest + `SKTestSession`** | `SKTestSession(contentsOf:)`; `resetToDefaultState()`, `clearTransactions()`; `disableDialogs = true` | CI regression of entitlement/quota logic (confirm API names in the Xcode SDK) |
| **E3 Sandbox via TestFlight** | Sandbox tester, Settings → App Store → Sandbox Account | Real ASC product metadata, receipts, review parity. Products take ~15 min after creation |

Seed quota in the simulator: `xcrun simctl spawn booted defaults write <bundle> <countKey> -int 5` and the date key.

## Cases (each lists the flow steps it validates; fill the IDs from your `flows.md`)
| # | Case | Env | Expected |
|---|---|---|---|
| S1 | Products load (US) | E1 | Plans shown, yearly first, correct prices |
| S2 | Load failure | E1/E2 | Single readable message + Retry |
| S3 | Missing/partial products | E1 | Remaining card only; none → empty message; no crash |
| S4 | Buy monthly | E1 | Pro on, paywall dismissed, scan continues, quota unchanged |
| S5 | Buy yearly | E1/E3 | Same; transaction id stored |
| S6 | User cancels sheet | E1 | No error, buttons re-enabled, no Pro |
| S7 | Fail transaction | E1/E2 | Plain-language error; retry succeeds after toggle off |
| S8 | Ask to Buy | E1/E2 | Neutral pending message; approve → Pro ≤5 s **and UI updates**; decline clears |
| S9 | Interrupted purchase | E1 | No stuck spinner, readable message |
| S10 | Restore matrix | E1/E2 | (a) active → Pro + confirmation (b) none → "nothing to restore" (c) offline → retryable (d) cancelled → silent |
| S11 | Verification failure | E1 | Never grant Pro from unverified |
| S12 | Expire while running | E1/E2 | Blocked right after foreground, not after relaunch |
| S13 | Renewal | E1/E2 | Pro never flips false; no telemetry spam |
| S14 | Refund (+ multi-product) | E1 | Single refund → off; refund monthly while yearly active → **still Pro** |
| S15 | Grace / billing retry | E1 | Pro retained in grace, lapses after |
| S16 | Reinstall | E1 | Auto-restored without a tap; Settings → Restore exists |
| S17 | Kill mid-purchase | E1 | Pro from unfinished transaction on relaunch |
| S18 | Offline Pro launch | E1/E3 | Never blocked |
| S19 | Storefront / currency | E1 | Localized `displayPrice`, no hard-coded "$" |
| S20 | Quota boundary & rollover | sim | 5 pass, 6th blocked; rollover resets; failed scan not charged |
| S21 | Sandbox account switch / 2nd device | E3 | Flag recomputed; no crash |
| S22 | Review path | E3 fresh install | Following review notes literally reaches the paywall |
| S23 | Time zone / clock | sim | Resets once per local day; document accepted bypass |
| S24 | Two-manager consistency | E1 | Single source of truth (one shared manager) |

## Automated tests to add
1. Quota with injected `UserDefaults` suite + clock: boundary, rollover, Pro bypass, remaining.
2. Entitlement manager with `SKTestSession`: S4, S6, S7, S8, S10, S12, S14.
3. Analytics filter: needed props survive (OMR bug: regex dropped `platform` because it matched `lat`), PII keys dropped.
4. XCUITest with seeded quota: Nth scan shows paywall (add accessibility identifiers: subscribe, restore, close).
5. Gating table test (`ProFeatureGate`): gated set matches paywall/ASC/review-notes copy.
6. Lint test for review notes (sample path mentioned, background-audio justification present).
