---
name: "4p12s-requirements"
display_name: "4p12s-requirements"
description: "第2步 — 业务需求分析：读取业务方原始需求文档，提炼生成原始需求登记表。"
version: "1.0.2"
---

# 需求访谈与登记

**Announce at start:** "I'm using the 4p12s-requirements skill to capture and clarify business requirements."

## 概述

将原始业务输入转换为交付流程的源需求登记。包含两个阶段：

1. **需求登记**：提取、分类、分配稳定 ID
2. **需求澄清**：对阻断 PRD 生成的问题向用户提问，持续处理直到足以支撑 PRD

## 双模运行

### 编排模式（被 4p12s-delivery-orchestrator 调用）

接收路径包，读取其中的输入文件，写入指定的输出路径，返回 `STEP_RESULT`。

路径包必须包含：

```yaml
repo_root:
delivery_dir:
state_path:
project_entry_path:
raw_input_path:          # 原始业务输入路径或文本
requirements_register_path:  # 已存在的 requirements-register.md 路径（更新场景）
output_paths:
allowed_write_paths:     # 仅 requirements-register.md
```

### 独立模式（用户直接调用）

用户直接提供业务需求（文本、文件路径、截图等）。Skill 自动发现：

- 当前项目根目录（`git rev-parse --show-toplevel` 或当前目录）
- 项目入口文档（`.gientech/AGENTS.md`、`AGENTS.md`、`README.md`）
- 输出路径：`deliveries/<需求名>/requirements-register.md`（如用户未指定）

独立模式直接面向用户输出 `requirements-register.md`，不需要返回 `STEP_RESULT`。

## 输出文件

写入唯一产物：`requirements-register.md`

```markdown
# <需求名> 源需求登记

## 输入来源

| 来源 | 位置 | 说明 |
| --- | --- | --- |

## 视觉证据登记

| image_id | 图片路径或附件名 | 区域位置 | 视觉观察 | 提取出的需求 ID | 置信度 | 未解决问题 |
| --- | --- | --- | --- | --- | --- | --- |

## 需求总览

- 业务目标：
- 成功标准：
- 范围内：
- 范围外：

## 源需求登记

| 需求 ID | 类型 | 来源位置 | 原文摘录或视觉观察 | 结构化表述 | 优先级 | 验收提示 | 当前状态 |
| --- | --- | --- | --- | --- | --- | --- | --- |

## 澄清问题

| 问题 ID | 关联需求 ID | 问题 | 影响 | 是否阻断后续 PRD | 询问方式 |
| --- | --- | --- | --- | --- | --- |

## 范围与风险

| 事项 | 关联需求 ID | 说明 | 下一步动作 |
| --- | --- | --- | --- |
```

## 需求 ID 规则

按需求类型分配稳定 ID：

- `BR-001`：业务背景、目标或成功标准
- `FUNC-001`：功能能力
- `RULE-001`：业务规则、校验、风控或策略
- `FLOW-001`：流程、状态流、正常路径、异常路径或边界路径
- `FIELD-001`：字段、数据源、数据含义或数据质量规则
- `UI-001`：页面结构、布局、控件、交互状态、视觉风格、截图证据或图片中可见的现有系统行为
- `PERM-001`：角色、权限、授权、审计或合规规则
- `NFR-001`：性能、安全、兼容性、可观测性、可用性或运维要求
- `INT-001`：上下游系统、API、事件、批处理、文件或外部依赖

已有登记中已有 ID 不得重新编号。同一前缀的新 ID 追加在当前最大编号之后。

## 提取规则

- 始终保留输出结构中的所有固定章节
- 保留原始业务含义，不将业务请求翻译为技术方案
- 对于图片/截图/设计稿/扫描件，先记录视觉观察作为证据，再映射到稳定需求 ID
- 当源描述涉及用户可见界面、页面/表单、字段、交互、输入/输出时，创建 `UI-*` 需求
- 不要将前端交互信号降级为仅 `NFR-*`
- 图片模糊、置信度低或与文本冲突时，创建澄清问题而非猜测
- 当复合需求有不同的验收信号、负责人、系统或风险时，进行拆分
- 将业务规则、数据规则、权限、异常路径和非功能需求作为独立行保留
- 澄清问题 ID（`Q-001`）是证据 ID，不是源需求 ID
- 仅当调用方或源明确排除时才标记 `暂不做`
- 不要编造优先级，缺失时使用 `未明确`

## 澄清流程

登记完成后，检查是否存在阻断 PRD 生成的澄清项：

1. 找到标记为阻断的澄清项，按影响合并为最少必要问题
2. 直接向用户提出澄清问题
3. 用户回答后，更新相关需求行、澄清行和验收提示
4. 保留原始源描述，将澄清答案作为补充证据记录
5. 重新审计是否还有阻断性澄清项
6. 仅在没有阻断性澄清项、用户明确无法提供关键答案时结束

**澄清边界**：
- 不提出设计、实现、测试、Git 或发布问题
- 不提出多种产品方案，除非源需求本身歧义且答案对 PRD 语义是必需的
- 不创建下游产物
- 不删除原始需求文本
- 如果答案改变了范围，记录范围决策和受影响的需求 ID
- 非阻断性澄清项也必须有处置结论

## 质量门禁

返回前验证：

- `requirements-register.md` 包含所有固定章节，包括 `视觉证据登记`
- 范围内的每个业务事实都有稳定的源需求 ID
- 每个澄清项都有阻断标志和下一步动作
- 非阻断性澄清项有下游阶段可引用的处置记录
- 每个文本或图片前端信号都映射到 `UI-*`，或有源/调用方证据明确排除
- 没有前端交互信号被仅降级为 `NFR-*`、风格描述或人工验收措辞
- 文件中没有出现下游专属 US、AC、SC、TC、Task、设计、实现、Git 或发布结论
- 澄清 ID 未替代源需求 ID

## STEP_RESULT（编排模式）

```yaml
step: requirements
status: running | passed | blocked | failed
output_files:
  - <delivery_dir>/requirements-register.md
required_inputs_checked:
  - <project_entry_path>
  - <raw_input_path_or_inline_text>
commands_run: none
gate_evidence:
blocking_reason:
next_action:
```

- `passed`：登记有足够证据支撑 PRD 生成
- `blocked`：存在阻断 PRD 生成的澄清问题且用户无法提供答案，或需要外部业务决策

## 边界

- 不创建 `prd.md`、`user-stories.md`、`design.md`、`tasks/`、`verification-plan.md`、`verification-result.md` 或 `deploy-log.md`
- 不开始实现、测试、Git 或发布

## 更新 delivery-state.md（仅独立运行模式）

> **编排模式**（被 `4p12s-delivery-orchestrator` 调用）下，`delivery-state.md` 由 orchestrator 统一处理，本 Skill **不直接更新**。

仅在**独立运行模式**（用户直接调用本 Skill）下，本 Step 完成后（或 `failed` / `blocked` 时）必须更新 `<state_path>`：

1. `## Step Checklist` 表中当前 Step 对应行的 `Status` 改为 `passed` / `failed` / `blocked`；`Evidence` 列填入产物文件路径（多文件用 `；` 分隔）
2. 顶部字段：`current_step` 设为下一 Step（`passed` 时）或保持当前 Step（`failed` / `blocked` 时）；`next_action` 设为对下一/当前 Step 的提示；`blocked` / `failed` 时填写 `blocking_reason`；`updated_at` 改为当前 ISO8601 时间戳
3. 不修改其他 Step / Task 的状态
