# Brain SCHEMA

This vault holds knowledge that AI agents read and the operator reads.

## Structure
- `meta/` — cross-project knowledge (voices, decisions)
- `operator/` — personal notes; agents do not read
- `projects/<name>/` — per-project subtree
  - `raw/` — immutable source files (PDFs, scrapes, transcripts)
  - `wiki/` — clean AI-generated summary pages with cross-references
  - `inbox/` — agents drop findings here, librarian sorts overnight
  - `outputs/` — drafts produced by agents
  - `silver_platters/` — weekly summary tables
  - `diary/` — daily orchestrator diaries
  - `docs/` — operational docs (restore procedure, etc.)
- `raw_dropzone/` — drop messy files here; conversion hook turns them into markdown

## Naming
- Files: lowercase-with-dashes.md
- Dates: YYYY-MM-DD
- Voices captures: `meta/voices/team/<source>/YYYY-MM-DD_<platform>.md`

## Privacy
- `operator/` is operator-only. Agents must never read it.
- Personal voices (`meta/voices/personal/`) is operator-only.
- Team voices (`meta/voices/team/`) is the agent corpus.
