---
name: "4p12s-technical-design"
display_name: "4p12s-technical-design"
description: "第5步 — 生成技术设计文档：基于需求层全部产物，设计技术架构、接口和数据模型。"
version: "1.0.2"
---

# 技术设计

**Announce at start:** "I'm using the 4p12s-technical-design skill to create the technical design document."

## 概述

为已接受的业务范围生成 `design.md`。连接需求和验收场景到当前项目的具体技术实现。

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
output_paths:
allowed_write_paths:    # 仅 design.md
```

### 独立模式

自动发现：
- 当前目录或指定目录下的 `requirements-register.md`、`prd.md`、`user-stories.md`
- 项目入口文档和项目结构
- 输出路径：与上游产物同目录的 `design.md`

独立模式下需要主动扫描项目结构以理解现有架构。

## 上下文发现

使用渐进式披露：

1. 首先读取项目入口文档：`.gientech/AGENTS.md`、`AGENTS.md` 或 README
2. 仅跟进本次功能所需的文档
3. 当文档无法识别文件、接口、模型、服务时，使用聚焦的符号或文本搜索
4. 仅读取与已接受范围直接相关的文件
5. 不要扫描整个仓库

## 输出文件

写入唯一产物：`design.md`

```markdown
# <需求名> 技术设计

## 输入依据

| Artifact | Path |
| --- | --- |

## 项目上下文

| 项目项 | 结论 | 来源 |
| --- | --- | --- |

## 实现范围

### 范围内
### 范围外

## 方案设计

### 模块与职责
### 接口、数据流或事件流
### 状态流与边界情况
### 权限、审计、安全或合规
### 配置、数据迁移或兼容性
### 错误处理与恢复

## 设计元素清单

| 设计元素 ID | 类型 | 名称 | 所属模块 | 来源需求 ID | 关联用户故事/验收场景 | 是否需要实现 | 是否需要验证 | 说明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

## 需求到设计映射

| 需求 ID | 用户故事/验收场景 | 设计承接 | 影响文件或模块 |
| --- | --- | --- | --- |

## 测试与验证设计

| 用户故事/验收场景 | 验证方式 | 提交前是否必须验证 | 说明 |
| --- | --- | --- | --- |

## 风险与开放问题

| 项目 | 关联需求 ID | 影响 | 下一步动作 | 是否阻断开发计划 |
| --- | --- | --- | --- | --- |
```

## 设计规则

- 范围内的每个需求 ID 必须有设计承接行或已记录的原因
- 每个 Git 阻断验收项必须有设计级验证策略
- 设计引用的 `US-xxx`、`AC-xxx`、`SC-xxx` 必须存在于上游 `user-stories.md` 中
- 不要创建新的用户故事、验收场景或场景用例 ID
- 不要添加上游产物中不存在的需求
- 不要编造文件路径或 API
- 优先使用现有架构和项目规范
- 设计引入的每个具体实现元素必须列在 `设计元素清单` 中
- 每个设计元素必须映射到上游需求 ID 和 `US-xxx` / `AC-xxx` / `SC-xxx`
- 设计必须内部一致
- 如果上游包含前端/页面/表单/交互需求，设计必须包含前端实现范围和 E2E 验证策略
- 不要使用澄清问题 ID 作为独立源需求

## 质量门禁

返回前验证：

- 每个范围内的需求 ID 都有设计承接或已记录的原因
- 每个 Git 阻断验收项都有验证策略
- 每个具体设计元素在 `设计元素清单` 中恰好出现一次
- 每个需要实现或验证的设计元素都映射到至少一个上游需求 ID 和 `US-xxx` / `AC-xxx` / `SC-xxx`
- 每个范围内的前端交互都有前端设计范围和 E2E 验证策略
- 前端范围没有在上游无明确排除时移到 `范围外`
- 没有引入新的下游专属验收场景或需求

## STEP_RESULT（编排模式）

```yaml
step: design
status: running | passed | blocked | failed
output_files:
  - <delivery_dir>/design.md
required_inputs_checked:
  - <delivery_dir>/requirements-register.md
  - <delivery_dir>/prd.md
  - <delivery_dir>/user-stories.md
  - <project_entry_path>
commands_run: none
gate_evidence:
blocking_reason:
next_action:
```

## 边界

- 不创建 `verification-plan.md`、`tasks/`、代码、提交或部署产物
- 不决定流程终点状态

## 更新 delivery-state.md（仅独立运行模式）

> **编排模式**（被 `4p12s-delivery-orchestrator` 调用）下，`delivery-state.md` 由 orchestrator 统一处理，本 Skill **不直接更新**。

仅在**独立运行模式**（用户直接调用本 Skill）下，本 Step 完成后（或 `failed` / `blocked` 时）必须更新 `<state_path>`：

1. `## Step Checklist` 表中当前 Step 对应行的 `Status` 改为 `passed` / `failed` / `blocked`；`Evidence` 列填入产物文件路径（多文件用 `；` 分隔）
2. 顶部字段：`current_step` 设为下一 Step（`passed` 时）或保持当前 Step（`failed` / `blocked` 时）；`next_action` 设为对下一/当前 Step 的提示；`blocked` / `failed` 时填写 `blocking_reason`；`updated_at` 改为当前 ISO8601 时间戳
3. 不修改其他 Step / Task 的状态
