---
name: "4p12s-implementation-execution"
display_name: "4p12s-implementation-execution"
description: "第8步 — 执行开发任务：严格按顺序逐个执行任务，编写代码和测试。"
version: "1.0.4"
---

# 开发任务执行

**Announce at start:** "I'm using the 4p12s-implementation-execution skill to execute the development task."

## 概述

执行选定的开发 Task。按 TDD 四步骤逐项完成：写失败测试 → 红灯验证 → 实现 → 绿灯验证。

本阶段只做开发任务。不写 E2E 测试，不运行最终集成测试和 E2E 测试，不提交、推送或发布。

## 核心原则：自动连续执行

**关键心智模型**：

1. **Task 不是交付点**：每个 Task 只是中间步骤，不是独立交付物。不要在每个 Task 完成后等待用户确认。
2. **默认连续执行**：除非遇到 `failed` / `blocked`，否则必须自动继续执行下一个 Task，不需要用户提醒。
3. **总结≠停止**：可以在每个 Task 完成后简要记录，但记录后必须立即开始下一个 Task，不要等待用户指示。
4. **只在阻塞时停止**：只有以下情况才停下来等待用户：
   - TDD 绿灯验证真实失败（`failed`）
   - 遇到外部依赖缺失或环境问题（`blocked`）
   - 需要用户决策的关键问题（如架构选择、需求歧义）

**反模式（必须避免）**：

- ❌ 每个 Task 完成后输出详细总结并等待用户确认
- ❌ 说"TASK-XXX 已完成，接下来需要执行 TASK-YYY，请确认"之类的话
- ❌ 把编译警告、lint 提示当作 `failed` 停下来
- ❌ 认为用户没有明确指示就不能继续
- ❌ 因为判断某个 Task"不适合 TDD"或"太简单/太基础"而跳过它，先去执行后面的 Task
- ❌ 自行重排任务顺序（按依赖拓扑、按 TDD 适配度、按业务重要性挑选起点）

**正确模式**：

- ✅ Task 完成后，简单标记状态，立即读取下一个 Task 文件并继续
- ✅ 只在真正需要用户决策时才停下来
- ✅ 把所有 Task 当作一个整体任务，而不是多个独立小任务
- ✅ 如果不确定是否继续，默认选择继续

## 执行顺序约束（强制）

**必须严格按照 `tasks/index.md` 的 `Task 目录清单` 表格行序，从第一个未完成的 Task 开始逐个执行。禁止跳选、禁止重排。**

### 起点确定

1. 读取 `tasks/index.md`，按 `Task 目录清单` 表格的**行序**（TASK-001 在前、TASK-020 在后）逐行检查每个 Task 的完成状态。
2. 第一个"未完成"的 Task 就是当前必须执行的 Task，无论它看起来是否适合 TDD、是否是脚手架/SQL/配置类任务。
3. 禁止以"该任务不适合 TDD""先做更有价值的任务"等理由跳过行序靠前的未完成 Task 去执行后面的 Task。

### "未完成"的判定标准

一个 Task 视为**已完成**，当且仅当同时满足：
- 其 Task 文件中所有 Step checkbox 均为 `- [x]`
- 其 `## 结果记录` 中已有 `tdd_green_result` 或等价的验证命令证据

其余情况一律视为**未完成**：Task 文件缺失、checkbox 仍为 `- [ ]`、结果记录为空模板，都算未完成。Task 文件缺失时必须返回 `blocked`，不得跳过该 Task 继续执行后面的 Task。

### 非 TDD 形态任务的处理

脚手架搭建、SQL 脚本、配置类等没有单元测试载体的 Task，**同样必须按行序执行，不得跳过**：
- 红灯步骤：以任务文件计划中产物的"缺失"作为红灯证据（如目标文件不存在、命令执行失败），在证据中注明 `expected_failure_reason: 目标产物缺失`
- 绿灯步骤：执行该 Task 文件中给出的验证命令（build、lint、prisma generate、SQL 执行、目录结构检查等），命令成功即绿灯通过
- 若 Task 文件没有给出任何可执行的验证命令且无法从项目规范中推导出验证方式，返回 `blocked`，在 `blocking_reason` 中说明，**禁止跳过**

### 依赖 Task 的处理

当前 Task 依赖的前置 Task 未完成时，只允许校验其产物/证据是否存在，**不得越过前置 Task 直接执行当前 Task**；前置 Task 未完成则当前 Task 返回 `blocked`。

## 双模运行

### 编排模式

被 `4p12s-delivery-orchestrator` 调用时，路径包必须包含：

```yaml
repo_root:
delivery_dir:
state_path:                # delivery-state.md 路径
project_entry_path:
tasks_index_path:
selected_task_path:        # 当前选定的 TASK-xxx.md
current_task:              # 当前 Task ID
dependency_task_paths:     # 已 passed 的依赖 Task 文件路径列表
requirements_register_path:
prd_path:
user_stories_path:
design_path:
output_paths:
allowed_write_paths:       # 当前 Task 文件、该 Task 计划中的产品代码和测试代码
```

