# Import historical tasks

Read this reference only for `$project-memory import [options]`.

## Supported forms

```text
$project-memory import
$project-memory import --dry-run
$project-memory import --apply
$project-memory import --since YYYY-MM-DD
$project-memory import --task TASK_ID
```

Treat `import` without `--apply` as a preview. Never import every visible task merely because the user invoked the command.

## Discover and confirm scope

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

## Import confirmed history

1. Read only confirmed tasks. Process long histories in bounded batches and summarize each task independently before synthesizing project state.
2. Use the original task date for chronology and the imported-session template in `../assets/imported-session-template.md`.
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
