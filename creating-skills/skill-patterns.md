# Skill 内部结构 Pattern 详解

来源：Google Cloud ADK 团队对 Anthropic、Vercel、Google 内部 skill 生态的研究（2026-03-24）

> 格式问题已解决（30+ agent 工具标准化了同一个 SKILL.md 格式）。现在的挑战是**内容设计**：同样的外壳，内部逻辑完全不同。

---

## 决策树：选哪个 Pattern？

```
需要传递库/规范知识？          → Tool Wrapper
需要每次生成一致的文档/输出？  → Generator
需要审查代码或内容？           → Reviewer
需求不明确，需要先收集信息？   → Inversion
任务有严格顺序、不能跳步？     → Pipeline
以上组合？                     → 组合使用（见末尾）
```

---

## 1. Tool Wrapper（工具包装器）

**核心思想**：把库/规范/CLI 知识打包成按需加载的上下文，只在 Agent 真正需要时才加载。

**适用场景**：Library Reference 类 skill，内部 SDK、设计系统、CLI 工具。

**结构**：
```
api-expert/
├── SKILL.md
└── references/
    └── conventions.md    # 完整规范，按需加载
```

**SKILL.md 模板**：
```markdown
---
name: api-expert
description: FastAPI 开发规范。Use when building, reviewing, or debugging FastAPI applications.
---

你是 FastAPI 专家。

## 审查代码时
1. 加载 references/conventions.md
2. 对照每条规范检查用户代码
3. 每个违规：引用具体规则 + 给出修复建议

## 写代码时
1. 加载 references/conventions.md
2. 严格遵循每条规范
3. 所有函数签名加类型注解
```

**关键点**：明确告诉 Agent **什么时候**加载 references/，而不是让它自己猜。

---

## 2. Generator（生成器）

**核心思想**：用模板 + 样式指南 + 变量收集，确保每次输出结构一致。Agent 充当"项目经理"，协调资产的读取和填充。

**适用场景**：Scaffolding 类 skill，API 文档生成、提交信息、项目架构。

**结构**：
```
report-generator/
├── SKILL.md
├── references/
│   └── style-guide.md    # 语气和格式规则
└── assets/
    └── report-template.md # 输出模板（Agent 复制填充）
```

**SKILL.md 模板**：
```markdown
---
name: report-generator
description: 生成结构化技术报告。Use when user asks to write, create, or draft a report or analysis.
---

Step 1: 加载 references/style-guide.md（语气和格式规则）
Step 2: 加载 assets/report-template.md（输出结构）
Step 3: 询问用户缺失的信息：主题、关键发现、目标受众
Step 4: 按样式指南填充模板，每个章节必须存在
Step 5: 输出完整的 Markdown 文档
```

**关键点**：SKILL.md 不包含实际模板内容，只负责**协调**资产的读取顺序。

---

## 3. Reviewer（审查者）

**核心思想**：把"检查什么"和"怎么检查"分离。检查标准外置到 checklist 文件，按严重级别分组输出。

**适用场景**：Code Quality 类 skill，PR 审查、安全审计、内容审核。

**结构**：
```
code-reviewer/
├── SKILL.md
└── references/
    └── review-checklist.md  # 可替换为不同领域的 checklist
```

**SKILL.md 模板**：
```markdown
---
name: code-reviewer
description: 审查代码质量和常见 bug。Use when user submits code for review or asks for a code audit.
metadata:
  severity-levels: error,warning,info
---

Step 1: 加载 references/review-checklist.md
Step 2: 理解代码目的，再开始批评
Step 3: 对每个违规：行号 + 严重级别（error/warning/info）+ 原因 + 具体修复
Step 4: 输出结构化审查：Summary → Findings（按严重级排序）→ Score → Top 3 建议
```

**关键点**：只需换掉 `review-checklist.md`（如换成 OWASP 安全清单），就能得到完全不同的专业审查，复用同一套 skill 基础设施。

---

## 4. Inversion（反转控制）

**核心思想**：反转控制权——Agent 先当"采访者"，强制收集完所有上下文再动手，对抗 Agent 急于生成的天性。

**适用场景**：需求不明确的 Workflow Automation、项目规划、任何"信息不足就生成会出错"的场景。

**结构**：
```
project-planner/
├── SKILL.md
└── assets/
    └── plan-template.md
```

**SKILL.md 模板**：
```markdown
---
name: project-planner
description: 通过结构化提问收集需求后再规划项目。Use when user says "I want to build", "help me plan", "start a new project".
metadata:
  interaction: multi-turn
---

你正在进行结构化需求采访。**在所有阶段完成前，不要开始构建或设计。**

## Phase 1 — 问题发现（逐一提问，等待每个回答）
- Q1: "这个项目为用户解决什么问题？"
- Q2: "主要用户是谁？技术水平如何？"
- Q3: "预期规模是多少？"

## Phase 2 — 技术约束（仅在 Phase 1 完全回答后）
- Q4: "部署环境是什么？"
- Q5: "有技术栈要求或偏好吗？"
- Q6: "不可妥协的需求是什么？"

## Phase 3 — 综合输出（仅在所有问题回答后）
1. 加载 assets/plan-template.md
2. 用收集的需求填充模板
3. 呈现计划，询问："这准确反映了你的需求吗？"
4. 根据反馈迭代直到用户确认
```

**关键点**：`DO NOT start building until all phases are complete` 是核心——显式的、不可绕过的门禁指令。

---

## 5. Pipeline（流水线）

**核心思想**：严格顺序执行，每步都有硬检查点，Agent 不能跳步或绕过。SKILL.md 本身就是工作流定义。

**适用场景**：复杂多步骤任务，DevOps 部署、文档生成、任何"跳步会出错"的场景。

**结构**：
```
doc-pipeline/
├── SKILL.md
├── references/
│   ├── docstring-style.md
│   └── quality-checklist.md
└── assets/
    └── api-doc-template.md
```

**SKILL.md 模板**：
```markdown
---
name: doc-pipeline
description: 通过多步流水线从 Python 源码生成 API 文档。Use when user asks to document a module or generate API docs.
metadata:
  steps: "4"
---

你正在运行文档生成流水线。**按顺序执行每步，步骤失败不得继续。**

## Step 1 — 解析与清单
分析代码，提取所有公共类/函数/常量，呈现为清单。
询问："这是你要文档化的完整公共 API 吗？"

## Step 2 — 生成 Docstring
对每个缺少 docstring 的函数：
- 加载 references/docstring-style.md
- 按样式生成 docstring，逐一呈现给用户确认
**在用户确认前不得进入 Step 3。**

## Step 3 — 组装文档
加载 assets/api-doc-template.md，将所有内容编译成完整 API 参考文档。

## Step 4 — 质量检查
对照 references/quality-checklist.md 审查。
发现问题先修复，再呈现最终文档。
```

**关键点**：每步只在需要时加载对应文件，保持 context window 干净。硬门禁（"在用户确认前不得进入"）是 Pipeline 的灵魂。

---

## Pattern 组合

Pattern 不是互斥的，可以像积木一样组合：

| 组合 | 场景 |
|------|------|
| Inversion → Generator | 先收集变量，再填充模板生成文档 |
| Pipeline + Reviewer | 流水线最后一步加审查，自我检验 |
| Tool Wrapper + Pipeline | 加载规范后，按严格步骤执行 |
| Inversion → Pipeline | 先采访需求，再按流水线执行 |

**实现方式**：在 SKILL.md 里直接引用其他 skill 的名字，Agent 会在已安装的情况下调用它们。
