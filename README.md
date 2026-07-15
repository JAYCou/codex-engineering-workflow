# Codex 工程工作流

一套为 OpenAI Codex 优化的中文工程工作流：按任务风险自动选择足够且不过度的计划、实现和验证强度。

它受 [Superpowers](https://github.com/obra/superpowers) 的工程方法论启发，但进行了独立重写。重点保留证据驱动、系统化调试、Git 安全和完成前验证，同时移除简单任务上的强制设计、强制 TDD、强制 worktree 和强制多 Agent。

## 适合谁

- 希望 Codex 尽量自主完成任务，只在真正需要授权时暂停的人；
- 希望小修改快速交付、复杂任务认真验证的人；
- 希望保护脏工作区、避免无关重构和未经验证的“已完成”的人；
- 主要使用中文沟通，但保留必要技术术语的人。

它不会替代具体项目已有的架构、测试、安全或发布规范。高度受管制的生产系统仍应使用项目自己的审批和变更流程。

## 工作强度

| 等级 | 场景 | 行为 |
|---|---|---|
| L0 只读 | 解释、审查、状态汇报 | 只读检查并提供证据 |
| L1 轻量 | 文案、小配置、局部低风险修改 | 直接修改并做最小相关验证 |
| L2 标准 | Bug、功能、重构、依赖调整 | 找真源、简短计划、实现并针对性验证 |
| L3 高风险 | 权限、迁移、生产、安全、不可逆操作 | 扩大验证，缺少关键授权时暂停 |

## 与 Superpowers 的主要差异

| 项目 | 本工作流 |
|---|---|
| brainstorming | 只在需求或架构确实需要讨论时使用 |
| TDD | 优先用于有测试体系的功能、Bug 和高回归风险变更，不机械强制 |
| worktree | 需要隔离或并行开发时使用，不是每次修改都创建 |
| 多 Agent | 只用于互不干扰且能明显节省时间的独立任务 |
| 用户确认 | 仅在凭据、生产、费用、不可逆操作、外部发布或重大取舍前暂停 |
| 验证 | 始终需要与风险匹配的真实证据，并为可能挂起的工具设置时间边界 |

## 仓库内容

```text
skills/codex-engineering-workflow/   全局 Codex Skill
templates/AGENTS.md                  可选的项目级规则模板
tests/scenarios.md                   RED/GREEN 行为压力测试记录
docs/superpowers/                    设计与实施计划
```

## Windows 全局安装

先克隆仓库：

```powershell
git clone https://github.com/JAYCou/codex-engineering-workflow.git
```

确认目标目录不存在后复制 Skill：

```powershell
$codexHome = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { "$env:USERPROFILE\.codex" }
$target = Join-Path $codexHome 'skills\codex-engineering-workflow'
if (Test-Path -LiteralPath $target) {
    throw "目标已存在，请先备份或按更新说明处理：$target"
}
Copy-Item -Recurse -LiteralPath `
  '.\codex-engineering-workflow\skills\codex-engineering-workflow' `
  -Destination $target
```

重新启动 Codex，使 Skill 列表刷新。

## 其他系统安装

把 `skills/codex-engineering-workflow` 整个目录复制到：

```text
${CODEX_HOME:-$HOME/.codex}/skills/codex-engineering-workflow
```

必须保留其中的 `SKILL.md` 和 `agents/openai.yaml`。

## 项目级 AGENTS.md

模板用于给某个仓库增加稳定安全边界。复制前先检查目标项目是否已有 `AGENTS.md`：

```powershell
$target = '.\你的项目\AGENTS.md'
if (Test-Path -LiteralPath $target) {
    throw '目标项目已有 AGENTS.md，请人工合并，不要覆盖。'
}
Copy-Item -LiteralPath `
  '.\codex-engineering-workflow\templates\AGENTS.md' `
  -Destination $target
```

项目已有规则时应人工合并，且项目自身的技术栈、安全和发布规范优先。

## 使用方式

Codex 可以根据任务描述自动触发，也可以显式调用：

```text
使用 $codex-engineering-workflow 修复这个 Bug，并运行与风险匹配的验证。
```

```text
使用 $codex-engineering-workflow 审查当前仓库，但先不要修改文件。
```

```text
使用 $codex-engineering-workflow 完成这个小配置修改，不要制造不必要的长计划。
```

## 更新

在仓库中拉取最新版本，先检查变更：

```powershell
git -C .\codex-engineering-workflow fetch origin
git -C .\codex-engineering-workflow diff HEAD..origin/main -- skills/codex-engineering-workflow
git -C .\codex-engineering-workflow pull --ff-only
```

确认 diff 后，把新版 Skill 目录复制到原安装位置。不要在未审查变更时自动覆盖全局 Skill。

## 卸载

先核对即将删除的绝对路径确实是该 Skill，再执行：

```powershell
$codexHome = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { "$env:USERPROFILE\.codex" }
$target = Join-Path $codexHome 'skills\codex-engineering-workflow'
$resolved = (Resolve-Path -LiteralPath $target).Path
if (-not $resolved.EndsWith('\skills\codex-engineering-workflow')) {
    throw "拒绝删除非预期路径：$resolved"
}
Remove-Item -LiteralPath $resolved -Recurse -Force
```

项目里的 `AGENTS.md` 可能已经与其他规则合并，不应使用卸载命令自动删除。

## 安全与隐私

- 不包含 hooks、后台进程或无限循环；
- 不包含遥测、反馈上传、排行榜或网络提交；
- 不保存凭据和任务内容；
- 不自动安装依赖；
- 外部发布、生产操作和不可逆变更仍需要相应授权。

## 验证

仓库使用独立 Agent 压力场景验证简单任务、脏工作区、重复失败、生产删除边界和 UI 真实验证。测试过程只使用本地夹具，不连接生产环境。

## 许可证

本项目使用 [MIT License](LICENSE)。Superpowers 是独立项目，本仓库并非其官方分支或发行版。
