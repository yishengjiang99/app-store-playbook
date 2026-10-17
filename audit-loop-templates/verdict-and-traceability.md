# Verdict comments, traceability table, merge-log

## Verifier verdict comment (one per PR, re-post after rebase/fix)

```markdown
**Verification verdict (<role>): PASS | FAIL | PARTIAL — <one-line reason>**

Steps re-walked: F5.5, F5.3 (regression)
- Issue ↔ PR: #N ↔ #M, closing keyword: Fixes | Refs (correct? y/n)
- Diff scope: <files>; no secrets / generated docs / agent files: y/n
- Regression test: `<TestClass.testName>` present and meaningful: y/n
- pbxproj refs in correct group: y/n
- CI: run <id> on <head sha>: SUCCESS | not run | cancelled (a verdict without a real test run on the
  head SHA is at most "reviewed", not "verified")
- Acceptance check: <what was walked, observable result>
- Caveats / follow-ups: <partial fixes, deferred items>
```

On FAIL include the exact error text and the root-cause file/line (e.g. the pbxproj group id).

## Traceability table (`docs/audit/traceability.md`, updated by the CoS routine)

| Step | Issue | Sev | Issue state | PR(s) | Verdict |
|---|---|---|---|---|---|
| F2.7 | #24 | P1 | CLOSED | #12 (merged `0a5954e`) | VERIFIED (main run <id> on <sha> PASSED, <n> tests) |
| P1.10 | #25 | P2 | CLOSED | #60 + #62 + #65 | VERIFIED (all parts merged; closed by CoS after verification) |

Verdict vocabulary: `OPEN` · `IN REVIEW` · `FAIL (reason)` · `MERGED-UNVERIFIED` · `VERIFIED` · `PARTIAL (caveat)`.
Append dated notes under the table (what merged, what is blocked, what runs are pending).

## merge-log lines (`#N | outcome | notes`)

```
#13 | merged 8e59c6f | F2.9; Fixes #32 (verified CLOSED). Re-reviewed: regression test present, scope ok. Sim CI SUCCESS on head; MERGEABLE/CLEAN before merge (--match-head-commit).
#7  | waiting | Stream B; review OK, CI green, HELD until Stream A lands (semantic collision on ScanQuota).
#56 | waiting | STACKED on #6 branch; body 'tmp' -> no Fixes line. Do NOT delete base branch.
```

## Merge command
`gh pr merge <N> --squash --match-head-commit <sha>` — no `--delete-branch` for stack bases.
