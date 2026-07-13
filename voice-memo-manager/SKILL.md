---
name: voice-memo-manager
description: List, transcribe, summarize, and delete Apple Voice Memos synced via iCloud. Use when the user wants to access, read, summarize, or delete voice memos.
allowed-tools: "Bash(python3:*), Read, Write, Task"
compatibility: macOS with Voice Memos iCloud sync enabled, Python 3
license: 0BSD
---

# Voice Memo Manager

List, transcribe, summarize, and delete Apple Voice Memos synced via iCloud.

## Prerequisites

Voice Memos must be synced with iCloud on macOS.

## Tools

This skill includes three scripts in its `scripts/` directory. Script paths in this document are relative to this skill's directory — your working directory is likely elsewhere, so invoke them with absolute paths (e.g. `python3 <skill-dir>/scripts/extract-apple-voice-memos-metadata`).

- **`extract-apple-voice-memos-metadata`** — Queries the CloudRecordings.db SQLite database (read-only) and outputs CSV with columns: `title`, `date`, `duration`, `path`, `has_transcript`. Supports optional flags: `--limit N` (default 10), `--offset N`, `--search TERM`, `--after YYYY-MM-DD`, `--before YYYY-MM-DD`.
- **`extract-apple-voice-memos-transcript`** — Extracts the embedded transcript from a `.m4a` file's `tsrp` atom. Outputs timestamped text with filler words removed, intelligent line breaks, and paragraph breaks at natural pauses.
- **`delete-apple-voice-memo`** — Deletes a voice memo permanently: removes the `.m4a` audio file and cleans up the `CloudRecordings.db` record (handling Core Data triggers). Supports `--no-backup` to skip the database backup (default: backs up then auto-cleans on success; restores from backup on failure).

## Step 1: Select a voice memo

Run the metadata script to find the right recording. Choose flags based on what the user asked for:

- **No specific request** → run with no flags (returns 10 most recent)
- **User mentions a topic or keyword** → use `--search TERM`
- **User mentions a time period** → use `--after YYYY-MM-DD` and/or `--before YYYY-MM-DD`
- **User wants to see more results** → use `--offset N` to paginate, or `--limit N` to increase the batch size

Flags can be combined, e.g. `--search work --after 2026-01-01 --limit 5`.

```bash
python3 scripts/extract-apple-voice-memos-metadata [flags]
```

If the user's request unambiguously identifies a single memo (e.g., "my latest voice memo", or a search that returns exactly one match), proceed directly with that memo without asking. Otherwise, present the results as a numbered list showing title, date, and duration — mark any memo with `has_transcript` = `no` as "(no transcript)" — and ask the user which memo they'd like to work with.

**Error handling:**
- "Database not found" → Voice Memos iCloud sync is not enabled on this Mac.

## Step 2: Extract and correct transcripts

For each selected memo, spawn a subagent with fresh context to extract the transcript and correct speech recognition errors. Using a subagent keeps raw transcripts out of the main context. The subagent prompt must contain:

1. The `title` and `path` values of the recording
2. The absolute path to this skill's `scripts/extract-apple-voice-memos-transcript` script
3. The absolute path to the colleague roster: `/Users/huabinhong/Library/Mobile Documents/iCloud~md~obsidian/Documents/hbh/知识库/baidu/研发效能组同事名单.md`
4. These instructions: run the transcript script, then correct speech recognition errors (homophones, misrecognized proper nouns like GSD/CC/AFlow/token, self-corrected speech errors) based on context, preserving original meaning and tone. **When a person's name appears in the transcript, read the colleague roster and match it to the correct real name** (e.g. 语音识别的"陈伟"可能是名单里的"陈帅"; "于雷"可能是"于磊"). Return the corrected timestamped transcript.

If your environment has no subagent or task tool, do the same work yourself: run the transcript script and correct the transcript.

Only run the transcript script directly (without correction) if the user explicitly asks to see the raw timestamped transcript:

```bash
python3 scripts/extract-apple-voice-memos-transcript "<FILENAME>.m4a"
```

**Error handling (reported by the subagent):**
- "tsrp atom not found" → This recording does not have an embedded transcript. Apple generates transcripts on-device and not all recordings will have one.
- File not found → The recording file may not have synced to this Mac yet.

## Step 3: Write unified journal entry

After **all** selected memos' transcripts are extracted and corrected, read `PROMPT.md` and process them together to produce a unified journal entry. Read all transcripts before writing — the 概述 and 观察总结 should be unified across all memos of the day, not per-memo.

The journal entry has this structure:

1. **概述** — Unified narrative summary across all memos of the day.
2. **转写** — All corrected transcripts in one collapsible `<details>` block, separated by `---` with each labeled by title and time.
3. **Todo** — Action items extracted from all memos (skip if none).
4. **观察总结** — A **metacognitive observer** speaking directly to the user (second person "你"), observing how the user *thinks* — patterns, blind spots, contradictions, emotional state. Not content restatement.

### Save to daily journal

Append under a `## 语音日记` section in the daily journal:

```
<Obsidian vault>/03.每日安排&记录/日记/<年> <月>月/YYYY-MM-DD.md
```

e.g. `/Users/huabinhong/Library/Mobile Documents/iCloud~md~obsidian/Documents/hbh/03.每日安排&记录/日记/26年 5 月/2026-05-28.md`

**Rules:**

- **Read the existing file first.** If it already has a `## 语音日记` section, replace its content. If not, append the section.
- **One day = one section.** All memos are unified into a single entry, not listed separately.

## Step 4: Delete a voice memo

When the user asks to delete a voice memo, first run the metadata script to confirm the target recording exists and get its `path` value. Then run the delete script with the recording's filename:

```bash
python3 scripts/delete-apple-voice-memo "<FILENAME>.m4a" [--no-backup]
```

The script will:
1. Delete the `.m4a` audio file
2. Back up `CloudRecordings.db` (unless `--no-backup`)
3. Temporarily remove Core Data triggers, delete the database record, fix folder counts, and restore triggers
4. Verify the deletion succeeded
5. Auto-clean the backup on success, or restore from backup on failure

**After deletion**, run the metadata script again to confirm the recording no longer appears in the list.

**Error handling:**
- "File not found" → The recording file may not have synced to this Mac. Check if the database record exists.
- "Database not found" → Voice Memos iCloud sync is not enabled on this Mac.
- Any database error → The script restores from backup automatically. Report the error to the user.
