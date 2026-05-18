# Keyword research sources — verbatim archive

This directory holds the original keyword research files imported into the
keyword registry. Files are stored verbatim — no edits, no dedup, no
reformatting. Each file is read-only reference for the registry rows derived
from it.

Imported on **2026-05-17** under audit `correlation_id=D2114073-8E5B-4F07-8793-884A21FAD537`
(`action=sources_archived`).

| File | Project | Fed registry | Parser used | Notes |
|---|---|---|---|---|
| `ASB_Keyword_Data_CONSOLIDATED_2026-05-17.md` | asbestos | `keyword_registry.md` | ASB-tier | April-22 consolidation of 4 source files; ~900 raw kw, ~161 deduped table rows |
| `ASB_Strategy_Framework_CONSOLIDATED_2026-05-17.md` | asbestos | `—` (scan-only) | none | Strategy/monetization prose; Fork G: scan-only, 0 registry rows contributed |
| `ASBESTOS_Batch4_Keyword_Research_Report_CONSOLIDATED_2026-05-17.md` | asbestos | `keyword_registry.md` | ASB-tier | May-17 Batch 4 session, 12 slugs recommended, ~130 deduped table rows |
| `MASTER_keywords.md` | SSG | `keyword_registry_ssg.md` | MASTER | SSG broad B2B scan, 9-col canonical schema, 309 raw rows / 267 unique |
| `SMART_Master_Research_Report_2026-05-17_consolidated.md` | SSG | `keyword_registry_ssg.md` | SMART | SSG alternatives/reviews/vs comparison cluster, 158 greenlit + ~30 sub-threshold |

## Rules

- Do not edit these files. They are operator-provided ground truth.
- If a source needs correction, replace the file and re-run import_research.py;
  do not patch in place.
- Re-imports are idempotent (rebuild_registry.py drift detection catches any
  delta). Re-importing the same file twice is safe.

## How to re-derive registry rows from these files

```
cd ~/agents/keyword-registry
python3 lib/import_research.py \
  ~/brain/projects/aiteam/keyword_registry/sources/ASB_Keyword_Data_CONSOLIDATED_2026-05-17.md \
  ~/brain/projects/aiteam/keyword_registry/sources/ASBESTOS_Batch4_Keyword_Research_Report_CONSOLIDATED_2026-05-17.md \
  --target-registry=asbestos
```

For SSG:
```
python3 lib/import_research.py \
  ~/brain/projects/aiteam/keyword_registry/sources/MASTER_keywords.md \
  ~/brain/projects/aiteam/keyword_registry/sources/SMART_Master_Research_Report_2026-05-17_consolidated.md \
  --target-registry=ssg
```

Both invocations write staging files to `~/agents/keyword-registry/staging/`,
NOT direct registry writes. Merge is a separate operator-approved step:

```
python3 lib/rebuild_registry.py --merge-staging=<staging_path>          # asbestos
python3 lib/rebuild_registry.py --site=ssg --merge-staging=<staging_path>  # ssg
```
