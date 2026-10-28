---
name: "4p12s-prd"
display_name: "4p12s-prd"
description: "第3步 — 生成产品需求文档：基于原始需求登记表，生成结构化的 PRD 产品需求文档。"
version: "1.0.2"
---

# 生成 PRD

**Announce at start:** "I'm using the 4p12s-prd skill to generate the Product Requirements Document."

## 概述

从已登记的业务需求生成 `prd.md`。保留业务含义，不转为技术设计。

## 双模运行

### 编排模式

路径包必须包含：

```yaml
delivery_dir:
state_path:
project_entry_path:
requirements_register_path:
prd_path:               # 已存在的 prd.md 路径（更新场景）
output_paths:
allowed_write_paths:    # 仅 prd.md
```

### 独立模式

自动发现：
- 当前目录或指定目录下的 `requirements-register.md`
- 项目入口文档
- 输出路径：与 `requirements-register.md` 同目录的 `prd.md`

如果找不到 `requirements-register.md`，提示用户先运行 `/4p12s-requirements`。

## 输出文件

写入唯一产物：`prd.md`

```markdown
# <需求名> PRD

## 1. 背景与目标

- 业务背景：
- 目标：
- 成功标准：
- 关联需求 ID：

## 2. 用户角色与场景

| 角色 | 场景 | 目标 | 关联需求 ID |
| --- | --- | --- | --- |

## 3. 功能范围

### 范围内

| 功能 | 说明 | 关联需求 ID |
| --- | --- | --- |

### 范围外

| 项目 | 原因 | 关联需求 ID |
| --- | --- | --- |

## 4. 业务流程

### 主流程
### 异常流程
### 边界流程

## 5. 业务规则

| 规则 | 说明 | 关联需求 ID |
| --- | --- | --- |

## 6. 字段、数据与状态口径

| 项目 | 口径 | 来源 | 关联需求 ID |
| --- | --- | --- | --- |

## 7. 系统边界与外部依赖

| 系统或依赖 | 关系 | 输入/输出 | 风险 | 关联需求 ID |
| --- | --- | --- | --- | --- |

## 8. 非功能要求

| 类型 | 要求 | 验收提示 | 关联需求 ID |
| --- | --- | --- | --- |

## 9. 验收标准

| 验收项 | 通过标准 | 关联需求 ID |
| --- | --- | --- |

## 10. 待确认问题

| 问题 | 关联需求 ID | 是否阻断后续阶段 | 下一步动作 |
| --- | --- | --- | --- |
```

## PRD 规则

- 范围内的每个需求 ID 必须出现在 PRD 中，或在 `范围外` 中附带原因
- 保留 `requirements-register.md` 中的业务含义；不要将其转为技术设计
- 验收标准必须是可观察和可测试的
- 每个范围内的 `UI-*` 需求必须出现在用户角色/场景、功能范围、流程和验收标准中
- 除非 `requirements-register.md` 明确排除前端，否则不要将前端范围移到 `范围外`
- PRD 验收标准是下游验收场景生成的输入依据
- 如果 PRD 生成时发现缺失边界、异常、字段、集成、权限或非功能验收标准，在此处添加并附带源需求 ID
- 保持未解决的非阻断性问题明确
- 不要在没有标记来源的情况下引入新需求
- 不要使用澄清问题 ID 作为唯一链接来源

## 质量门禁

返回前验证：

- 源需求 ID 已覆盖
- 业务目标、范围、流程、规则、字段、依赖和非功能要求在存在上游内容时已体现
- 验收标准足够具体，可以供给验收场景生成
- 每个验收标准都有稳定引用和源需求 ID
- 每个范围内的 `UI-*` 需求都有 PRD 覆盖和至少一个可观察的验收标准
- 没有前端范围被静默移到 `范围外`
- 没有 PRD 阻断性问题仍未解决
- 没有编造不受支持的需求

## STEP_RESULT（编排模式）

```yaml
step: prd
status: running | passed | blocked | failed
output_files:
  - <delivery_dir>/prd.md
required_inputs_checked:
  - <delivery_dir>/requirements-register.md
  - <project_entry_path>
commands_run: none
gate_evidence:
blocking_reason:
next_action:
```

## 边界

- 不创建 `user-stories.md`、`design.md`、`tasks/`、`verification-plan.md`、`verification-result.md` 或 `deploy-log.md`
- 不开始实现、测试、Git 或发布

## 更新 delivery-state.md（仅独立运行模式）

> **编排模式**（被 `4p12s-delivery-orchestrator` 调用）下，`delivery-state.md` 由 orchestrator 统一处理，本 Skill **不直接更新**。

仅在**独立运行模式**（用户直接调用本 Skill）下，本 Step 完成后（或 `failed` / `blocked` 时）必须更新 `<state_path>`：

1. `## Step Checklist` 表中当前 Step 对应行的 `Status` 改为 `passed` / `failed` / `blocked`；`Evidence` 列填入产物文件路径（多文件用 `；` 分隔）
2. 顶部字段：`current_step` 设为下一 Step（`passed` 时）或保持当前 Step（`failed` / `blocked` 时）；`next_action` 设为对下一/当前 Step 的提示；`blocked` / `failed` 时填写 `blocking_reason`；`updated_at` 改为当前 ISO8601 时间戳
3. 不修改其他 Step / Task 的状态
