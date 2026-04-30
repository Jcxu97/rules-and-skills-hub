# claude-code-multi-agent-harness

独立仓库：**Cursor Agent Skill**（[Agent Skills](https://agentskills.io/) 约定），与任何具体应用项目解耦。  
在**非琐碎任务**上引导 Agent 走 **Spec → 执行 → 评审 pass**，强调 **少追问**、**可验证信源**（DOI / 高星 GitHub / 官方文档），并包含 **学术类**（文献、综述、实验设计）的浓缩纪律。

**不是**可执行插件；**不包含**业务代码。

## 安装（推荐：用户级，全项目生效）

Cursor 会从 **`~/.cursor/skills/`** 自动发现 Skills，见 [Cursor 文档](https://cursor.com/docs/context/skills)。

**方式 A — 克隆到 skills 目录（推荐）**

```bash
git clone https://github.com/Jcxu97/claude-code-multi-agent-harness.git ~/.cursor/skills/claude-code-multi-agent-harness
```

Windows（PowerShell）示例：

```powershell
git clone https://github.com/Jcxu97/claude-code-multi-agent-harness.git "$env:USERPROFILE\.cursor\skills\claude-code-multi-agent-harness"
```

**方式 B — 手动复制**  
将本仓库中的 **`SKILL.md`** 放入：

`~/.cursor/skills/claude-code-multi-agent-harness/SKILL.md`

文件夹名须为 **`claude-code-multi-agent-harness`**（与 `name` 字段一致）。

可选：在 **Cursor → Settings → Rules → User Rules** 加一句说明，强化「复杂任务遵循 harness」习惯。

## 文档

- 全文约定：**[SKILL.md](./SKILL.md)**

## 与其他 Skills

`SKILL.md` 内附有 **可选叠加 Skills** 表（验证、调试、科学写作、Mermaid/UML 等），可按需安装。

## 许可证

**MIT** — 见 [LICENSE](./LICENSE)。

## 相关

若你在使用 **[VLV](https://github.com/Jcxu97/vlv)** 等其它项目，可在那些仓库的 README 中链接本仓库以安装本 Skill；**本 Skill 不以子目录形式附属于 VLV**。
