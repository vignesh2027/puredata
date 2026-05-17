# puredata — Website

> Landing page for **puredata**, the Python library for automatic data cleaning and silent drift detection.

**Live site:** https://vignesh2027.github.io/puredata

---

## What Is This Repo?

This repo contains the marketing website for [puredata.py](https://github.com/vignesh2027/puredata.py).
The site is a single-file static HTML page — no build step, no framework, no dependencies.
It deploys automatically to GitHub Pages on every push to `main`.

The Python library lives at → https://github.com/vignesh2027/puredata.py

---

## Install the Library

\`\`\`bash
pip install puredatalib
\`\`\`

\`\`\`python
import puredata

clean_df, report = puredata.clean(df)       # clean in one line
contract = puredata.watch(train_df)          # fit a data contract
result   = puredata.check(prod_df, contract) # validate production
\`\`\`

---

## What the Site Covers

| Section | Description |
|---|---|
| **AutoClean** | Nine-stage cleaning pipeline — encoding, whitespace, types, dates, duplicates, categories, units, nulls, outliers |
| **DataWatch** | Seven compatibility checks — schema, dtype, null rate, range, drift (dual-gate PSI + KS), cardinality, custom rules |
| **Comparison** | Feature table vs. pandas, pyjanitor, great_expectations, evidently |
| **Benchmarks** | Measured timings on real datasets (MacBook Pro M3) |
| **Integrations** | MLflow, Weights & Biases, DVC, Polars, scikit-learn, File I/O |
| **CLI** | \`puredata clean\`, \`watch\`, \`check\`, \`score\`, \`dashboard\` |
| **Roadmap** | Shipped features, next up, and future plans |

---

## Repo Structure

\`\`\`
puredata-website/
├── index.html          # entire site — single file, no build step
├── sitemap.xml         # XML sitemap for search engine indexing
└── .github/
    └── workflows/
        └── pages.yml   # auto-deploy to GitHub Pages on push to main
\`\`\`

---

## Local Preview

No install needed. Just open the file:

\`\`\`bash
# Option 1 — open directly in browser
open index.html

# Option 2 — serve with Python (avoids any CORS quirks)
python3 -m http.server 8080
# then visit http://localhost:8080
\`\`\`

---

## Deployment

Deployment is fully automatic via GitHub Actions.

Every push to \`main\` triggers [\`.github/workflows/pages.yml\`](.github/workflows/pages.yml),
which uploads \`index.html\` and \`sitemap.xml\` to GitHub Pages.

No manual steps. No build tools. No secrets needed.

**Pages must be enabled once in repo settings:**
\`Settings → Pages → Source → GitHub Actions\`

---

## Design

| Property | Value |
|---|---|
| Font | Inter (body) + JetBrains Mono (code) |
| Background | \`#0a0a0a\` |
| Accent | \`#00d4ff\` cyan |
| Mode | Dark only |
| Dependencies | Google Fonts CDN only — no JS frameworks |
| JS | ~30 lines — copy buttons + active nav highlight |

---

## Related Links

| | |
|---|---|
| Python Library | https://github.com/vignesh2027/puredata.py |
| PyPI Package | https://pypi.org/project/puredatalib/ |
| Live Website | https://vignesh2027.github.io/puredata |
| Issues | https://github.com/vignesh2027/puredata.py/issues |
| Changelog | https://github.com/vignesh2027/puredata.py/blob/main/CHANGELOG.md |

---

## Author

**Vignesh** — sole author and maintainer of puredata and this site.

MIT License
