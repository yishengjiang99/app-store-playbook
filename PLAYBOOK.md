# App Store Playbook

Reusable playbook for App Store screenshots, metadata, and submission.
Run the same process for every app.

## Directory Structure

```
app-store-playbook/
├── make_screenshots.py      # Unified screenshot generator
├── PLAYBOOK.md              # This file
├── metadata-template/       # Metadata file templates
│   └── en-US/
│       ├── name.txt
│       ├── subtitle.txt
│       ├── description.txt
│       ├── keywords.txt
│       ├── promotional_text.txt
│       ├── support_url.txt
│       ├── marketing_url.txt
│       ├── privacy_url.txt
│       └── whatsnew.txt
└── configs/
    ├── protune.json         # ProTune screenshot config
    ├── omr-sheet-cam.json   # OMR screenshot config
    ├── ai-music-radar.json  # (template)
    └── finalcap.json        # (template)
```

## Step 1: Screenshots

### Config

Create a JSON config per app in `configs/`:

```json
{
    "app_name": "My App",
    "output_dir": "/path/to/docs/asc/screenshots/en-US",
    "source_dir": "/path/to/docs/asc/screenshots/en-US/source",
    "brand": {
        "bg": [17, 17, 17],
        "accent": [224, 168, 18],
        "ink": [245, 245, 247],
        "muted": [168, 168, 172],
        "card": [26, 26, 29]
    },
    "shots": [
        {
            "slug": "01-hero",
            "hero": "Hero value\\nprop here.",
            "sub": "Supporting explanation",
            "source": "hero-mockup.png"
        }
    ]
}
```

### Rules (from ProTune/OMR experience)

1. **4-5 frames max** — hero value prop first, then key features
2. **Hero text ≤5 words per line** — large, ExtraBold, centered at top
3. **Every frame explains functionality** — not just pretty screenshots
4. **Brand colors** — match the app's palette (dark + accent, or light + accent)
5. **Source images** — use real app UI captures or styled mockups in `source/`
6. **Output specs**:
   - `iphone-69-NN-slug.png` — 1320x2868, RGB, no alpha
   - `ipad-13-NN-slug.png` — 2064x2752, RGB, no alpha
7. **Naming** — `{device}-{NN}-{slug}.png`, NN is zero-padded order

### Generate

```bash
python3 make_screenshots.py --config configs/myapp.json
```

## Step 2: Metadata

Copy `metadata-template/en-US/` to your app's `docs/asc/metadata/en-US/`
and fill in each file:

| File | Max Length | Notes |
|------|-----------|-------|
| `name.txt` | 30 chars | App Store name |
| `subtitle.txt` | 30 chars | Appears under name |
| `description.txt` | 4000 chars | Full description, use sections |
| `keywords.txt` | 100 chars | Comma-separated, no spaces |
| `promotional_text.txt` | 170 chars | Shows above description, editable without new version |
| `support_url.txt` | URL | Support page |
| `marketing_url.txt` | URL | Marketing page (optional) |
| `privacy_url.txt` | URL | Privacy policy (required if collecting data) |
| `whatsnew.txt` | 4000 chars | "What's New" for version updates |

### Description Template Structure

```
[One-sentence value prop]

HOW IT WORKS
1. ...
2. ...
3. ...

[KEY DIFFERENTIATOR SECTION]
...

[FEATURE LIST]
• ...
• ...

[WHO IT'S FOR]
...

GOOD TO KNOW
[Honest limitations — builds trust, reduces bad reviews]

[PRIVACY / OPEN SOURCE if applicable]
```

## Step 3: Submit

Each app repo has `asc-submit-app-store.yml` workflow:

```bash
# Dispatch via gh-api
gh-api POST "/repos/{owner}/{repo}/actions/workflows/{workflow_id}/dispatches" \
  --data '{"ref":"main","inputs":{"build_number":"18","version_string":"1.0"}}'
```

The workflow:
1. Syncs metadata from `docs/asc/metadata/en-US/*.txt`
2. Uploads screenshots from `docs/asc/screenshots/en-US/*.png`
3. Attaches the specified build
4. Submits for review

## Per-App Checklist

- [ ] Screenshots generated (4-5 frames, both sizes)
- [ ] Screenshots uploaded to ASC (verify in dashboard)
- [ ] All metadata files filled in
- [ ] `whatsnew.txt` present for version updates
- [ ] Build number bumped
- [ ] Submitted via workflow
- [ ] Verified WAITING_FOR_REVIEW in ASC

## Apps Using This Playbook

| App | Repo | Config | Status |
|-----|------|--------|--------|
| ProTune AI Camera | yishengjiang99/photo-recipes | `configs/protune.json` | ✅ Live (v1.1) |
| AI Camera - Music Reader | yishengjiang99/omr-sheet-cam | `configs/omr-sheet-cam.json` | 🔄 v1.0 in review |
| AI Music Radar | yishengjiang99/earsheet | `configs/ai-music-radar.json` | 🔄 v1.0 in review |
| FinalCap | yishengjiang99/finalcut | `configs/finalcap.json` | 🔄 v1.0 in review |
