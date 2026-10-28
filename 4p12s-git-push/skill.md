---
name: "4p12s-git-push"
display_name: "4p12s-git-push"
description: "第11步 — Git 提交推送：测试通过后，将代码 commit 并 push 到远程仓库。"
version: "1.0.5"
---

# Git 推送

**Announce at start:** "I'm using the git-push-4p12s skill to commit and push delivery changes."

## 概述

执行交付流程的 Git 提交和推送步骤。验证通过后，仅暂存交付相关文件、提交、推送、验证远端 HEAD。

## 双模运行

### 编排模式

路径包必须包含：

```yaml
repo_root:
delivery_dir:
state_path:
project_entry_path:
verification_result_path:
tasks_index_path:
tasks_dir_path:
deploy_log_path:        # 已存在的 deploy-log.md（如有）
output_paths:
allowed_write_paths:    # deploy-log.md 和本次交付文件集的 Git 暂存范围
```

### 独立模式

自动发现：
- 当前 Git 仓库根目录
- 项目 Git/提交规范（见下方"提交信息规范"）
- `verification-result.md`、`deploy-log.md`
- 当前 `git status` 和 `git diff`

独立模式执行前会检查 E2E 验证是否通过。

## 前置检查

`verification-result.md` 是必选项。如果缺失、为空、没有 `## E2E 验证结果`、E2E 验证结论不是通过、列出了 Git 阻断项，返回 `blocked`。

`delivery-state.md` 中的 `verification_e2e` Step 必须 `passed`（编排模式下）。

## 提交信息规范（commit-msg）

> **这是本 Skill 的核心约束。** 所有提交信息必须通过项目 husky `commit-msg` hook 校验。

### 格式定义

```
${commit}:${taskId}_${name}：${submitDescription}
```

> ⚠️ 注意：`name` 与 `submitDescription` 之间的冒号是 **中文全角冒号 `：`**，其余均为英文半角。

### 字段说明

| 字段 | 说明 | 取值约束 |
|------|------|----------|
| `commit` | 提交类型 | `Feat`（需求）/ `Task`（任务）/ `BugFix`（缺陷） |
| `taskId` | 任务/需求 ID，不可含下划线 `_` | 如 `33413` |
| `name` | 任务/需求名称，**可含中文冒号 `：`**，贪婪匹配到最后一个 `：` 为分隔符 | 如 `新增harness的使用`、`需求：新增harness` |
| `submitDescription` | 本次提交描述，不可为空 | 如 `add harness L1 integration` |

### 示例

```
Feat:33413_新增harness的使用：add harness L1 integration
Task:33414_代码审查优化：optimize review logic
BugFix:33415_修复空指针：resolve null pointer exception
```

### 组装方式

不需要正则校验。直接从提取的任务信息中填充模板即可，husky `commit-msg` hook 会在 `git commit` 时自动校验格式。

组装时**必须严格遵守**以下规则：

1. **逐字节原样填充**：每个字段值必须与原始数据完全一致，禁止任何修改
2. **禁止插入任何不可见字符和空格**：
   - 半角空格 ` `（0x20）
   - 全角空格（U+3000，字节 `E3 80 80`）
   - 全角字符形式的伪空格（如 `EF BC A0` 等 UTF-8 多字节字符）
   - 零宽空格、零宽连接符等 Unicode 空白字符
   - **以上任何一种都不能出现在字段值内部或字段之间**
3. **name 不做任何转换**：
   - 中文就是中文，英文就是英文，混合就保持混合
   - 不要在中文和英文之间加空格（如 `新增harness` 不能变成 `新增 harness`）
   - 不要做翻译、不要做格式化、不要做分词
4. **submitDescription** 从 `git diff` 总结，不照搬任务 `<desc>`

❌ 错误示例（name 中被插入了全角空格，肉眼难以发现）：
```
Task:33413_新增 harness 的使用：refactor wiki structure
```
原始 title 是 `新增harness的使用`，中间没有空格。插入的全角空格在终端里看起来像普通空格，但 husky hook 会校验失败。

✅ 正确示例：
```
Task:33413_新增harness的使用：refactor wiki structure
```

## 任务信息提取（提交前必须执行）

> 在执行 `git commit` **之前**，必须完成以下任务信息提取流程。

### 步骤 1：搜索上下文中的任务信息

在以下位置查找任务数据：

