---
name: "4p12s-e2e-test"
display_name: "4p12s-e2e-test"
description: "第10步 — E2E 测试：按验证计划执行端到端测试，验证完整业务流程。"
version: "1.0.2"
---

# E2E 测试

**Announce at start:** "I'm using the 4p12s-e2e-test skill to run end-to-end verification with real services."

## 概述

完整的 E2E 阶段：写测试代码 → 启动真实服务 → 准备数据 → 执行真实 E2E → 修复失败 → 重跑直到通过。

开发任务阶段不写 E2E 测试代码。本阶段必须根据 `verification-plan.md` 中的测试用例契约补齐 E2E 测试代码。

## 双模运行

### 编排模式

路径包必须包含：

```yaml
repo_root:
delivery_dir:
state_path:
project_entry_path:
verification_plan_path:
verification_result_path:
user_stories_path:
prd_path:
output_paths:
allowed_write_paths:    # verification-result.md、E2E 目标测试文件、修复所需的产品代码和配置
```

### 独立模式

自动发现：
- 当前目录或指定目录下的 `verification-plan.md`、`verification-result.md`
- 项目入口文档和 E2E 框架配置（Playwright、Cypress 等）
- 输出路径：与上游产物同目录的 `verification-result.md`

## 执行流程

```text
读取 verification-plan.md 中所有 E2E / cross_boundary_e2e TC
-> 为每个 TC 编写或补齐目标 E2E 测试代码
-> 优先读取 .gientech/AGENTS.md 或 project_entry_path 发现服务启动和 E2E 命令
-> 启动或连接该用例所需的真实目标服务和依赖服务
-> 执行所有必需服务的健康检查
-> 准备测试数据
-> 执行发现到的真实 E2E 命令
-> 若失败，归因并修复产品代码或测试代码
-> 重跑失败 TC 和受影响回归项
-> 全部通过后写入 verification-result.md##E2E 验证结果
-> 返回 STEP_RESULT
```

## E2E 测试代码编写

要求：
- 使用 `verification-plan.md` 指定的目标测试文件
- 覆盖每个 E2E / cross_boundary_e2e TC
- 前端用户流测试必须通过真实前端和真实后端/API 交互
- 测试必须包含真实接口断言，不能只断言静态页面、loading、路由跳转或 mock 响应
- 禁止在 E2E 测试代码中使用 `page.route().fulfill()`、`intercept`、`mock`、`jest.fn()` 或任何请求拦截/伪造响应的手段
- 不得凭空发明新的业务范围
- 可以修复或补充既有 E2E 测试代码

## 服务启动和真实验证

必须执行从 `.gientech/AGENTS.md`、`project_entry_path` 或其引用文档中发现的命令，不得凭常识猜命令。

### 真实服务

- 前端用户流：必须启动或连接前端服务和后端/API 服务
- API E2E / 跨服务 E2E：必须启动或连接目标 API/服务和必要依赖服务
- 记录最小执行证据
- 执行每个必需服务的健康检查
- 健康检查不通过不得继续执行 E2E

### 测试数据

- 按 `verification-plan.md` 准备测试数据
- 记录数据准备动作
- 如果测试数据缺失且无法准备，返回 `blocked`

### E2E 执行

- 执行从项目入口文档发现的 E2E 命令
- 前端用户流 E2E 必须访问真实前端和真实后端/API
- 禁止使用 mock 结果作为通过证据
- 每个提交前必须执行的 E2E TC 都必须有真实执行记录

## 修复循环

1. 对失败 TC 归因：产品代码、E2E 测试代码、环境、测试数据、配置
2. 按失败 TC 关联的 US/AC/SC 和 PRD 读取必要上下文
3. 修复前端代码、后端代码、配置、测试数据或 E2E 测试代码
4. 重跑失败 TC 和受影响回归项
5. 最多 5 轮。超过后返回 `blocked` 或 `failed`

## 输出文件

