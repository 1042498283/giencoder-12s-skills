---
name: "4p12s-verification-plan"
display_name: "4p12s-verification-plan"
description: "第6步 — 生成验证计划：基于所有需求和设计产物，制定集成测试和 E2E 测试策略。"
version: "1.0.2"
---

# 验证计划

**Announce at start:** "I'm using the 4p12s-verification-plan skill to create the verification plan."

## 概述

从上游交付产物生成 `verification-plan.md`。定义"要验证什么"，不回答"命令如何运行"。

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
verification_plan_path:  # 已存在时
output_paths:
allowed_write_paths:     # 仅 verification-plan.md
```

### 独立模式

自动发现：
- 当前目录或指定目录下的 `requirements-register.md`、`prd.md`、`user-stories.md`、`design.md`
- 项目入口文档和测试框架配置
- 输出路径：与上游产物同目录的 `verification-plan.md`

## 验证契约边界

本阶段只回答"要验证什么"，不回答"命令如何运行"。

必须生成：
- 覆盖关系：需求、用户故事、验收场景、场景用例和设计元素
- 验证层级：unit / component / service_api / service_integration / cross_boundary_e2e
- Git 阻断与提交前自动化决策
- 目标测试文件或测试目录约定
- 真实接口断言、禁用 Mock 规则和测试数据要求
- 执行期所需真实服务类型
- 前端是否适用

不写：
- 启动或连接命令、端口和地址
- 健康检查命令、集成测试命令、E2E 命令
- 命令来源证据

## 输出文件

写入唯一产物：`verification-plan.md`

```markdown
# <需求名> 验证计划

## 输入依据

| Artifact | Path |
| --- | --- |

## 验证覆盖表

| 来源需求 ID | 用户故事 ID | 验收场景 ID | 场景用例 ID | PRD 验收标准 | Given/When/Then 摘要 | 验证类型 | 验证对象层级 | 最低验证层级 | 是否允许降级验证 | 降级理由 | 是否产品行为变更 | 是否提交前自动化验证 | 是否阻断 Git | 是否自动化 | 目标测试文件/目录 | 所需真实服务类型 | 前端是否适用 | 测试数据要求 | 真实接口断言 | 禁用 Mock 规则 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

## 设计元素验证映射

| 设计元素 ID | 名称 | 所属模块 | 来源需求 ID | 用户故事/验收场景 | 场景用例 | 验证用例 | 最低验证层级 | 是否阻断 Git |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

## 测试范围

### 范围内
### 范围外

## 测试数据

| 数据项 | 用途 | 要求 | 来源 |
| --- | --- | --- | --- |

## 执行期发现要求

| 用例 ID | 发现对象 | 必须优先读取 | 可继续聚焦读取 | 发现失败时处理 |
| --- | --- | --- | --- | --- |

## 验证用例

### TC-001 <用例标题>

- 关联需求 ID：
- 关联用户故事 ID：
- 关联验收场景 ID：
- 关联场景用例 ID：
- Given/When/Then 摘要：
- 优先级：P0 / P1 / P2
- 类型：单元 / 接口 / 集成 / E2E候选 / 人工验收 / 非代码验证 / 安全 / 性能
- 是否产品行为变更：
- 验证对象层级：
- 最低验证层级：
- 是否允许降级验证：
- 降级理由：
- 是否提交前自动化验证：
- 是否提交前必须执行：
- 未执行是否阻断 Git：
- 是否自动化测试：
- 目标测试文件/目录：
- 前置条件：
- 所需真实服务类型：
- 前端是否适用：
- 真实接口断言：
- 禁用 Mock 规则：
- 测试数据：
- 步骤：
  1.
  2.
  3.
- 预期结果：
- 覆盖说明：

## 风险与未覆盖项

| 风险/缺口 | 关联需求 ID | 原因 | 是否阻断 Git | 下一步动作 |
| --- | --- | --- | --- | --- |
```

## 覆盖规则

- 范围内的每个需求 ID 必须映射到至少一个测试用例或验证项
- 每个 PRD 验收标准必须映射到至少一个测试用例
- `user-stories.md` 的每个 `AC-xxx` / `SC-xxx` 必须映射到至少一个测试用例
- `design.md##设计元素清单` 中的每个元素必须映射到至少一个测试用例或非代码处理决策
- 公共入口点、API、路由、控制器、集成调用、持久化状态变更、前端页面的最低验证层级必须匹配外部可观察边界
- `TC-xxx` 由本文件拥有；`US-xxx`、`AC-xxx`、`SC-xxx` 必须从 `user-stories.md` 复用
- Git 阻断验收项必须有提交前验证证据或列为阻断项
- 目标测试文件/目录必须是完整的项目相对路径，不要使用占位符
- 每个测试用例必须声明 `验证对象层级` 和 `最低验证层级`
- 跨模块、跨服务、持久化状态变更、权限、审计、用户关键路径默认不可降级
- 任何范围内的前端交互必须映射到至少一个前端 E2E 或 `cross_boundary_e2e` 验证用例
- 前端 E2E 用例必须设置提交前自动化验证，并声明 `frontend` 和后端真实服务类型
- 前端 E2E 用例不能只依赖静态渲染、loading、路由导航或纯 UI 断言
- 命令、端口、健康检查是执行阶段发现内容，不属于本阶段
- 不要将范围缩减为仅快乐路径

## 质量门禁

返回前验证：

- `verification-plan.md` 已创建或更新，包含所有必需章节
- 每个测试用例都有稳定的 `TC-xxx` ID
- 测试用例引用的 `US-xxx`、`AC-xxx`、`SC-xxx` 都存在于上游
- 所有需求 ID、PRD 验收标准、设计元素都已覆盖
- 每个 Git 阻断项都有验证路径或列为阻断项
- 每个提交前用例都有目标测试文件/目录、所需真实服务类型、测试数据要求、真实接口断言和禁用 Mock 规则
- 每个范围内的前端交互都有 E2E 用例
- 目标测试文件/目录与项目运行器配置一致
- 没有目标测试文件包含占位符路径
- 此阶段不生成测试代码或任务拆分

## STEP_RESULT（编排模式）

```yaml
step: verification_plan
status: running | passed | blocked | failed
output_files:
  - <delivery_dir>/verification-plan.md
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

- 不编写自动化测试代码
- 不运行测试
- 不编辑产品代码
- 不创建 `tasks/`
- 不决定流程终点状态

## 更新 delivery-state.md（仅独立运行模式）

> **编排模式**（被 `4p12s-delivery-orchestrator` 调用）下，`delivery-state.md` 由 orchestrator 统一处理，本 Skill **不直接更新**。

仅在**独立运行模式**（用户直接调用本 Skill）下，本 Step 完成后（或 `failed` / `blocked` 时）必须更新 `<state_path>`：

1. `## Step Checklist` 表中当前 Step 对应行的 `Status` 改为 `passed` / `failed` / `blocked`；`Evidence` 列填入产物文件路径（多文件用 `；` 分隔）
2. 顶部字段：`current_step` 设为下一 Step（`passed` 时）或保持当前 Step（`failed` / `blocked` 时）；`next_action` 设为对下一/当前 Step 的提示；`blocked` / `failed` 时填写 `blocking_reason`；`updated_at` 改为当前 ISO8601 时间戳
3. 不修改其他 Step / Task 的状态