1. **当前会话上下文（系统提示词注入的 XML）** — 搜索 `<work_task_context source="selected_work_task">` 块：

   **full 模式**（任务描述较短，整体注入）：
   ```xml
   <work_task_context source="selected_work_task">
     <task_info>
       <title>【研发团队】新增harness的使用</title>
       <job_id>JOB-100001</job_id>
       <task_id>33413</task_id>
       <task_type>BUSINESS_REQUIREMENT</task_type>
     </task_info>

     <remark mode="full" total_chars="1234">
       ...任务描述全文...
     </remark>

     <attachments count="0">
       <attachment id="att:1" name="design.md" kind="markdown" url="..." />
     </attachments>
   </work_task_context>
   ```

   **digest 模式**（任务描述过长，仅注入摘要与 chunk 索引）：
   ```xml
   <work_task_context source="selected_work_task">
     <task_info>
       <title>【研发团队】新增harness的使用</title>
       <job_id>JOB-100001</job_id>
       <task_id>33413</task_id>
       <task_type>BUSINESS_REQUIREMENT</task_type>
     </task_info>

     <work_task_digest>
       <summary><item source="...">...</item></summary>
       <scope><item source="...">...</item></scope>
       <requirements><item source="...">...</item></requirements>
       <acceptance_criteria><item source="...">...</item></acceptance_criteria>
       <open_questions><item source="...">...</item></open_questions>
     </work_task_digest>

     <work_task_chunk_index total_chars="80000">
       <chunk id="c1" title="背景与目标" preview="..." />
       <chunk id="c2" title="验收标准" preview="..." />
     </work_task_chunk_index>

     <attachments count="3">...</attachments>
   </work_task_context>
   ```

2. **编排模式的 `tasks/index.md`** — 已通过的 Task 记录
3. **`delivery-state.md`** 中的任务关联字段

提取字段映射：

| 提取字段 | 映射到 commit-msg |
|----------|-------------------|
| `<task_type>`（值为 `TECHNICAL_REQUIREMENT` / `DEFECT` / `BUSINESS_REQUIREMENT`；**映射到 `commit`**：缺陷类（`DEFECT`）→ `BugFix`；需求类（`BUSINESS_REQUIREMENT`）→ `Feat`；任务类（`TECHNICAL_REQUIREMENT`）→ `Task`；缺失默认 `Task`） | `commit` |
| `<task_id>` | `taskId` |
| `<title>`（仅去掉 `【...】` 前缀标签，**保留原文语言不变**，不做翻译） | `name` |
| `git diff --stat` + 暂存文件的变更摘要（**见下方说明**） | `submitDescription` |

> ⚠️ **`submitDescription` 不是任务 `<desc>` 的原文。** 它必须描述**本次 git commit 实际做了什么**，需要结合 `git diff` 内容自行总结。
>
> 任务的 `<desc>` 只是任务的整体目标/背景，而 `submitDescription` 是本次提交的具体变更动作。
>
> 示例：
> - 任务 `<desc>`：`1.全员使用harness L1的版本。2.搜集使用情况，优化harness`
> - ✅ 正确的 `submitDescription`：`add harness L1 integration and config`（本次提交做了什么）
> - ❌ 错误的 `submitDescription`：`1.全员使用harness L1的版本。2.搜集使用情况，优化harness`（直接照搬任务描述）

### 步骤 2：任务信息缺失时的处理

当上下文中 **不存在** `<work_task_context>` 信息时，按以下流程处理：

```
┌─────────────────────────────────────────┐
│  上下文中是否有 <work_task_context>？   │
│                                         │
│  YES ──→ 提取字段，组装 commit-msg      │
│           └──→ 直接进入提交              │
│                                         │
│  NO ───→ 提示用户填写任务信息            │
│           │                             │
│           ├─→ 用户提供了信息             │
│           │    └──→ 按规范组装 commit-msg │
│           │                              │
│           └─→ 用户选择「强制跳过」       │
│                └──→ 使用默认值组装        │
│                    commit-msg             │
└─────────────────────────────────────────┘
```

#### 2a. 提示用户填写

向用户请求以下信息：

```
⚠️ 未检测到 <work_task_context>，请提供以下内容以继续 git push：

1. 任务类型（Feat / Task / BugFix）：
2. 任务 ID（task_id）：
3. 任务名称（name）：
4. 本次提交描述（submitDescription）：

或者输入「跳过」使用默认值强制提交。
```

#### 2b. 用户强制跳过时的默认值

当用户选择跳过时，使用以下默认值：

| 字段 | 默认值 |
|------|--------|
| `commit` | `Task` |
| `taskId` | `0000` |
| `name` | `untitled` |
| `submitDescription` | 基于 `git diff --stat` 自动生成的简要描述 |

最终生成的 commit-msg 形如：
```
Task:0000_untitled：update files per git diff stat
```

> ⚠️ 即使用户强制跳过，最终 commit-msg **仍然必须**符合 commit-msg hook 的正则格式，否则提交会被 husky 拒绝。

## 范围选择

