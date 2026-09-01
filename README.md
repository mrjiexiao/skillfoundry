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
| `$project-memory status` | Read and summarize the current project state. |
| `$project-memory close [focus]` | Record the current task and refresh the canonical state. |
| `$project-memory import [options]` | Preview or import strongly associated historical Codex tasks. |

`project-memory` is explicit-only by default and updates project memory when you invoke it.

#### 中文说明

`project-memory` 用于在不同 Codex 任务之间保留项目上下文，同时避免把全部开发历史堆进 `AGENTS.md`。它将当前状态、会话索引和仅追加的历史记录分开管理，持续保存已经完成的工作、设计决策、验证证据、遗留问题和下一步行动。

默认生成简体中文项目记忆；代码标识符、路径、命令和原始错误信息保持原样。如果项目已有其他文档语言约定，Skill 会遵循项目约定。

## 中文使用指南

### 1. 安装 Skill

```bash
git clone https://github.com/mrjiexiao/skillfoundry.git
mkdir -p ~/.codex/skills
cp -R skillfoundry/skills/project-memory ~/.codex/skills/project-memory
```

如果 Codex 没有立即发现新 Skill，请重新启动 Codex 或新建一个任务。这个 Skill 默认通过 `$project-memory` 显式调用。

### 2. 在项目中初始化记忆

在需要保存长期上下文的项目中打开 Codex，然后执行：

```text
$project-memory init
```

它会识别当前 Git 根目录；如果不是 Git 项目，则尝试识别 SVN 工作副本，最后才使用当前工作区。初始化后会创建：

```text
AGENTS.md
docs/project-memory/PROJECT_STATE.md
docs/project-memory/SESSION_INDEX.md
docs/project-memory/sessions/
```

其中：

- `PROJECT_STATE.md` 是项目当前状态的权威快照，记录当前阶段、已完成工作、问题、风险和下一步行动。
- `SESSION_INDEX.md` 是会话索引，最新记录排在最前。
- `sessions/` 保存仅追加的会话摘要，避免旧历史被覆盖。
- `AGENTS.md` 只增加一小段入口说明，让后续 Codex 任务知道应先读取哪些记忆文件。

重复执行 `init` 会补齐或修复缺失结构，并保留已有项目说明。

### 3. 日常使用流程

建议在一次开发任务结束、准备切换任务或结束当前会话时执行：

```text
$project-memory close
```

也可以附带本次工作的重点：

```text
$project-memory close 修复登录超时并补充回归测试
```

`close` 会结合当前对话、当前任务中已知的文件修改、已经执行的命令和测试结果，完成两件事：

1. 在 `docs/project-memory/sessions/` 新增一份本次会话记录。
2. 更新 `PROJECT_STATE.md`，保留仍然有效的约束、未解决问题和下一步行动。

下次打开同一个项目时，可以先查看当前状态：

```text
$project-memory status
```

`status` 以只读方式汇总当前记忆。由于初始化时已经在 `AGENTS.md` 加入入口说明，后续 Codex 任务通常也会在开始实质性工作前读取 `PROJECT_STATE.md`。

### 4. 导入历史 Codex 任务（可选）

先预览与当前项目有强关联证据的历史任务：

```text
$project-memory import
```

常用筛选方式：

```text
$project-memory import --since 2026-01-01
$project-memory import --task TASK_ID
$project-memory import --dry-run
```

`import` 默认以只读方式生成候选预览。Skill 会优先使用工作区标识、项目标识或仓库根目录等强证据关联任务；确认候选范围后再使用 `--apply` 导入。

确认候选任务后，再明确要求应用已确认的范围：

```text
$project-memory import --apply
```

导入记录会保存来源任务 ID 以避免重复，并将历史对话中的说法与当前仓库能够验证的事实分开标注。

### 5. 命令速查

| 命令 | 是否写文件 | 用途 |
|---|---:|---|
| `$project-memory init` | 是 | 初始化或修复项目记忆结构。 |
| `$project-memory status` | 否 | 查看当前阶段、问题、风险和下一步行动。 |
| `$project-memory close [重点]` | 是 | 记录本次会话并刷新当前状态。 |
| `$project-memory import` | 否 | 预览可导入的历史任务。 |
| `$project-memory import --apply` | 是 | 导入当前对话中已经确认的任务范围。 |
| `$project-memory` | 视情况而定 | 结构不存在时执行 `init`；已存在时执行 `close`。 |

### 6. 数据与授权

- 项目记忆保存在当前项目的 `docs/project-memory/` 中。
- Skill 会要求移除密码、令牌、私钥、Cookie、授权请求头和敏感个人信息。
- 历史导入默认先预览、再确认、后写入，并通过来源任务 ID 防止重复导入。
- 如果项目历史包含公司机密或个人数据，在公开仓库前仍应人工复查生成的记忆文件。

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

The skill instructs Codex to redact passwords, tokens, private keys, cookies, authorization headers, and sensitive personal data from project-memory files.

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
