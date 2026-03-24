# 多 Agent workspace 架构

## 核心原则

**每个 Agent 必须有独立的 workspace**。  
共用 workspace = 让多 Agent 失去意义最快的方式。

---

## 推荐目录结构

```
~/.openclaw/
├── workspace/                    # 主 Agent（默认）
│   ├── AGENTS.md
│   ├── SOUL.md
│   └── ...
├── workspace-writer/             # 写作 Agent
│   ├── AGENTS.md                 # 专注内容创作的规则
│   ├── SOUL.md                   # 创意、感性的性格
│   └── ...
├── workspace-coder/              # 编程 Agent
│   ├── AGENTS.md                 # 专注代码质量的规则
│   ├── SOUL.md                   # 严谨、精确的性格
│   └── ...
└── skills/                       # 共享 skills（所有 Agent 可加载）
    └── project-context/
        └── SKILL.md              # 项目背景、用户基础偏好
```

---

## 共享信息策略

**问题**：有些信息所有 Agent 都需要知道（项目背景、用户固定偏好），但每个 workspace 都手抄一份，改起来很痛苦。

**解决方案**：做成共享 skill，放到 `~/.openclaw/skills/`。

```markdown
# ~/.openclaw/skills/project-context/SKILL.md
---
name: project-context
description: 项目基础背景，所有 Agent 在处理项目相关任务时加载
---

# 项目背景

## 项目简介
[项目名称、核心目标、技术栈]

## 用户基础偏好
[输出风格、语言偏好、不喜欢的表达方式]

## 常用约定
[命名规范、文件组织、关键路径]
```

改一处，所有 Agent 同步更新。

---

## openclaw.json 多 Agent 配置

```json
{
  "agents": {
    "list": [
      {
        "id": "manager",
        "workspace": "~/.openclaw/workspace/",
        "subagents": {
          "allowAgents": ["writer", "coder"]
        }
      },
      {
        "id": "writer",
        "workspace": "~/.openclaw/workspace-writer/"
      },
      {
        "id": "coder",
        "workspace": "~/.openclaw/workspace-coder/"
      }
    ]
  }
}
```

**注意**：`subagents.allowAgents` 是权限白名单。  
没有出现在这个列表里的 Agent，manager 无法通过 `sessions_spawn` 调用它。

---

## workspace vs agentDir 区别

| 概念 | 存放内容 | 类比 |
|------|---------|------|
| `workspace` | SOUL.md、AGENTS.md、skills…（怎么干活） | 员工的工作手册 |
| `agentDir`（openclaw.json 字段） | auth-profiles.json、models.json…（运行状态） | 员工的工位和工具箱 |

`agentDir` 不是磁盘上天然存在的目录名，它只是 openclaw.json 里的配置字段，指向存放运行状态的路径。