**注意**：编排模式下，`delivery-state.md` 由 orchestrator 统一更新，本 Skill **不直接更新**该文件。

### 独立模式

用户直接调用本 Skill 时，自动发现：
- 当前目录或指定目录下的 `tasks/index.md` 和 `tasks/TASK-xxx.md`
- 上游产物（requirements-register.md、prd.md、user-stories.md、design.md）
- 项目入口文档和相关源码
- 依赖 Task：从 `tasks/index.md` 中读取已标记为 `passed` 的 Task，读取其 `## 结果记录` 作为依赖参考

执行策略：
- **默认模式**：按 `tasks/index.md` `Task 目录清单` 表格行序依次执行所有未完成的 Task（"未完成"判定见《执行顺序约束》）。
  - **起点**：必须从表格行序中第一个未完成的 Task 开始，禁止从中间某个 Task 起步（用户显式指定 Task ID 时除外）
  - **继续条件**：当前 Task 返回 `passed` 后，自动继续执行下一个未完成的 Task
  - **停止条件**：仅当以下情况之一发生时才停止：
    1. 任一 Task 返回 `failed` 或 `blocked`（立即停止）
    2. 所有未完成 Task 都已执行完毕并返回 `passed`
  - **重要**：编译警告、lint 提示、非关键检查失败不等同于 `failed`，只有 TDD 绿灯验证真实失败才标记为 `failed`
- **指定 Task ID**：用户指定 Task ID 时，仅执行该 Task。

**注意**：独立模式下，技能执行结束后必须更新 `<state_path>`（`delivery-state.md`）。

## 执行边界

只允许：
- 执行选定的 Task（一个或多个，取决于执行模式）
- 修改当前 Task 文件计划中的产品代码和测试代码
- 按 TDD 写失败测试、运行红灯、实现、运行绿灯
- 更新当前 `TASK-xxx.md` 的 checkbox、证据和结果记录
- 返回 STEP_RESULT

不得：
- 改写 `tasks/index.md`
- 写 E2E 测试代码
- 运行最终集成测试和 E2E 测试
- 提交、推送、发布
- 发现计划不合适时擅自重拆任务
- 跳过行序靠前的未完成 Task 去执行后面的 Task

**重要说明**：
- `verification-result.md` 由后续的 verification 阶段 Skill 编写，本 Skill 不写该文件
- Task 文件的 `## 结果记录` 是主要证据源
- `delivery-state.md` 的更新规则：编排模式下由 orchestrator 统一更新；独立模式下由本 Skill 在执行结束后更新，仅记录状态和证据文件路径

## 步骤状态

Task 文件中的步骤状态只使用 Markdown checkbox：

```markdown
- [ ] Step N: <步骤名>
- [x] Step N: <步骤名>
```

执行规则：
- 每完成一个步骤，必须立即把该 Step 从 `- [ ]` 改为 `- [x]`，并在 Task 文件的 `## 结果记录` 中写入证据
- 每次更新 checkbox 后，必须重新读取 Task 文件确认落盘
- 返回 `passed` 前，必须最后读取 Task 文件确认没有 `- [ ] Step`
- 只要还有任何 TDD 步骤不是 `- [x]`，不得返回 `passed`

## 执行流程

```text
读取 tasks/index.md
-> 按 Task 目录清单表格行序逐行判定完成状态，定位第一个未完成的 Task 作为当前 Task（用户显式指定 Task ID 时除外）
-> 读取当前 TASK-xxx.md
-> 读取 dependency_task_paths 中每个依赖 Task 的 ## 结果记录（编排模式）或按 checkbox 与结果记录判定依赖 Task 已完成并读取其 ## 结果记录（独立模式）
-> 校验 Task 与目录清单一致
-> 审查 TDD 步骤和文件计划是否完整
-> 读取当前 Task 直接相关代码和测试
-> Step 1 编写失败测试
-> 将 Step 1 checkbox 改为 - [x]，在 Task 文件的 ## 结果记录中写入证据，并重新读取确认落盘
-> Step 2 运行红灯验证，确认失败原因指向目标能力缺失
-> 将 Step 2 checkbox 改为 - [x]，在 Task 文件的 ## 结果记录中写入证据，并重新读取确认落盘
-> Step 3 实现最小产品代码
-> 将 Step 3 checkbox 改为 - [x]，在 Task 文件的 ## 结果记录中写入证据，并重新读取确认落盘
-> Step 4 运行绿灯验证
-> 将 Step 4 checkbox 改为 - [x]，在 Task 文件的 ## 结果记录中写入证据，并重新读取确认落盘
-> 必要时运行 lint/typecheck/build 等检查
-> 最后一次重新读取 Task 文件，自检所有 checkbox
-> 返回 STEP_RESULT
```

不要跳过红灯。编译、lint、类型检查不能作为红灯验证。

## Task 迭代器

执行多个 Task 时的迭代逻辑：

