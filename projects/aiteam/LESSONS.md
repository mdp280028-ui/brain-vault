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

### Brain vault git-init (later that day)

- Pre-flight checks catch blockers before they're expensive. Scoping `~/brain/` size, existing git state, existing SSH state, and crontab state up front flagged the missing `~/.ssh/` directory BEFORE we got to step 6 (push) where it would have stalled the chain.
- Verify operator-stated assumptions before relying on them. The build prompt said "the ust-contractors repo uses SSH on this machine, so the key should already be there." It wasn't. Don't pattern-match to "this should already work" — check.
- Pause points are leverage. Reporting the staged file list before the initial commit gave the operator a chance to opt `dashboard.log` and `inbox/` out of permanent history.
- `ssh -T git@github.com` exits 1 on success. GitHub doesn't provide shell access, so SSH disconnects with exit 1 after authenticating. Look for `Hi <user>! You've successfully authenticated` in stdout, not exit code 0.
- `ssh-keyscan -t <key-type> <host> >> ~/.ssh/known_hosts` is the right way to pre-populate host fingerprints for non-interactive SSH. Avoids the interactive prompt without disabling host verification.
- Specific token-pattern regex beats loose secret-keyword grep for pre-commit scans. `(sk-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{20,}|xoxb-[a-zA-Z0-9-]{30,}|AKIA[A-Z0-9]{16})` catches actual credentials; `secret|api_key|password` returns false positives from any doc mentioning secrets management.
- Per-repo `git config user.name` works without `--global` — useful for machines hosting repos under multiple identities. The `--local` flag is implicit when run inside a repo. Don't pollute global identity for one project.
- `git rm --cached <file>` (with `-r` for dirs) removes from index without touching the working tree. Correct unstage pattern when adjusting gitignore mid-build.

