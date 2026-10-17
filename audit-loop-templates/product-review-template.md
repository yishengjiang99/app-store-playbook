# Product & design review — <app> (internal; NOT filed on GitHub)

Repo @ sha · date · author. **Constraint:** does not change <core engine>. Companion: `flows.md` (cite bug IDs, don't repeat them).
Evidence tags: `[CODE path]` · `[S#]` external source (URL in appendix) · `[INFERENCE]` · `[UNVERIFIED]`. No invented benchmarks.

## Sections
1. Executive summary: what the app is today (evidence table), 3 strategic findings, top-5 recommendations, launch-risk callouts
2. Market & competitor read (pricing, free tier, paywall pattern; real reviews of the closest comparable)
3. First-run & activation · 4. Free→Pro funnel & paywall (trigger timing, redesign spec, pricing checklist, trial?)
5. Core experience polish · 6. Retention · 7. Trust (privacy story, analytics consent, accuracy expectations, subscription honesty)
8. Accessibility · 9. Listing/ASO/screenshots · 10. Metrics to instrument (snake_case events, funnels)
11. Parked ideas · 12. Prioritised backlog (P0 launch / P1 first 2–4 weeks / later) with a **Touches core?** column

## product-tracker.md
```markdown
## Recommendations
| # | Item | Effort | Impact | Status |
|---|------|--------|--------|--------|
| R1 | <item> | S/M/L | High/Med/Low | Overlaps audit P1.6 (Stream D) / Backlog / Done #PR |

## Launch risks
- <guideline # — risk>: status

## Open decisions for <user>   (each with a default)
## Decisions made by Chief of Staff (<date>; reversible, revisit anytime)
1. <decision + default + trigger to revisit>
```
Enhancements that become real work → file as `ENH` issues, tracked separately from bugs.
