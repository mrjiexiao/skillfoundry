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
- No argument: use `init` if the structure is absent; otherwise use `close`.

## Initialize

1. Inspect the repository structure, existing documentation, VCS status, recent relevant history, and build/test entry points.
2. Create missing directories and files from the templates in `assets/`.
3. Populate known facts from repository evidence. Mark unavailable facts as `Unknown`; never invent them.
4. Add the project-memory block from `assets/agents-project-memory-snippet.md` to the repository-root `AGENTS.md`.
   - Create `AGENTS.md` if absent.
   - Preserve all existing content.
   - If the marker block already exists, update that block in place instead of adding another copy.
5. Do not create an artificial session log during initialization unless meaningful development work also occurred.
6. Do not commit or push.

## Import historical tasks

Support these forms:

```text
$project-memory import
$project-memory import --dry-run
$project-memory import --apply
$project-memory import --since YYYY-MM-DD
$project-memory import --task TASK_ID
```

Treat `import` without `--apply` as a preview. Never import every visible task merely because the user invoked the command.

### Discover and confirm scope

1. Ensure the project-memory structure exists.
2. Identify the current project's canonical path and VCS root. Record a remote URL only as supporting evidence; do not expose embedded credentials.
3. Prefer the host's Codex task APIs for listing and reading tasks, such as `list_threads` and `read_thread` when available. Treat “task”, “thread”, “chat”, and “conversation” as equivalent host terminology.
4. Filter candidates using strong association evidence in this order:
   - an exact workspace or project identifier match;
   - an exact canonical working-directory or repository-root match;
   - explicit references to the repository path plus corroborating file paths.
5. Do not select a task based only on a similar title or shared technology name.
6. Exclude the current task, unrelated tasks, already imported task IDs, and tasks outside `--since` or `--task` filters.
7. Present a compact candidate table containing task ID, date, title, association evidence, and import status. Ask for confirmation before writing unless `--apply` names an already confirmed set in the current conversation.
8. If task APIs are unavailable, request exported transcripts or explicit task identifiers. Do not inspect raw internal Codex session storage unless the user explicitly authorizes that fallback.

### Import confirmed history

1. Read only confirmed tasks. Process long histories in bounded batches and summarize each task independently before synthesizing project state.
2. Use the original task date for chronology and the imported-session template in `assets/imported-session-template.md`.
3. Write one session record per imported task. Include a stable `Source task ID` line and, when available, a task link. Use the filename:

   ```text
   docs/project-memory/sessions/YYYY-MM-DD-HHMM-imported-short-topic.md
   ```

4. Make import idempotent:
   - search all session records for the exact source task ID before writing;
   - skip an exact match and report it as already imported;
   - if the ID is absent but the content appears duplicated, skip it as uncertain and request review;
   - never overwrite an existing session record or create a second record for the same task ID.
5. Treat conversation claims as historical evidence, not proof of the repository's current state. Label each important outcome as:
   - repository-verified;
   - conversation-reported;
   - superseded by later work;
   - no longer verifiable.
6. Reconcile claims against current files, VCS history, tests, issues, and later imported sessions when available. Never mark a reported fix as currently resolved without corroboration.
7. Redact secrets and sensitive personal data before writing. Preserve useful variable names or secret locations without values.
8. Insert imported records into `SESSION_INDEX.md` by original task date while preserving existing rows and schema.
9. After the batch, synthesize `PROJECT_STATE.md` once:
   - preserve the current state as authoritative;
   - add durable historical decisions and verified milestones;
   - record contradictions and unresolved uncertainty explicitly;
   - avoid replacing newer facts with older imported claims.
10. Validate source-ID uniqueness, links, chronological ordering, status consistency, and redaction. Report imported, skipped, uncertain, and failed task IDs separately.
11. Do not commit or push unless explicitly requested.

## Close a session

1. Ensure the project-memory structure exists; initialize missing pieces before continuing.
2. Build an evidence set from:
   - the current conversation;
   - `git status`/`git diff` or the SVN equivalents;
   - files actually changed;
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
9. Re-read the three changed memory files and check links, duplicate rows, status consistency, and secret redaction.
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