从以下内容确定本次交付的文件集：
- 交付产物和当前交付目录
- 已通过 Task 记录：`changed_files`、`tests_added_or_changed`
- `verification-result.md##E2E 测试代码` 记录的目标测试文件和修改文件
- `git status` 和聚焦的 `git diff`

仅暂存属于本次交付的明确路径。保留无关变更。如果交付文件集无法与无关变更分离，返回 `blocked`。

## 执行流程

1. 运行 `git status` 和聚焦的 `git diff`
2. 确定本次交付的明确文件列表
3. 仅暂存这些明确路径
4. **【必须】提取任务信息**（见"任务信息提取"章节）
5. 按 **commit-msg 格式模板** 直接填充组装：`${commit}:${taskId}_${name}：${submitDescription}`
6. 执行 `git commit -m "<组装好的 commit-msg>"`
7. 推送当前分支
8. 验证远端分支 HEAD 等于本地提交哈希
9. 向 `deploy-log.md` 追加一条新的 Git Push 记录（保留历史记录，见"deploy-log.md##Git Push"写入规则）
10. 返回 `STEP_RESULT`

进入本步骤后，除任务信息缺失外不要向用户请求其他确认。仅在分支不明确、提交规范不明确、远端权限失败、推送失败或无法分离无关变更时阻断。

不要创建或切换分支。不要创建、打开或等待 Pull Request / Merge Request。

## deploy-log.md##Git Push

### 写入规则（追加，不覆盖）

- `deploy-log.md` 不存在：创建文件，写入标题 `# Deploy Log` 和 `## Git Push` 章节，再追加本次记录。
- `deploy-log.md` 存在但没有 `## Git Push` 章节：在文件末尾追加 `## Git Push` 章节，再追加本次记录。
- `deploy-log.md` 已有 `## Git Push` 章节：先完整读取文件现有内容，把本次记录追加到该章节末尾（下一个二级标题之前；无下一个二级标题时为文件末尾）。
- 每次执行本 Skill 只追加一条新记录；严禁删除、覆盖或重排历史 Git Push 记录和其他章节（如 `## Release Trigger`）。
- 记录编号自增：`record_no` = 现有 Git Push 记录条数 + 1，首条为 1。

### 记录模板

```markdown
### Git Push #<record_no>

- record_no:
- push_time:              # ISO8601 时间戳，如 2026-08-12T10:30:00+08:00
- commit_type:            # Feat | Task | BugFix
- task_id:
- task_name:
- commit_hash:
- branch:
- remote:
- staged_files:
- commit_message:         # 完整 commit-msg，含中文冒号
- commit_command:
- commit_exit_code:
- push_command:
- push_exit_code:
- remote_head_check_command:
- remote_head:
- result:
```

`result` 必须说明推送证据是否完整。它不是发布证据。

不要将 MR/PR URL 记录为必需的 Git Push 证据。

## STEP_RESULT（编排模式）

```yaml
step: git_push
status: running | passed | blocked | failed
output_files:
  - <delivery_dir>/deploy-log.md
required_inputs_checked:
  - <delivery_dir>/delivery-state.md
  - <delivery_dir>/verification-result.md
  - <delivery_dir>/tasks/index.md
  - <project_entry_path>
task_info:
  source: context_xml | user_input | force_skip_default
  commit_type:
  task_id:
  task_name:
  submit_description:
commands_run:
gate_evidence:
blocking_reason:
next_action:
```

仅当提交哈希、推送命令结果、远端分支、远端 HEAD 一致性、已暂存文件列表和 `deploy-log.md##Git Push` 证据全部完整时才使用 `passed`。

## 边界

- 不触发发布或部署 API
- 不写入 `deploy-log.md##Release Trigger`
- 不覆盖、删除或改写 `deploy-log.md` 中的历史记录与其他章节，仅追加本次记录
- 不创建或切换分支
- 不创建、打开、准备或等待 Pull Request / Merge Request
- 不决定流程终点状态

## 更新 delivery-state.md（仅独立运行模式）

> **编排模式**（被 `4p12s-delivery-orchestrator` 调用）下，`delivery-state.md` 由 orchestrator 统一处理，本 Skill **不直接更新**。

仅在**独立运行模式**（用户直接调用本 Skill）下，本 Step 完成后（或 `failed` / `blocked` 时）必须更新 `<state_path>`：

1. `## Step Checklist` 表中当前 Step 对应行的 `Status` 改为 `passed` / `failed` / `blocked`；`Evidence` 列填入产物文件路径（多文件用 `；` 分隔）
2. 顶部字段：`current_step` 设为下一 Step（`passed` 时）或保持当前 Step（`failed` / `blocked` 时）；`next_action` 设为对下一/当前 Step 的提示；`blocked` / `failed` 时填写 `blocking_reason`；`updated_at` 改为当前 ISO8601 时间戳
3. 不修改其他 Step / Task 的状态