```text
读取 tasks/index.md
-> 获取所有未完成 Task 列表（严格按 Task 目录清单表格行序，不得按依赖拓扑或 TDD 适配度重排）
-> 对于每个 Task：
   -> 读取 Task 文件
   -> 执行 TDD 流程（Step 1-4）
   -> 更新 Task 文件的 checkbox 和结果记录
   -> 返回 STEP_RESULT
   -> 如果 status == passed 且还有下一个 Task：
      -> 继续执行下一个 Task
   -> 如果 status == failed 或 blocked：
      -> 停止执行，返回当前状态
   -> 如果 status == passed 且没有下一个 Task：
      -> 执行结束
```

**关键点**：
- 每个 Task 的 `passed` 状态是继续执行下一个 Task 的唯一条件
- 不要在单个 Task 执行过程中停止（除非遇到 `failed` / `blocked`）
- 执行完一个 Task 后，模型应自动读取下一个 Task 文件并继续

## TDD 要求

必须完成：
1. 编写失败测试
2. 运行红灯命令，确认失败原因指向目标能力缺失
3. 实现最小产品代码和必要支撑代码
4. 运行绿灯命令，确认通过

证据要求：
- `tdd_red_result` 记录 `cwd`、`command`、`exit_code`、`result`、`output_summary`、`expected_failure_reason`
- `tdd_green_result` 记录 `cwd`、`command`、`exit_code`、`result`、`output_summary`
- `commands_run` 中的每条命令都有结构化证据
- `changed_files` 和 `tests_added_or_changed` 中列出的路径必须存在

## 结果记录

必须更新选定 Task 的 `## 结果记录`（在 Task 文件中）：

```yaml
changed_files:
tests_added_or_changed:
commands_run:
  - cwd:
    command:
    exit_code:
    result:
    output_summary:
tdd_red_result:
tdd_green_result:
additional_checks:
requirement_ids_covered:
user_stories_covered:
remaining_risks:
```

**更新时机**：每个步骤完成后立即更新该步骤的证据，所有步骤完成后更新汇总信息。

命令证据必须结构化。不能只写 `PASS`、`FAIL`、`通过`、`失败`。

## 通过条件

只有同时满足以下条件，才能返回 `passed`：

- 当前 Task 的所有 TDD checkbox 都是 `- [x]`
- 已重新读取确认没有 `- [ ] Step`
- 每个完成步骤都有证据（记录在 Task 文件的 `## 结果记录` 中）
- 红灯验证真实执行且失败原因指向目标能力缺失
- 绿灯验证真实执行且通过
- 结果记录完整
- 当前 Task 的关联需求和用户故事在结果记录中可追踪

## STEP_RESULT

```yaml
step: implementation
status: running | passed | blocked | failed
output_files:
  - <selected_task_path>
required_inputs_checked:
  - <tasks_index_path>          # 编排模式；独立模式自动发现
  - <selected_task_path>        # 当前执行的 Task 文件
  - <requirements_register_path> # 编排模式；独立模式自动发现
  - <prd_path>                  # 编排模式；独立模式自动发现
  - <user_stories_path>         # 编排模式；独立模式自动发现
  - <design_path>               # 编排模式；独立模式自动发现
  - <project_entry_path>        # 项目入口文件
commands_run:
gate_evidence:
  checkbox_audit:
    - step:
      status: "- [x]"
      evidence:
blocking_reason:
next_action:
```

`gate_evidence.checkbox_audit` 必须列出当前 Task 的每个 TDD Step，且每项 status 都必须是 `- [x]`。

**状态流转规则**：
- `running` → `passed`：当前 Task 所有 TDD 步骤完成且绿灯验证通过 → **继续执行下一个 Task**
- `running` → `failed`：绿灯验证真实失败 → **停止执行**
- `running` → `blocked`：遇到外部依赖缺失或环境问题 → **停止执行**
- `passed`：所有未完成 Task 都已执行完毕 → **执行结束**

**多 Task 执行时的返回策略**：
- 每个 Task 完成后立即返回该 Task 的 STEP_RESULT
- 如果还有下一个 Task，模型应自动开始执行，不停止
- 只有遇到 `failed` / `blocked` 或所有 Task 完成时才停止

## 更新 delivery-state.md（仅独立模式）

> **编排模式**下，`delivery-state.md` 由 orchestrator 统一处理，本 Skill **不直接更新**。

仅在**独立模式**下，技能执行结束后（无论 `passed`、`failed` 或 `blocked`）必须更新 `<state_path>`：

1. **`## Step Checklist` 表**：当前 Step 对应行的 `Status` 改为 `passed` / `failed` / `blocked`；`Evidence` 列填入产物文件路径（多文件用 `；` 分隔）
2. **顶部字段**：
   - `passed` 时：`current_step` 设为下一 Step，`next_action` 设为对下一 Step 的提示
   - `failed` / `blocked` 时：`current_step` 保持当前 Step，填写 `blocking_reason`，`next_action` 设为对当前 Step 的提示
   - `updated_at` 改为当前 ISO8601 时间戳
3. 不修改其他 Step / Task 的状态
