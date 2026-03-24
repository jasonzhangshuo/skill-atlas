# Skill 分类详解

来源：Anthropic 工程师 Thariq 的内部实践总结（2026-03-24）

## 1. Library / Tool Reference（库与工具参考）

**核心价值**：教 Claude 正确使用某个库、CLI 或 SDK，尤其是内部库或 Claude 容易出错的常用库。

**典型结构**：
- SKILL.md：概述 + 常见用法
- references/：函数签名、参数说明
- gotchas/：已知陷阱和错误用法

**例子**：
- `billing-lib` — 内部计费库：边界情况、常见陷阱
- `internal-platform-cli` — 内部 CLI 的每个子命令 + 使用时机
- `frontend-design` — 让 Claude 更好地理解你的设计系统

---

## 2. Verification（验证技能）

**核心价值**：描述如何测试和验证代码是否正常工作。通常与 Playwright、tmux 等外部工具配合。

**最佳实践**：
- 让 Claude 录制输出视频，便于回看测试过程
- 在每个步骤强制执行程序化断言
- 在 skill 中包含多种脚本

**例子**：
- `signup-flow-driver` — 无头浏览器跑完注册→邮件验证→引导流程，含状态断言钩子
- `checkout-verifier` — 用 Stripe 测试卡驱动结账 UI，验证发票状态
- `tmux-cli-driver` — 需要 TTY 的交互式 CLI 测试

---

## 3. Data & Observability（数据与可观测性）

**核心价值**：连接数据和监控栈。包含带凭据的数据获取库、特定 Dashboard ID、常见工作流说明。

**例子**：
- `funnel-query` — "哪些事件要 join 才能看到注册→激活→付费" + 有 canonical user_id 的表
- `cohort-compare` — 比较两个用户群的留存/转化，标记统计显著差异
- `grafana` — datasource UID、集群名、问题→Dashboard 对照表

---

## 4. Workflow Automation（工作流自动化）

**核心价值**：把重复流程变成一条命令。通常指令简单，但可能依赖其他 skill 或 MCP。

**最佳实践**：把执行结果存入日志文件，让模型保持一致性并能回顾历史执行。

**例子**：
- `standup-post` — 聚合工单系统、GitHub 活动、历史 Slack → 格式化站会，只输出变化
- `create-<ticket-system>-ticket` — 强制 schema（有效枚举值、必填字段）+ 创建后工作流
- `weekly-recap` — 合并的 PR + 关闭的工单 + 部署 → 格式化回顾帖

---

## 5. Scaffolding（脚手架）

**核心价值**：为代码库中的特定功能生成框架样板。当脚手架有自然语言要求、无法纯靠代码覆盖时特别有用。

**例子**：
- `new-<framework>-workflow` — 用你的注解搭建新服务/工作流/处理器
- `new-migration` — 迁移文件模板 + 常见陷阱
- `create-app` — 预置你的 auth、日志和部署配置的新内部应用

---

## 6. Code Quality & Review（代码质量与审查）

**核心价值**：在组织内执行代码规范，辅助代码审查。可包含确定性脚本以提高健壮性。可作为 hooks 或 GitHub Action 自动运行。

**例子**：
- `adversarial-review` — 派生一个全新视角的子 agent 来批评，实施修复，迭代直到问题降级为吹毛求疵
- `code-style` — 执行代码风格，尤其是 Claude 默认做不好的风格
- `testing-practices` — 如何写测试、测什么的指南

---

## 7. DevOps & Deployment（DevOps 与部署）

**核心价值**：帮助在代码库中拉取、推送和部署代码。可能引用其他 skill 来收集数据。

**例子**：
- `babysit-pr` — 监控 PR → 重试不稳定的 CI → 解决合并冲突 → 启用自动合并
- `deploy-<service>` — 构建 → 冒烟测试 → 渐进式流量切换（含错误率对比）→ 回归自动回滚
- `cherry-pick-prod` — 隔离 worktree → cherry-pick → 冲突解决 → 带模板的 PR

---

## 8. Debugging & Investigation（调试与调查）

**核心价值**：接收一个症状（Slack 线程、告警、错误签名），经过多工具调查，产出结构化报告。

**例子**：
- `<service>-debugging` — 为流量最高的服务映射症状→工具→查询模式
- `oncall-runner` — 获取告警 → 检查常见嫌疑 → 格式化发现
- `log-correlator` — 给定请求 ID，从所有可能处理过它的系统拉取匹配日志

---

## 9. Operations & Maintenance（运维与维护）

**核心价值**：执行日常维护和运维程序——其中一些涉及破坏性操作，需要护栏。让工程师更容易遵循关键操作的最佳实践。

**例子**：
- `<resource>-orphans` — 找到孤立的 pods/volumes → 发到 Slack → 浸泡期 → 用户确认 → 级联清理
- `dependency-management` — 你们组织的依赖审批工作流
- `cost-investigation` — "为什么存储/出口账单飙升"，含具体的 bucket 和查询模式

---

## 分发与共享

### 小团队（少量 repo）
直接把 skill 提交到 repo（`./.claude/skills`）。

⚠️ 注意：每个 check-in 的 skill 都会增加模型的 context 消耗。

### 大团队（多 repo）
建立内部 Plugin Marketplace：
1. 有 skill 想分享 → 上传到 GitHub sandbox 文件夹，在 Slack 宣传
2. 获得足够使用量后 → PR 移入 marketplace
3. 保持策展机制，避免低质量或重复的 skill 进入

### 依赖管理
Skill 之间可以相互依赖。直接在 SKILL.md 中引用其他 skill 的名字，模型会在已安装的情况下调用它们。

### 使用追踪
用 `PreToolUse` hook 记录 skill 使用情况，找出：
- 哪些 skill 最受欢迎
- 哪些 skill 触发率低于预期
