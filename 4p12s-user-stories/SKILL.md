---
name: "4p12s-user-stories"
display_name: "4p12s-user-stories"
description: "第4步 — 生成用户故事：基于原始需求和 PRD，拆分为用户故事文档。"
version: "1.0.2"
---

# 用户故事

**Announce at start:** "I'm using the 4p12s-user-stories skill to generate user stories and acceptance scenarios."

## 概述

从 `requirements-register.md` 和 `prd.md` 生成 `user-stories.md`，将需求转换为可验收的用户故事和 Given/When/Then 场景。

## 双模运行

### 编排模式

路径包必须包含：

```yaml
delivery_dir:
state_path:
project_entry_path:
requirements_register_path:
prd_path:
user_stories_path:      # 已存在的 user-stories.md（更新场景）
output_paths:
allowed_write_paths:    # 仅 user-stories.md
```

### 独立模式

自动发现：
- 当前目录或指定目录下的 `requirements-register.md` 和 `prd.md`
- 项目入口文档
- 输出路径：与上游产物同目录的 `user-stories.md`

如果找不到上游产物，提示用户先运行前置 Skill。

## 输出文件

写入唯一产物：`user-stories.md`

```markdown
# <需求名> 用户故事

## 输入依据

| Artifact | Path |
| --- | --- |

## 场景总览

| 用户故事 ID | 验收场景 ID | 场景用例 ID | 来源需求 ID | PRD 验收标准 | 角色 | 目标 | 是否产品行为变更 | 最低验证层级 | 是否提交前自动化验证 | 是否阻断 Git | 说明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

## 用户故事

### US-001 <故事标题>

- 来源需求 ID：
- 角色：
- 目标：
- 价值：
- 范围内：
- 范围外：

#### AC-001 <验收场景标题>

- 场景用例 ID：SC-001
- 来源需求 ID：
- 来源 PRD 验收标准：
- 是否产品行为变更：是 / 否
- 最低验证层级：unit / component / service_api / service_integration / cross_boundary_e2e
- 是否允许降级验证：是 / 否
- 降级理由：
- 是否提交前自动化验证：是 / 否
- 是否阻断 Git：是 / 否
- 暂不自动化验证的原因：
- 风险判断：
- 后续处理结论：

```gherkin
Given ...
When ...
Then ...
And ...
```

## 缺口与阻断

| 项目 | 关联需求 ID | 影响 | 下一步动作 |
| --- | --- | --- | --- |
```

## 映射规则

- 范围内的每个需求 ID 必须出现在至少一个 `US-xxx` 中
- PRD 的每个验收标准必须映射到至少一个 `AC-xxx` / `SC-xxx`
- `US-xxx`、`AC-xxx`、`SC-xxx` 三类 ID 由 `user-stories.md` 拥有，是下游的验收入口
- PRD 验收标准是输入依据，下游验收场景 ID 由本文件重新分配
- 如果需要的验收场景没有对应的 PRD 验收标准，返回 `blocked` 而非凭空创建
- 验收场景以黑盒优先：先描述外部可观察行为，再描述内部实现
- 每个 `AC-xxx` 必须包含完整 Given/When/Then
- 主流程、异常流程、边界条件、权限、数据规则、集成点、状态流转和失败路径必须在相关时体现
- 范围内的前端或用户交互验收默认为 `cross_boundary_e2e`，必须标记为 Git 阻断
- 不要引入上游产物中不存在的需求
- 不要使用澄清问题 ID 作为独立源需求 ID

## 质量门禁

返回前验证：

- `user-stories.md` 已创建或更新
- 文件包含 `输入依据`、`场景总览`、`用户故事`、`缺口与阻断`
- 每个范围内需求 ID 都映射到至少一个 `US-xxx`
- PRD 的每个验收标准都映射到至少一个 `AC-xxx` / `SC-xxx`
- 每个 `US-xxx` 都包含角色、目标、价值、范围内和范围外
- 每个 `AC-xxx` 都包含完整 Given/When/Then 和 Git 阻断决策
- 没有创建脱离 PRD 验收标准的下游专属场景
- 范围内的每个前端交互都有 E2E 或跨边界自动化验证路径

## STEP_RESULT（编排模式）

```yaml
step: user_stories
status: running | passed | blocked | failed
output_files:
  - <delivery_dir>/user-stories.md
required_inputs_checked:
  - <delivery_dir>/requirements-register.md
  - <delivery_dir>/prd.md
  - <project_entry_path>
commands_run: none
gate_evidence:
blocking_reason:
next_action:
```

## 边界

- 不创建技术设计、验证计划、任务、代码、提交或部署产物
- 不决定流程终点状态

## 更新 delivery-state.md（仅独立运行模式）

> **编排模式**（被 `4p12s-delivery-orchestrator` 调用）下，`delivery-state.md` 由 orchestrator 统一处理，本 Skill **不直接更新**。

仅在**独立运行模式**（用户直接调用本 Skill）下，本 Step 完成后（或 `failed` / `blocked` 时）必须更新 `<state_path>`：

1. `## Step Checklist` 表中当前 Step 对应行的 `Status` 改为 `passed` / `failed` / `blocked`；`Evidence` 列填入产物文件路径（多文件用 `；` 分隔）
2. 顶部字段：`current_step` 设为下一 Step（`passed` 时）或保持当前 Step（`failed` / `blocked` 时）；`next_action` 设为对下一/当前 Step 的提示；`blocked` / `failed` 时填写 `blocking_reason`；`updated_at` 改为当前 ISO8601 时间戳
3. 不修改其他 Step / Task 的状态
