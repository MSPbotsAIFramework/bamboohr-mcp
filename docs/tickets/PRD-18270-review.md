# PRD-18270 · Code review

Status: frozen @ 87b5407

## Round 1 · 2026-11-13 · reviewed dev/PRD-18270 @ 245966a (base main @ d079d05)

Verdict: changes requested

### Findings

1. [must-fix] docs/tickets/PRD-18270.md:10-330 — The ticket doc is almost entirely in Chinese (需求理解 / 现状 / 产品设计 / 技术设计 / 测试用例 / 自测 / 交接 / 工单往来 sections). Vendor MCP repos must be English-only end to end — code, comments, tool descriptions, errors, docs, commit messages (`vendor-mcp-development-sop.md` §14 checklist: "仓库内容全部英文…文档…均无中文 (§0.4)"; `references/docs-tree.md:29` likewise says MCP repos only get an English `docs/tickets/PRD-XXXXX.md`). Only the two sections added in this round's rebuild (`Self-test`, `Handoff`, line 331 onward) are in English. This predates the 2026-10-27 vendor-mcp track split (original design was written 2026-09 under the generic bilingual convention) but it is being merged into `main` for the first time in this round, so it has to comply with the current repo-content standard before it lands. Needs translating to English (content/meaning unchanged) — this is a wording fix, not a requirements or test-case content change, so it does not need to go back through 5b/groom.
2. [must-fix] docs/tickets/PRD-18270.ac.json:1-298 — All 20 test-case entries have Chinese `text`/`module`/`notes`/`evidence` fields. Same English-only rule applies to the `.ac.json` file for MCP repos (`references/docs-tree.md:29`: "只写英文的 docs/tickets/PRD-XXXXX.md…和 .ac.json"). Needs translating to English; judgments/results (`pass`/`blocked`/`untested`) must not change, only the language of the free-text fields.
3. [must-fix] docs/modules/README.md, docs/modules/bamboohr-mcp.md, docs/tickets-index.md — these three files must not exist in a vendor MCP repo at all. `references/docs-tree.md:29` carves out an explicit exception for MCP repos: "不写 `docs/modules/`、`docs/tickets-index.md`" (the generic app-track accumulation-layer docs). They were added in commit `197b963` ("seed module docs + tickets index"), which applied the generic app-track docs-tree convention instead of the MCP-specific one. They are also written in Chinese, compounding finding 1. Remove all three files; if any of the "known unverified assumption" content in `bamboohr-mcp.md` (the un-verified response-field names, TC-BTO-020) is worth keeping, fold it into `PRD-18270.md`'s English Self-test/Handoff section instead — it is already covered there in substance.

No findings on the actual tool implementation: `src/bamboohr_mcp/tools/time_off.py` cleanly reuses the existing `BambooHRClient`/`_NO_CREDS`/`BambooHRError` pattern from `employees.py`, the parameter mapping (`employee_id`→`employeeId`, `time_off_type_id`→`type`) matches the tech design, all 17 automated test-case IDs in the ledger have a matching test function in `tests/test_time_off.py`, the README Tool List / Test Examples additions are consistent with the new tool, and the "MSPbots Connector" README section is byte-for-byte identical to `reference/readme-connector-section.md` (and untouched by this ticket — added separately in `edb3ab4`). `json.dumps(..., indent=2, ensure_ascii=False)` without a response-size/`limit` cap is a checklist gap per SOP §14 ("序列化无 `indent`"; list tools should have a `limit`), but it is byte-for-byte the same pattern already used by `bamboohr_list_employees`/`bamboohr_get_employee` elsewhere in this repo (`src/bamboohr_mcp/tools/employees.py:31-32`) — pre-existing repo-wide convention, not introduced by this ticket, so not a must-fix here; flagged as a suggestion for a separate repo-wide cleanup ticket.

### Fixes (round 1)

All three must-fix items addressed on the same `dev/PRD-18270` branch; no new branch opened, no requirements or test-case judgments changed:

1. Translated `docs/tickets/PRD-18270.md` lines 10-330 (Requirement understanding / Current state / Product design / Technical design / Test cases / Self-test / Handoff / Ticket correspondence) from Chinese to English in place; content and meaning unchanged, only the wording language. The two sections already in English (`Self-test (2026-11-13 rebuild...)`, `Handoff (2026-11-13)`) were left untouched. A handful of inline cross-references inside those two English sections that still named Chinese section headings ("工单往来", "技术设计", "现状", and a quoted "有就跑") were also updated to point at the new English heading names.
2. Translated all 20 entries in `docs/tickets/PRD-18270.ac.json` — `text`/`module`/`notes`/`evidence` fields — from Chinese to English. No `result` value changed (still 17 `pass`, 2 `blocked`, 1 `untested`); re-validated with `ticket_store.py validate --ac docs/tickets/PRD-18270.ac.json` -> `ledger OK: 20 cases`.
3. Removed `docs/modules/README.md`, `docs/modules/bamboohr-mcp.md`, `docs/tickets-index.md` (`git rm`). The one substantive fact in `bamboohr-mcp.md` worth keeping — the unverified response-field-name assumption tied to TC-BTO-020 — was already present in English in `PRD-18270.md`'s Open Questions (line 68) and Self-test section (TC-BTO-020 row, and the "Carried-over open items" bullet in Handoff), so nothing needed to be re-added.

Gates re-run on the fixed branch (docs-only change, no source file touched):
- `uv sync && uv run pytest -q` -> 24 passed, 0 failed (same count as round 1's self-test; no regression).
- `uv run ruff check src tests` -> all checks passed.
- Chinese-text scan (`grep -rnP '[\x{4e00}-\x{9fff}]' --exclude-dir=.git --exclude-dir=.venv --exclude='PRD-*-review.md' .`) -> no hits anywhere in the repo (this review doc itself is excluded from the scan per mcp-track.md §3.2 step 7, since it is written by the review leg).
- Local smoke (`/health`, `initialize` with placeholder gateway headers `domain: acmeinc`/`api-key: key123`, missing-headers case): `/health` -> 200; `initialize` -> 200 with a valid MCP response; missing both headers -> 401. Same behavior as round 1's self-test; no code changed this round so no change was expected.
- `gitleaks detect` -> not available on this host, skipped (same as round 1).
- README `## MSPbots Connector` section: untouched, confirmed identical to round 1 (no README edits this round).

Ready for round 2 review.

## Round 2 · 2026-11-13 · reviewed dev/PRD-18270 @ 87b5407 (base main @ d079d05)

Verdict: approved

### Findings

No must-fix findings this round. Diff since round 1 (`245966a...87b5407`) touches only docs — no `src`/`tests` change — so the round 1 implementation review (tool code, tests, README connector section) still stands unchanged.

Verified round 1's three must-fix items are actually resolved, not just claimed:
1. `grep -rnP '[\x{4e00}-\x{9fff}]' --exclude-dir=.git --exclude-dir=.venv --exclude='PRD-*-review.md' .` over the whole repo -> no hits. `docs/tickets/PRD-18270.md` and `PRD-18270.ac.json` are now English throughout (spot-checked Open Questions, Self-test, Handoff sections — content and judgments read the same as the round 1 Chinese version, only the language changed).
2. `docs/modules/README.md`, `docs/modules/bamboohr-mcp.md`, `docs/tickets-index.md` no longer exist in the tree (confirmed with `find docs -type f`); the one fact worth keeping from the removed `bamboohr-mcp.md` (TC-BTO-020's unverified response-field assumption) is present in `PRD-18270.md` (Open Questions line 68, Self-test row, Handoff "Carried-over open items").
3. `ticket_store.py validate --ac docs/tickets/PRD-18270.ac.json` -> `ledger OK: 20 cases`; result counts unchanged from round 1 (17 pass / 2 blocked / 1 untested — reconfirmed by parsing the JSON directly), so the translation did not touch any verdict.

Re-ran gates myself on `87b5407` (docs-only change, but confirming the claim rather than trusting it):
- `uv sync -q && uv run pytest -q` -> 24 passed, same as round 1, no regression.
- `uv run ruff check src tests` -> all checks passed.
- README `## MSPbots Connector` section (`README.md:11-82`) diffed byte-for-byte against `~/.claude/skills/vendor-mcp-development/reference/readme-connector-section.md` -> identical (only a trailing blank line difference, not content). Untouched by this ticket.

The round 1 suggestion (no `limit`/size cap on `json.dumps` for `bamboohr_list_time_off_requests`, matching the pre-existing repo-wide pattern in `employees.py`) stands as a suggestion for a separate repo-wide cleanup ticket, not a must-fix here.

Carried-over, non-blocking: TC-BTO-012/014 (blocked — product has not defined `start>end` behavior or a performance SLA; implemented as pass-through to upstream per the tech design's documented assumption) and TC-BTO-020 (not executed — no real/sandbox BambooHR account on this host). Per `references/mcp-track.md` §3.2, missing a vendor test account does not block; TC-BTO-012/014 reflect an accepted design assumption, not an unresolved review finding, and round 1 did not flag them either.

Ready to merge into `main`, publish to INT, and check the connector wiki.
