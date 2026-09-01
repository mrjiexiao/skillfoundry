# SkillFoundry

Reusable Codex skills focused on durable, verifiable development workflows.

可复用的 Codex Skills，专注于持久、可验证的开发工作流。

## Included skills

### Project Memory

`project-memory` maintains repository-local development memory across separate Codex tasks without turning `AGENTS.md` into a long-running development log.

It organizes memory into:

- `PROJECT_STATE.md`: the compact, authoritative snapshot of the project's current state.
- `SESSION_INDEX.md`: a newest-first index of recorded work sessions.
- `sessions/`: append-only session summaries containing decisions, evidence, unresolved issues, and next actions.
- A small idempotent block in `AGENTS.md` that tells future Codex tasks where to load context.

The skill supports four explicit operations:

| Command | Purpose |
|---|---|
| `$project-memory init` | Initialize or repair the repository-local memory structure. |
| `$project-memory status` | Read the current state without changing files. |
| `$project-memory close [focus]` | Record the current task and refresh the canonical state. |
| `$project-memory import [options]` | Preview or import strongly associated historical Codex tasks. |

`project-memory` is explicit-only by default. It does not update project memory unless you invoke it.

#### 中文说明

`project-memory` 用于在不同 Codex 任务之间保留项目上下文，同时避免把全部开发历史堆进 `AGENTS.md`。它将当前状态、会话索引和仅追加的历史记录分开管理，持续保存已经完成的工作、设计决策、验证证据、遗留问题和下一步行动。

默认生成简体中文项目记忆；代码标识符、路径、命令和原始错误信息保持原样。如果项目已有其他文档语言约定，Skill 会遵循项目约定。

## Installation

Clone this repository and copy the skill directory into your Codex skills directory:

```bash
git clone https://github.com/mrjiexiao/skillfoundry.git
mkdir -p ~/.codex/skills
cp -R skillfoundry/skills/project-memory ~/.codex/skills/project-memory
```

Restart Codex or open a new task if the skill is not discovered immediately, then invoke it explicitly:

```text
$project-memory status
```

To update an existing installation, replace only the installed `project-memory` directory with the version from this repository. Review local modifications before replacing it.

## Generated project files

The skill creates or maintains these files inside the active project, not inside this repository:

```text
AGENTS.md
docs/project-memory/PROJECT_STATE.md
docs/project-memory/SESSION_INDEX.md
docs/project-memory/sessions/
```

It prefers the current Git root, then the SVN working-copy root, and finally the active workspace root.

## Privacy and safety

The skill instructs Codex to redact passwords, tokens, private keys, cookies, authorization headers, and sensitive personal data from project-memory files. It does not commit or push changes unless the user explicitly requests that separately.

Historical task import is conservative: candidates must have strong project-association evidence, imports require confirmation, and source task IDs are recorded to prevent duplicates.

Always review generated memory before publishing a project repository, especially when the underlying development history contains confidential information.

## Repository layout

```text
skills/
└── project-memory/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    └── assets/
        ├── agents-project-memory-snippet.md
        ├── imported-session-template.md
        ├── project-state-template.md
        ├── session-index-template.md
        └── session-template.md
```

## License

MIT. See [LICENSE](LICENSE).
