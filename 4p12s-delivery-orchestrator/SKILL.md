---
name: "4p12s-delivery-orchestrator"
display_name: "4p12s-delivery-orchestrator"
description: "第1步 — 端到端流程初始化：用户输入业务需求，生成交付状态跟踪表，启动整个流程。"
version: "1.0.3"
---

# 端到端交付主控流程 - 初始化器

**Announce at start:** "I'm using the 4p12s-delivery-orchestrator skill to initialize the delivery state file."

## 概述

本 Skill 是端到端交付系统的**初始化器**。它**只做一件事**：在指定交付目录下创建并初始化 `delivery-state.md` 状态账本。

**明确不执行：**

- 不调用任何子 Skill
- 不执行 4p12s-requirements、4p12s-prd、4p12s-user-stories、4p12s-technical-design、4p12s-verification-plan、4p12s-implementation-tasks、4p12s-implementation-execution、4p12s-integration-test、4p12s-e2e-test、4p12s-git-push、4p12s-deployment-execution 等任何 Step
- 不执行自动推进、不构造路径包、不复核产物
- 不进行任何验证、修复或回退

后续 Step 由用户或其它调度流程按需触发。

## 初始化流程

当用户触发此 Skill 时：

1. 确认项目路径（`repo_root`）和业务需求名称（`delivery_name`）
2. 创建交付目录 `deliveries/<需求名>/`（如不存在）
3. 在该目录下创建并写入 `delivery-state.md`
4. 输出初始化结果（交付目录路径 + delivery-state.md 路径）

完成后立即结束本 Skill，不进入任何后续 Step。

## 交付目录

```text
deliveries/<需求名>/
└── delivery-state.md
```

仅创建 `delivery-state.md` 一个文件，其它产物文件由对应的子 Skill 在后续 Step 中按需生成。

## delivery-state.md 初始结构

```markdown
# 交付状态

- workflow: end_to_end_delivery
- delivery_name: <需求名>
- workflow_status: pending
- current_step: requirements
- current_task:
- blocking_reason:
- next_action: 等待用户或编排器触发 4p12s-requirements Skill
- updated_at: <ISO8601 时间戳，格式示例：2024-01-15T10:30:00+08:00>

## Step Checklist

| Step | Artifact | Status | Evidence |
| --- | --- | --- | --- |
| requirements | requirements-register.md | pending | |
| prd | prd.md | pending | |
| user_stories | user-stories.md | pending | |
| design | design.md | pending | |
| verification_plan | verification-plan.md | pending | |
| tasks | tasks/ | pending | |
| implementation | tasks/ | pending | |
| verification_integration | verification-result.md（集成部分） | pending | |
| verification_e2e | verification-result.md（E2E 部分） | pending | |
| git_push | deploy-log.md##Git Push | pending | |
| release_trigger | deploy-log.md##Release Trigger | pending | |

**注意：** Evidence 列填入产物文件路径，多文件时用英文分号 `;` 分隔。

## Task Checklist

| Task ID | Task File | Status | Linked Story/Acceptance | Evidence |
| --- | --- | --- | --- | --- |
```

**说明：** Task Checklist 由 `4p12s-implementation-tasks` Skill 在 tasks 阶段填充，初始化时为空。

## 合法状态

```text
Step/Task: pending | running | passed | blocked | failed
workflow_status: pending | running | blocked | failed | release_triggered
```

不得使用 `todo`、`done`、`complete`、`success` 等状态别名。

**状态说明：**
- `pending`：尚未开始
- `running`：正在执行
- `passed`：已完成且验证通过
- `blocked`：被外部因素阻断（如等待用户输入、业务决策）
- `failed`：执行失败（包括业务逻辑失败和系统级错误）
- `release_triggered`：发布已触发（workflow_status 专用）

## 初始化输出

初始化完成后，直接声明：本 Skill 已结束，后续 Step 需要由调用方或其它流程触发，不在本 Skill 范围内执行。

## 禁止事项

- 调用任何子 Skill
- 执行 4p12s-requirements、4p12s-prd、4p12s-user-stories、4p12s-technical-design、4p12s-verification-plan、4p12s-implementation-tasks、4p12s-implementation-execution、4p12s-integration-test、4p12s-e2e-test、4p12s-git-push、4p12s-deployment-execution 等 Step
- 构造路径包、注入项目上下文、扫描约定文档
- 自动推进、复核产物、修复、回退
- 修改或创建 `delivery-state.md` 之外的任何文件
- 在交付目录中预先创建 requirements-register.md、prd.md、design.md 等后续产物
