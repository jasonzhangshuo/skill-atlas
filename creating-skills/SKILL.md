---
name: creating-skills
description: Use when creating, designing, or improving agent skills. Triggers: "写一个 skill", "创建 skill", "帮我做个 skill", "skill 怎么写", "skill 不触发", "skill 太长了", "优化 skill", "skill 分类", "want to create a skill", "write a skill", "build a skill", "skill description not triggering", "skill best practices".
---

# Creating Skills — Best Practices

> 两个核心认知：
> 1. **Skill 是文件夹，不是 Markdown 文件。** 整个目录结构都是 context engineering 的一部分。
> 2. **格式问题已解决，内容设计才是真正的挑战。** 同样的 SKILL.md 外壳，内部逻辑可以完全不同。

## 第一步：选类型

先判断 skill 属于哪类（最好只属于一类），详见 [skill-taxonomy.md](skill-taxonomy.md)：

| 类型 | 核心价值 |
|------|---------|
| Library/Tool Reference | 教 Agent 正确使用某个库/CLI/SDK |
| Verification | 描述如何测试和验证代码 |
| Data & Observability | 连接数据和监控栈 |
| Workflow Automation | 把重复流程变成一条命令 |
| Scaffolding | 生成框架样板代码 |
| Code Quality & Review | 执行代码规范和审查 |
| DevOps & Deployment | 拉取/推送/部署代码 |
| Debugging & Investigation | 从症状出发多工具调查 |
| Operations & Maintenance | 日常运维，含破坏性操作的护栏 |

## 第二步：选内部 Pattern

确定类型后，选择 SKILL.md 内部的逻辑结构，详见 [skill-patterns.md](skill-patterns.md)：

| Pattern | 适用场景 | 核心机制 |
|---------|---------|---------|
| **Tool Wrapper** | 库/规范知识 | 按需加载 references/ 文档 |
| **Generator** | 一致性输出 | 模板 + 样式指南 + 变量收集 |
| **Reviewer** | 代码/内容审查 | 外置 checklist，按严重级分组 |
| **Inversion** | 需求不明确 | Agent 先采访用户，收集完再动手 |
| **Pipeline** | 复杂多步骤 | 严格顺序 + 硬检查点，不能跳步 |

**Pattern 可以组合**：Pipeline 末尾嵌入 Reviewer；Generator 开头用 Inversion 收集变量。

## 第三步：写好关键要素

### Description = 触发条件，不是内容摘要

```yaml
# ❌ 描述了 workflow，Agent 会走捷径跳过正文
description: Use when creating skills - gather requirements, design structure, write SKILL.md

# ✅ 只描述触发场景
description: Use when creating, designing, or improving agent skills. Triggers: "写一个 skill"...
```

### 文件夹结构 = 渐进式披露

```
skill-name/
├── SKILL.md              # 核心指令 + 告知 Agent 哪些文件存在、何时读
├── references/           # 规则/规范/API 文档（Agent 按需读取）
├── assets/               # 输出模板（Agent 复制填充）
├── scripts/              # 可执行脚本（Agent 直接调用）
└── config.json           # 用户配置（首次运行时填写）
```

**关键**：在 SKILL.md 里明确告诉 Agent 什么时候读哪个文件，它才会在合适时机读取。

### Gotchas 章节 = 最高信号密度

每次 Agent 失败就往里加一条。这个章节随时间积累成最有价值的部分。

### 给代码，不给指令

给 Agent 可执行脚本和库函数，让它专注于**组合**（决定做什么），而不是**重建**（重写样板）。

### 给灵活度，不要过度约束

```markdown
# ❌ 过于具体，无法适应变化
"Always use create_ticket with status='open' and priority='medium'"

# ✅ 给信息 + 灵活度
"Use the ticket API. Default priority is 'medium' unless user specifies urgency."
```

## 其他工程实践

**config.json 做首次配置**：需要用户输入的信息（Slack channel 等）存在 config.json，不存在时 Agent 询问用户并创建。

**Session 级 Hooks**：只在该 skill 激活时生效，适合"偶尔需要但不想全局开启"的场景（如 `/careful` 阻止危险操作）。

**内置记忆**：在 skill 目录内存储日志/JSON/SQLite，让 Agent 能回顾历史。持久化数据用 `${CLAUDE_PLUGIN_DATA}`，避免 skill 升级时丢失。

## 创建流程

```
1. 选类型（九种之一）→ 2. 选 Pattern（五种，可组合）
→ 3. 写 description（触发条件）→ 4. 搭文件夹结构
→ 5. 写 SKILL.md 核心指令（<200 行）→ 6. 加 Gotchas
→ 7. 测试：触发是否正确？Agent 是否按预期行动？
```

## 常见错误

| 错误 | 修正 |
|------|------|
| Description 总结了 workflow | 改为只写触发条件 |
| SKILL.md 超过 500 行 | 把详情移到 references/ |
| 没有 Gotchas 章节 | 加上，哪怕先写一条 |
| 所有内容塞进一个文件 | 用文件夹结构做渐进式披露 |
| 指令过于具体 | 给信息 + 灵活度 |
| 没有可执行脚本 | 把重复操作提取为脚本 |
| 需求不明就直接生成 | 用 Inversion pattern 先采访用户 |
| 多步骤流程没有检查点 | 用 Pipeline pattern 加硬门禁 |

## 参考资料

- Skill 九种分类详解：[skill-taxonomy.md](skill-taxonomy.md)
- Skill 五种内部 Pattern：[skill-patterns.md](skill-patterns.md)
- 官方 Cursor Skill 规范：`~/.cursor/skills-cursor/create-skill/SKILL.md`
