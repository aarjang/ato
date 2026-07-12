# ato Token Measurements — 2026-06-28

Project: ato (self-measurement — small project, ~2800 lines)
> Note: repeat on a large project (30k+ lines) for meaningful data

---

## Focus File Stats (measured)

| Metric | Value |
|---|---|
| `.ato_focus_auto.md` lines | 478 |
| `.ato_focus_auto.md` words | 1,863 |
| `.ato_focus_auto.md` ~tokens | ~2,400 (words × 1.3) |
| Embedded files count | 8 |

### Embedded files (8 × head -80):
- `bin/ato` (2752 lines → 80 embedded)
- `README.md` (837 lines → 80 embedded)
- `bin/cms` (30 lines → all)
- `.claude/settings.json` (17 lines → all)
- `install.sh` (53 lines → all)
- `.gitignore` (15 lines → all)
- `templates/TASKS.md` (10 lines → all)
- `DECISIONS.md` (14 lines → all)

### Token breakdown inside focus file:
| Component | Lines | ~Tokens |
|---|---|---|
| CONTEXT.md (embedded) | 93 | ~560 |
| DECISIONS.md (embedded) | 14 | ~85 |
| 8 files × up to 80 lines | ~640 | ~3,840 |
| Headers/structure/rules | ~30 | ~180 |
| **Total** | **~778** | **~4,665** |

> Discrepancy: word-count estimate (~2,400) vs line estimate (~4,665).
> Word count is more reliable — the line estimate overcounts because most lines
> in code files are short (< 6 tokens/line average for code).
> Actual: ~2,400–3,000 tokens is the realistic range.

---

## Cache Behavior — Known Issue

Line 1091 in `bin/ato`:
```
# Area: current changes  |  branch: main  |  2026-06-28 21:56
```
Timestamp changes every run → focus file never cache-hits.
CONTEXT.md content follows immediately after → also never cached.

---

## Baseline Context — NEEDS MANUAL MEASUREMENT

Run these steps in a fresh Claude Code session on a large project:

1. `rm -f /tmp/.ato_session_*` (clear lock)
2. Open Claude Code in the project
3. Before typing anything, run `/context`
4. Record:

| Metric | Value |
|---|---|
| System prompt tokens | ___ |
| CLAUDE.md | ___ |
| `.ato_focus_auto.md` (injected via hook) | ___ |
| Total baseline before first message | ___ |

---

## Cache Check — NEEDS MANUAL MEASUREMENT

After sending one message, check statusline or `/status`:

| Metric | Value |
|---|---|
| `cache_creation_input_tokens` | ___ |
| `cache_read_input_tokens` | ___ |
| Cache hit ratio (read / total) | ___ |

Expected if timestamp bug exists: cache_read ≈ 0 on every message.

---

## Key Finding from Current Design

The focus file signal quality on this project is poor:
- `bin/ato` and `README.md` were selected because they were recently modified
- These are the two largest files in the project — 80 lines each gives almost
  no useful signal for a 2,752-line binary
- A Claude that reads this focus file and then tries to work on any feature
  will almost certainly need to open `bin/ato` again for more context

The value proposition only holds if the embedded 80 lines contain the
*specific function or section* Claude needs — which is unlikely when the
file is 2,752 lines and no function-level targeting exists.
