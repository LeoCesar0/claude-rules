---
description: Read the last 2 chat transcripts of the current project to recall what was being worked on and suggest next steps — continue the previous task or move to a new one
disable-model-invocation: true
allowed-tools: Read, Glob, Bash(ls *, head *, tail *, grep *, wc *, jq *, pwd, sed *, stat *, awk *)
effort: medium
---

# Recall

Read the last 2 chat transcripts of the current project to remind the user what they were working on. Output is a concise summary per chat plus a single suggested next step. The summary should jog memory — not document.

## Invocation

- `/ahead:recall` — print the recall summary to chat (no file written)

## Step 1: Locate the project's session files

Claude Code stores transcripts at `~/.claude/projects/<slug>/*.jsonl`, where `<slug>` is the current working directory with `/` replaced by `-`.

```bash
slug=$(pwd | sed 's|/|-|g')
dir="$HOME/.claude/projects/$slug"
```

If `$dir` does not exist or contains no `.jsonl` files: tell the user "No previous sessions found for this project." and stop.

## Step 2: Pick the 2 previous sessions

List `.jsonl` files by mtime descending. **Skip the first** (= current session — its file is being written to right now and always has the freshest mtime). Take the next 2.

```bash
ls -t "$dir"/*.jsonl 2>/dev/null | tail -n +2 | head -n 2
```

- If 0 files remain: stop with the "no previous sessions" message.
- If 1 file remains: process that one and explicitly note that only one prior session exists.

## Step 3: Extract content (adaptive by session size)

First measure the file:

```bash
lines=$(wc -l < "$file")
size=$(stat -c '%s' "$file")
```

Choose a read strategy based on size — be generous with context (modern context windows have room):

### Small session (< 500 lines)
Read the **whole file** with the `Read` tool. No windowing needed.

### Medium session (500–3000 lines)
Wide windows — capture both ends:

```bash
head -n 500 "$file"            # opening: intent, framing, first few exchanges
tail -n 1500 "$file"           # closing: recent work, current status, last messages
```

### Large session (3000+ lines)
Three windows — opening, sample of middle, closing:

```bash
head -n 800 "$file"                                    # opening
awk -v total="$lines" -v start=$((lines/2 - 150)) -v end=$((lines/2 + 150)) \
    'NR>=start && NR<=end' "$file"                     # middle sample (~300 lines)
tail -n 2000 "$file"                                   # closing
```

### Per-session derived signals (always run)

Regardless of strategy above, also collect:

```bash
# Top 30 files touched — strongest signal of work area
grep -oE '"file_path":"[^"]+"' "$file" | sort | uniq -c | sort -rn | head -n 30

# Bash commands run — what was being tested/built
grep -oE '"command":"[^"]{1,200}"' "$file" | sort -u | head -n 30

# Session bounds for reporting
wc -l "$file"
stat -c '%y' "$file"
```

### Clean text extraction with jq (preferred when available)

When `jq` is installed, prefer it over raw grep for message content — it strips the JSON noise:

```bash
# All user messages, cleanly
jq -r 'select(.type=="user") | .message.content // (.message.content[]?.text // empty)' "$file" 2>/dev/null

# All assistant text messages
jq -r 'select(.type=="assistant") | .message.content[]? | select(.type=="text") | .text' "$file" 2>/dev/null
```

If `jq` is missing or the schema doesn't match, fall back to plain grep — do not fail.

## Step 4: Synthesize per chat

Don't just look at the last few messages — use the **whole window** you extracted. Look for:

- **Topic** (3–7 words) — derive from the cluster of files touched + first user messages + dominant subject in user messages throughout the session. Not just the first message; sessions often pivot.
- **Motivation** — one sentence on why the work started. Pull from the opening user messages.
- **Problem** — one sentence on what was being solved. The "what" can become clearer mid-session than at the start.
- **Status** — pick one based on the **tail** of the transcript:
  - `done` — last assistant said "tests pass", "shipped", "resolved", commit landed
  - `in-progress` — work clearly underway, no closure
  - `blocked` — explicit failure, unanswered question, or error state
  - `abandoned` — user pivoted mid-flow with no return
  - `unclear` — none of the above signals strong enough
- **Next step** — derived from:
  1. An explicit next step the assistant proposed in its last message, or
  2. A pending user question that was never answered, or
  3. Empty if `status: done`

Be specific. Mention actual file names, function names, or concepts that came up — not generic phrases like "the refactor" or "the bug fix". If the session worked on `auth.ts:validateToken`, name it.

## Step 5: Output format

Print to chat — no file written. Keep each bullet to one tight sentence, but the sentence must be **specific**.

```markdown
## Chat 1 — <topic>
- **Motivation**: …
- **Problem**: …
- **Status**: <done | in-progress | blocked | abandoned | unclear>
- **Suggested next step**: …

## Chat 2 — <topic>
- **Motivation**: …
- **Problem**: …
- **Status**: …
- **Suggested next step**: …

---
**Suggestion**: <one line — continue chat X because Y / both look done; obvious next is Z / no clear direction, tell me where to focus>
```

Match the output language to the user's working language (Portuguese-BR, English, etc. — infer from the active conversation).

## Rules

- Do **not** mix content between the two chats. Present them separately. Only synthesize across them in the final `Suggestion` line.
- Read **generously** — small sessions in full; large sessions with wide windows. The cost of being vague is worse than the cost of reading more.
- Be **specific** in the output — file names, function names, concrete concepts. If the summary could fit any project, it's too generic.
- Do **not** write any file — output is chat-only by design.
- If both chats look concluded and the next step is obvious from cues in the transcripts (a TODO mentioned, an observation referenced, a spec waiting to be implemented), suggest it. Otherwise, ask the user to redirect.

## Edge cases

- **No previous sessions** → "Sem histórico anterior neste projeto." / "No previous sessions for this project." Stop.
- **Only 1 previous session** → show that one; note the absence of a 2nd.
- **Very short session** (`wc -l` < 20) → label as "brief session, limited context" — still show what you can.
- **Two sessions on the same topic** → still present separately; in the suggestion line, recommend continuing the most recent.
- **`jq` not installed** → fall back to plain grep; do not fail.
