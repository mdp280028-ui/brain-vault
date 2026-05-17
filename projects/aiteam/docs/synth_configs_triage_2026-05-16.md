# Config Synthesizer test outputs — triage 2026-05-16

The 4 keyword-configs synthesized during the config-synthesizer sign-off run
(now untracked in `~/projects/asbestos-contractors/content/asbestos/keyword-configs/`).
For each, checked 5 possible downstream consumers in the asbestos pipeline +
the live AsbestosHQ site.

Lookup paths used:
- drafts → `~/projects/asbestos-contractors/content/asbestos/drafts/<slug>.json`
- feedback → `~/projects/asbestos-contractors/content/asbestos/feedback/<slug>-r*.md`
- approved → `~/projects/asbestos-contractors/content/asbestos/approved/guides/<slug>.json`
- assignment-batch → `~/projects/asbestos-contractors/content/asbestos/assignment-batch-*<slug>*.md`
- on site → `APPROVED_GUIDE_SLUGS` set in `~/projects/asbestoshq-site/src/components/GuideArticle.tsx`

## Triage results

| Slug | Draft? | Feedback? | Approved? | Assignment-batch? | On site? | Verdict |
|---|---|---|---|---|---|---|
| asbestos-attic-insulation | no | no | no | no | no | **TEST** |
| asbestos-window-glazing | no | no | no | no | no | **TEST** |
| asbestos-textured-paint | no | no | no | no | no | **TEST** |
| asbestos-cement-board | no | no | no | no | no | **TEST** |

All four came back clean across all five lookups. No downstream consumer for
any of them. They are test artifacts from the sign-off run.

## Adjacency notes (related-but-distinct slugs found nearby)

- A new `assignment-batch-chrysotile-attic-insulation-removal.md` appeared in
  the pipeline tree during this session (from the parallel assignment-drafter
  agent, not from config-synthesizer). That slug is **`chrysotile-attic-insulation-removal`**
  — distinct from the synthed **`asbestos-attic-insulation`**. The two
  primary keywords differ (one is cost-intent for chrysotile-specifically
  removal, the other is general identification + remediation). No collision.
- `drafter_queue.txt` (also new this session) only references
  `chrysotile-attic-insulation-removal` (already ✓-marked completed). None
  of the 4 synth test slugs are queued.
- The existing `vermiculite-insulation-guide.json` config is the closest
  pattern-match for `asbestos-attic-insulation` — overlapping vocabulary
  (vermiculite, Zonolite, W.R. Grace, Libby) but a different SERP target.
  If the operator decides to KEEP `asbestos-attic-insulation`, it should
  be checked for cannibalization with the shipped vermiculite guide before
  ingestion.

## Recommended next action

Operator decides per-slug. Each has two valid paths:

- **DELETE** (treat as test artifact):
  ```bash
  cd ~/projects/asbestos-contractors
  rm content/asbestos/keyword-configs/asbestos-attic-insulation.json \
     content/asbestos/keyword-configs/asbestos-window-glazing.json \
     content/asbestos/keyword-configs/asbestos-textured-paint.json \
     content/asbestos/keyword-configs/asbestos-cement-board.json
  ```
- **KEEP** (treat as real production config — operator plans to write the
  guide later). No git action needed; they'll get picked up when the
  matching `assignment-batch-<slug>.md` is added and `run-batch.sh` runs.

If KEEPing any, the natural follow-up is the assignment-drafter agent (which
already exists in `~/agents/assignment-drafter/`, per the parallel session
state) — point it at the kept slug to author the assignment-batch markdown
the pipeline expects.

The configs themselves were spot-checked during sign-off and are
publication-quality (vocabulary correct, entities canonical, ranges in-band).
Quality is not the deciding factor — only whether each slug is on the
editorial roadmap.