```markdown
## E2E 验证结果

### 执行摘要

- 执行时间：
- 验证结论：验证通过 / 验证失败 / 验证阻断
- 服务地址：
- Git 阻断项：无 / 有

### E2E 测试代码

| 用例 ID | 目标测试文件 | 动作 | 结果 |
| --- | --- | --- | --- |

### 最小执行证据规则

- 只记录发现来源、执行动作、命令摘要、结果、关键输出摘要
- 命令摘要格式：`<cwd> <command>`
- 不记录完整 stdout/stderr
- 不能只写"完成""已执行""通过"

### 服务启动证据

| 服务 | 发现来源 | 动作 | 命令摘要 | 健康检查摘要 | 结果 | 关键输出摘要 |
| --- | --- | --- | --- | --- | --- | --- |

### 测试数据证据

| 数据项 | 发现来源 | 动作 | 命令摘要或外部数据来源 | 结果 | 关键输出摘要 |
| --- | --- | --- | --- | --- | --- |

### 用例执行结果

| 用例 ID | 关联需求 ID | 测试文件 | 发现来源 | 命令摘要 | 结果 | 真实接口断言 | 关键输出摘要 |
| --- | --- | --- | --- | --- | --- | --- | --- |

### 失败与修复记录

| 轮次 | 用例 ID | 归因 | 修改文件 | 重跑结果 |
| --- | --- | --- | --- | --- |

### 未执行项

| 项目 | 原因 | 是否阻断 Git | 处理结论 |
| --- | --- | --- | --- |
```

## 通过条件

仅当全部满足时，才能返回 `passed`：

- 每个 E2E TC 都有目标测试代码
- 当前 E2E 用例所需的所有真实服务都已启动或连接，并有健康检查证据
- 前端用户流包含真实前端和真实后端/API 证据
- 测试数据已准备或明确无需准备
- E2E 命令已真实执行
- 所有提交前必须执行的 E2E TC 都通过
- 每个 E2E TC 都有真实接口断言证据
- `verification-result.md##E2E 验证结果` 已写入
- Git 阻断项为零

以下情况不得返回 `passed`：
- 只写了测试代码但没有运行
- 只运行了前端 webServer 或静态页面检查
- 没有真实服务或真实 API 证据
- 使用 mock 结果作为通过证据
- 健康检查、测试数据或 E2E 命令缺失

## STEP_RESULT（编排模式）

```yaml
step: verification_e2e
status: running | passed | blocked | failed
output_files:
  - <delivery_dir>/verification-result.md
required_inputs_checked:
  - <delivery_dir>/delivery-state.md
  - <delivery_dir>/verification-plan.md
  - <delivery_dir>/verification-result.md
  - <delivery_dir>/user-stories.md
  - <delivery_dir>/prd.md
  - <repo_root>/.gientech/AGENTS.md（存在时必须读取）
  - <project_entry_path>
commands_run:
gate_evidence:
blocking_reason:
next_action:
```

## 边界

- 不重复运行集成测试，除非 E2E 修复影响了集成边界
- 不提交、不推送、不触发发布
- 不报告流程终点状态

## 更新 delivery-state.md（仅独立运行模式）

> **编排模式**（被 `4p12s-delivery-orchestrator` 调用）下，`delivery-state.md` 由 orchestrator 统一处理，本 Skill **不直接更新**。

仅在**独立运行模式**（用户直接调用本 Skill）下，本 Step 完成后（或 `failed` / `blocked` 时）必须更新 `<state_path>`：

1. `## Step Checklist` 表中当前 Step 对应行的 `Status` 改为 `passed` / `failed` / `blocked`；`Evidence` 列填入产物文件路径（多文件用 `；` 分隔）
2. 顶部字段：`current_step` 设为下一 Step（`passed` 时）或保持当前 Step（`failed` / `blocked` 时）；`next_action` 设为对下一/当前 Step 的提示；`blocked` / `failed` 时填写 `blocking_reason`；`updated_at` 改为当前 ISO8601 时间戳
3. 不修改其他 Step / Task 的状态
