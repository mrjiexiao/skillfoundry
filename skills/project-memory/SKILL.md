---
name: project-memory
description: Maintain durable, repository-local development memory across Codex sessions. Invoke explicitly as `$project-memory init` to initialize project memory, `$project-memory import` to preview or import past Codex tasks, `$project-memory close [focus]` to record the current session and refresh canonical state, or `$project-memory status` to read current state. Use when development history, resolved and unresolved issues, decisions, evidence, and next actions must survive across separate tasks without bloating AGENTS.md.
---

# Project Memory

Maintain a compact current-state document plus append-only session summaries. Keep `AGENTS.md` as a short entry point, never as the history store.

## Resolve the project root

1. Prefer the current Git root from `git rev-parse --show-toplevel`.
2. Otherwise prefer the SVN working-copy root from `svn info --show-item wc-root` when available.
3. Otherwise use the active workspace root and state that assumption.
4. Never initialize in the Codex home directory unless it is itself the intended project.

Use these repository-local paths:

```text
AGENTS.md
docs/project-memory/PROJECT_STATE.md
docs/project-memory/SESSION_INDEX.md
docs/project-memory/sessions/
```

Preserve user changes. Re-read every target immediately before editing and merge concurrent changes instead of overwriting them.

## Document language

- Write all newly created or updated project-memory prose in Simplified Chinese by default, including headings, summaries, status descriptions, decisions, problems, and next actions.
- Keep code identifiers, file paths, commands, task IDs, URLs, product names, and verbatim error messages in their original form.
- Follow an explicit user request or an established repository-wide documentation language when it requires another language.
- Do not mechanically translate untouched historical records merely to normalize language.

## Select the operation

- `init`: initialize or repair the project-memory structure.
- `import [options]`: discover, preview, and import historical Codex tasks associated with this project.
- `close [focus]`: summarize this session, add a log, and refresh current state. Treat trailing text as the session focus.
- `status`: read and summarize current state without writing.
- No argument: use `init` if the structure is absent; otherwise use `status`.

## Initialize

1. Inspect the repository structure, existing documentation, VCS status, recent relevant history, and build/test entry points.
2. Create missing directories and files from the templates in `assets/`.
3. Populate known facts from repository evidence. Mark unavailable facts as `Unknown`; never invent them.
4. Add the project-memory block from `assets/agents-project-memory-snippet.md` to the repository-root `AGENTS.md`.
   - Create `AGENTS.md` if absent.
   - Preserve all existing content.
   - If the marker block already exists, update that block in place instead of adding another copy.
5. Do not create an artificial session log during initialization unless meaningful development work also occurred.

## Import historical tasks

For `$project-memory import [options]`, read [references/import.md](references/import.md) before proceeding and follow its discovery, confirmation, deduplication, reconciliation, and validation workflow.

## Close a session

1. Ensure the project-memory structure exists; initialize missing pieces before continuing.
2. Build an evidence set from:
   - the current conversation;
   - files whose changes are already known from the current task;
   - commands and tests actually run;
   - existing project-memory files.
3. Redact passwords, tokens, private keys, cookies, authorization headers, and sensitive personal data. Record secret locations or variable names only when useful.
4. Create exactly one session file using local time:

   ```text
   docs/project-memory/sessions/YYYY-MM-DD-HHMM-short-topic.md
   ```

   Use a factual lowercase hyphenated topic. Add a numeric suffix rather than overwrite a collision.
5. Fill the session template with concise evidence:
   - distinguish completed, partially completed, blocked, and merely discussed work;
   - list changed files and commands when known;
   - never report an unrun test as passing;
   - link existing specs, commits, issues, ADRs, or logs instead of duplicating them.
6. Insert one newest-first row into `SESSION_INDEX.md`. Do not duplicate an existing session path.
7. Refresh `PROJECT_STATE.md` as the canonical current snapshot:
   - update the timestamp and latest-session link;
   - merge new facts with existing facts;
   - move genuinely resolved items out of active problems;
   - keep unresolved issues, blockers, and next actions explicit;
   - preserve still-valid architecture, constraints, commands, and file pointers;
   - do not erase unrelated work from concurrent sessions.
8. Keep `PROJECT_STATE.md` below 24 KiB when practical. Compact older completed detail into short outcomes with session links; retain active risks and decisions.
9. Re-read the new session file, `SESSION_INDEX.md`, `PROJECT_STATE.md`, and any files created or repaired during this operation. Check links, duplicate rows, status consistency, and secret redaction.
10. Re-read the files immediately before handoff and report the written paths and important open items.

## Report status

Read `PROJECT_STATE.md` first. Read `SESSION_INDEX.md` next. Open individual session files only when the requested detail requires them. Report the current phase, completed work, open issues, blockers, and next actions without modifying files.

## Templates

- Use `assets/project-state-template.md` for the canonical snapshot.
- Use `assets/session-index-template.md` for the chronological index.
- Use `assets/session-template.md` for each append-only session record.
- Use `assets/imported-session-template.md` for each imported historical task.
- Use `assets/agents-project-memory-snippet.md` for the idempotent `AGENTS.md` block.

The bundled templates use Simplified Chinese. Adapt headings to an explicitly requested or established repository convention only when all required information remains represented.
