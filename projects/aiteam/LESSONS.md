# AITEAM Lessons Learned

Growing log. Each ctx save appends lessons under a date heading.

---

## 2026-05-15

- Verify wrapper model attribution BEFORE shipping cost-sensitive decisions. `ai-do.sh` silently used Opus (Claude Code default inheritance) and invalidated step 17 v1's "Sonnet wins" tier test. Pin `--model` explicitly in every wrapper.
- Bash heredoc `${5:-{}}` has brace-matching ambiguity — corrupted 16 audit_log payloads with a literal trailing `}`. Use `PAYLOAD="${5-}" ; [ -z "$PAYLOAD" ] && PAYLOAD="{}"`.
- `claude -p --output-format json` returns `total_cost_usd` and per-model `modelUsage` directly. No need to maintain a pricing map. Trust Claude Code's own numbers.
- Claude Code routes some work to Haiku even on Sonnet/Opus calls. Per-tier cost logging requires one row per model in `modelUsage`, not per call.
- Operator-suggested fixes deserve verification before literal implementation. The "export AGENT_ID env var" suggestion wouldn't have stuck — wrappers do `AGENT_ID="${2:-shell}"` which shadows env. Pass as `$2` instead.
- Schema mismatches in specs vs reality silently break SQL. The ctx-system spec used `timestamp`/`cost_usd`/`title`; real schema is `ts`/`estimated_cost_usd`/`description`. Schema-check before writing queries.
- Empty-data graceful handling is load-bearing for credibility. `/weekly_review` early-exits when <2 diary days. Better to honestly say "not enough data" than hallucinate.
- macOS bash is 3.2.57. No `${var,,}`, no assoc arrays, no `mapfile`/`readarray`. `${var/pattern/}` works. Process substitution works. `case` patterns work.
- Append-only audit logs are recoverable; mutable ones aren't. When a bug corrupts history, write a corrective row pointing back. Never UPDATE/DELETE. The corruption itself is a fact worth recording.
- Round-2+ discussion prompts MUST explicitly require engagement with a specific prior claim. Without this, multi-round devolves into parallel monologues — each agent restating its position.
- Document gaps only after verification. Initially documented `rubric.md` as not git-tracked; it actually was (commit `8ffd923`). Verify before claiming.
- Cost cap discipline: budget for retries. A client-side JSON parse error in step 18 burned a 3rd run on a 2-run cap. Save to file first, parse second.
