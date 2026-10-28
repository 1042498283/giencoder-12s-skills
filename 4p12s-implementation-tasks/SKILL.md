---
name: "4p12s-implementation-tasks"
display_name: "4p12s-implementation-tasks"
description: "第7步 — 拆分开发任务：基于技术设计和验证计划，拆分为带依赖关系的子任务。"
version: "1.0.2"
---

# 开发任务计划

**Announce at start:** "I'm using the 4p12s-implementation-tasks skill to create the development task plan."

## 概述

把已确认的产品范围和技术设计转成 `tasks/` 下的开发计划。每个 Task 都是可执行的小计划，步骤使用 Markdown checkbox，执行者可以按步骤 TDD 完成。

本阶段只拆开发任务。不拆 E2E 测试代码任务，不写验证执行任务，不写发布任务。

## 双模运行

### 编排模式

路径包必须包含：

```yaml
repo_root:
delivery_dir:
state_path:
project_entry_path:
requirements_register_path:
prd_path:
user_stories_path:
design_path:
output_paths:
allowed_write_paths:    # 仅 tasks/ 目录
```

### 独立模式

自动发现：
- 当前目录或指定目录下的 `requirements-register.md`、`prd.md`、`user-stories.md`、`design.md`
- 项目入口文档、编码规范、测试规范
- 输出路径：与上游产物同目录的 `tasks/`

独立模式下需要聚焦读取与本次设计相关的源码和测试目录。

## 输出

```text
tasks/
├── index.md
├── TASK-001.md
└── TASK-xxx.md
```

不得写入：`delivery-state.md`、`verification-result.md`、`deploy-log.md`、产品代码、测试代码。

## 拆分原则

每个 Task 应对应一个可独立 TDD 完成的产品行为、服务能力、页面行为、接口能力或技术支撑变更。

每个 Task 必须包含：
- 清晰目标
- 具体文件清单
- 失败测试步骤
- 红灯验证命令
- 实现步骤
- 绿灯验证命令
- 结果记录区

拆分规则：
- 按用户故事、验收标准、设计模块和实际写集聚合
- 同一业务行为涉及 DTO、类型、路由、配置、页面、服务编排时，合并到直接使用它们的 Task
- 不生成独立 DTO、类型定义、路由、配置或纯文件改造 Task
- 不生成 "后续补测试" Task
- 不生成 E2E 测试代码 Task
- Task ID 就是执行顺序；后面的 Task 可以依赖前面的，前面的不得依赖后面的

## tasks/index.md 格式

```markdown
# <需求名> 开发任务索引

## 输入依据

| Artifact | Path |
| --- | --- |

## 目标与范围

- 目标：
- 范围内：
- 范围外：

## Task 目录清单

| Task ID | Task 文件 | 目标 | 关联用户故事/验收标准 | 依赖 Task |
| --- | --- | --- | --- | --- |

## 开放问题

| 问题 | 影响 | 是否阻断开发 | 下一步动作 |
| --- | --- | --- | --- |
```

## TASK-xxx.md 格式

```markdown
# TASK-001 <任务名>

## 目标

- 说明：
- 关联需求 ID：
- 关联用户故事/验收标准：
- 依赖 Task：

## 文件计划

- Create:
- Modify:
- Test:

## TDD 步骤

- [ ] Step 1: 编写失败测试
  - 目标测试文件：
  - 测试意图：
  - 关键断言：

- [ ] Step 2: 运行红灯验证
  - cwd:
  - command:
  - 期望结果：失败，且失败原因指向目标能力缺失
  - 证据：

- [ ] Step 3: 实现最小代码
  - 变更文件：
  - 实现说明：

- [ ] Step 4: 运行绿灯验证
  - cwd:
  - command:
  - 期望结果：通过
  - 证据：

## 结果记录

- changed_files:
- tests_added_or_changed:
- commands_run:
- tdd_red_result:
- tdd_green_result:
- additional_checks:
- requirement_ids_covered:
- user_stories_covered:
- remaining_risks:
```

## 质量门禁

返回前验证：

- `tasks/index.md` 已生成
- 每个 Task 文件已生成
- 每个 Task 在 `Task 目录清单` 中恰好出现一次
- Task 顺序和依赖方向合法
- 每个 Task 都包含失败测试、红灯验证、实现、绿灯验证四个基础步骤
- 每个 Task 都有明确文件计划
- 每个 Task 的红灯/绿灯命令来自项目规范或聚焦配置查找
- 没有 E2E 测试代码任务、集成测试执行任务、Git 任务或发布任务

## STEP_RESULT（编排模式）

```yaml
step: tasks
status: running | passed | blocked | failed
output_files:
  - <delivery_dir>/tasks/
required_inputs_checked:
  - <delivery_dir>/requirements-register.md
  - <delivery_dir>/prd.md
  - <delivery_dir>/user-stories.md
  - <delivery_dir>/design.md
  - <project_entry_path>
commands_run: none
gate_evidence:
blocking_reason:
next_action:
```

## 边界

- 不实现代码
- 不执行测试
- 不做 Git 或发布

## 更新 delivery-state.md（仅独立运行模式）

> **编排模式**（被 `4p12s-delivery-orchestrator` 调用）下，`delivery-state.md` 由 orchestrator 统一处理，本 Skill **不直接更新**。

仅在**独立运行模式**（用户直接调用本 Skill）下，本 Step 完成后（或 `failed` / `blocked` 时）必须更新 `<state_path>`：

1. `## Step Checklist` 表中当前 Step 对应行的 `Status` 改为 `passed` / `failed` / `blocked`；`Evidence` 列填入产物文件路径（多文件用 `；` 分隔）
2. 顶部字段：`current_step` 设为下一 Step（`passed` 时）或保持当前 Step（`failed` / `blocked` 时）；`next_action` 设为对下一/当前 Step 的提示；`blocked` / `failed` 时填写 `blocking_reason`；`updated_at` 改为当前 ISO8601 时间戳
3. 不修改其他 Step / Task 的状态
