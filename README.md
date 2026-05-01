# rules-and-skills-hub

个人 Claude Code / Cursor 规则与 Skills 的统一管理仓库。

## 目录结构

```
├── claude-code-global/          # Claude Code 全局配置备份
│   ├── CLAUDE.md                # ~/.claude/CLAUDE.md 的备份（当前活跃版）
│   └── settings.json            # ~/.claude/settings.json 模板（token 已脱敏）
├── sources/                     # 上游规则源快照
│   ├── cursor-style-rule/       # Cursor 响应风格规则
│   ├── claude-code-multi-agent-harness/  # spec-delegate-review 多代理工作流
│   └── local-working-skills/    # safe-patch-compact-dbx 等本地 skill
├── SKILL.md                     # 旧版合并 skill（已停用，保留参考）
└── README.md
```

## 当前生效的规则体系

### 全局 CLAUDE.md（`~/.claude/CLAUDE.md`）

融合了两套开源规则 + 自定义自主执行原则：

| 来源 | 内容 | 链接 |
|------|------|------|
| **andrej-karpathy-skills** | 代码纪律 §1-4：Think Before Coding / Simplicity First / Surgical Changes / Goal-Driven Execution | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) |
| **obra/superpowers** | 工作流方法论：Brainstorming / Writing Plans / Systematic Debugging / Verification Before Completion / Subagent-Driven Development | [obra/superpowers](https://github.com/obra/superpowers) |
| **自定义** | 角色分工（用户=方向/决策/验收，AI=调研/设计/编码/测试/验证）+ 自主执行原则表 | 本仓库原创 |

### 核心设计理念

**用户做 high-level 的事，AI 做 low-level 的事。**

- 用户只需要一句话触发完整工作流
- AI 自行探索、设计、实施、验证
- 只在不可逆操作或路线选择时才打断用户
- 产出必须经过验证才能宣告完成

### settings.json 配置

- **bypassPermissions** 模式：无确认弹窗
- **Hooks**：PreCompact 存档 ledger + SessionStart 注入上次上下文
- **已禁用**：Databricks 限流保护（100 行 Edit 限制、Bash 500ms 节流）

## 如何使用

### 新机器部署

```bash
# 1. 克隆本仓库
git clone https://github.com/Jcxu97/rules-and-skills-hub.git

# 2. 复制全局 CLAUDE.md
cp rules-and-skills-hub/claude-code-global/CLAUDE.md ~/.claude/CLAUDE.md

# 3. 按模板配置 settings.json（填入你的 token）
cp rules-and-skills-hub/claude-code-global/settings.json ~/.claude/settings.json
# 编辑 ~/.claude/settings.json，替换 <YOUR_TOKEN>
```

### 更新规则

直接编辑 `~/.claude/CLAUDE.md`，然后同步回本仓库：

```bash
cp ~/.claude/CLAUDE.md rules-and-skills-hub/claude-code-global/CLAUDE.md
cd rules-and-skills-hub && git add -A && git commit -m "sync CLAUDE.md" && git push
```

## 上游引用

- [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) — Karpathy 编码纪律
- [obra/superpowers](https://github.com/obra/superpowers) — Claude Code skills 工作流框架

## 版本历史

| 日期 | 变更 |
|------|------|
| 2025-05-02 | v2: 融合 Karpathy Rules + Superpowers 工作流 + 自主执行原则；移除 Databricks 限流 hooks |
| 初始 | v1: unified-rules-skills-harness（Cursor 风格 + 多代理 + chunked-edit + Databricks 防护） |
