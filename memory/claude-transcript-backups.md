---
name: claude-transcript-backups
description: Automated transcript backups exist because an unclean shutdown zeroed a sidebar index file and silently hid a week of work.
metadata: 
  node_type: memory
  type: project
  originSessionId: 5b1c4619-ffa9-4ce1-8797-d3fb48fef09d
  modified: 2026-07-22T10:07:49.879Z
---

On 2026-07-22 a Claude Code conversation vanished from the desktop sidebar. The
transcript was intact; what died was the app's sidebar index entry at
`%APPDATA%\Claude\claude-code-sessions\<workspace>\<profile>\local_<id>.json`,
which had become **163,305 bytes of pure NUL** after an unclean shutdown. The app
silently skips index entries it cannot parse, so the chat just disappeared with no
error. The affected session was the Sony F130 parts control work
([[sony-smt-parts-verification]]), transcript `29b2b51c`, 1672 messages spanning
16–22 July 2026.

Repair method: clone a known-good index entry, swap `sessionId`, `cliSessionId`,
`createdAt`, `lastActivityAt`, `lastFocusedAt`, `title`; validate as JSON before
writing; fully quit the app (it survives in the tray) to force a re-read. The
original title is unrecoverable once the file is zeroed.

**Why:** transcripts live in `~/.claude/projects/<slug>/<sessionId>.jsonl` and
survive this failure, but the only route back to them was a UI that had lost the
pointer. A day's thinking was effectively invisible.

**How to apply:** readable exports of every conversation now land in
`Documents\Dantec\ClaudeBackups\` as markdown, written by
`Documents\Dantec\Tools\Export-ClaudeTranscripts.ps1` under the Windows scheduled
task `ClaudeTranscriptBackup` (every 2 hours, incremental). If a chat goes missing,
read the backup first, and resume the live session with
`claude --resume <sessionId>` rather than hunting the sidebar. Note that
`Register-ScheduledTask` needs elevation here; `schtasks /Create` works unelevated.
