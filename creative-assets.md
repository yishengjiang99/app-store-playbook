# Product Page Creative Assets

## Header and Search Results

App Store Connect → App → Product Page Information → Header and Search Results

These are creative assets that customize how your app appears in:
- **Header**: The top banner on your App Store product page
- **Search Results**: How your app appears in App Store search

### Specs

| Asset | Size | Format | Notes |
|-------|------|--------|-------|
| Header | 1320 x 2868 px | PNG/JPG | Portrait, shown at top of product page |
| Search Results | 1320 x 2868 px | PNG/JPG | Icon + first screenshot area |

### Design Guidelines

1. **Header**: Use your strongest visual — the hero screenshot or a branded
   graphic. Keep text large and minimal (it scales down on smaller phones).
2. **Search Results**: The first 1-2 seconds of attention. Show the core
   value prop visually. Avoid tiny text.

### Generating with the Playbook

The `make_screenshots.py` script generates the base screenshots. For Header
assets, use the `01-hero` frame — it's designed as the strongest visual.

```bash
# The 01-hero frame works as the Header asset
cp docs/asc/screenshots/en-US/iphone-69-01-hero.png docs/asc/header/header.png
```

### Per-App Status

| App | Header | Search Results |
|-----|--------|----------------|
| ProTune AI Camera | ❌ Not set | ❌ Not set |
| AI Camera - Music Reader | ❌ Not set | ❌ Not set |
| AI Music Radar | ❌ Not set | ❌ Not set |
| FinalCap | ❌ Not set | ❌ Not set |

> **Note**: These are optional but recommended. They improve conversion from
> browse/search to install. The screenshot shows "No header asset added" —
> this is normal for v1.0; add them when you have bandwidth.
