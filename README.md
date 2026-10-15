# 拼豆图纸技能包 · Pindoupic Bead Pattern（Agent Skill + MCP）

把任意图片变成能照着拼的拼豆图纸：带色号的格子图 + 各色用量 + 材料清单。
**在你本机出图，图片不上传，断网也能用。**

- 官网：<https://www.pindoupic.com/agent-tools?ref=mcp-registry>
- npm 包：`@javascribe/pindoupic`
- MCP 服务名（官方 Registry）：`io.github.qxshn/pindoupic-bead-pattern`

## 4 步接入（不懂代码也可以）

1. **复制下面这一行**

   ```bash
   npx -y -p @javascribe/pindoupic@latest pindoupic install
   ```

2. **发给你的 AI 助手**（Claude Code / Cursor / Codex / Qoder 等，直接粘进对话框，让它执行）
3. **等它跑完**（第一次约 3 分钟，要下载图片解码组件；之后 2 秒）
4. **说你的需求**，例如「把这张照片转成 80×80 的拼豆图纸，用 MARD 色板」

想手动接入、或只想接 MCP 不装技能，看下面的「高级用法」。

## 装成技能包（skills.sh）

```bash
npx skills add qxshn/pindoupic-perler-bead-pattern --agent '*' -y
```

`--agent '*' -y` 是给**非交互环境**（AI 助手、CI、脚本）用的：少了这两个旗标，这条命令会等键盘选宿主，
选不了就**什么都不装并退 1**（实测原文：`Interactive prompt required but stdin is not a TTY. Nothing was installed.`）。
人在终端里可以只跑 `npx skills add qxshn/pindoupic-perler-bead-pattern`，它会交互式问你要装到哪几家。

装完技能落进各宿主的技能目录（Claude Code `.claude/skills/`、Codex/Cline `.agents/skills/`、
Qoder `.qoder/skills/`），下次你说「拼豆图纸」它自己就用这套流程。

[![Install with skills.sh](https://skills.sh/b/qxshn/pindoupic-perler-bead-pattern.svg)](https://skills.sh/qxshn/pindoupic-perler-bead-pattern)

## 装成 Claude 插件（本地 stdio MCP）

在 Claude Code 里：

```
/plugin marketplace add qxshn/pindoupic-perler-bead-pattern
/plugin install pindoupic-bead-pattern@pindoupic
```

插件自带 `.mcp.json`，等价于手动写这条 MCP 配置：

```json
{ "mcpServers": { "pindoupic": { "command": "npx", "args": ["-y", "-p", "@javascribe/pindoupic@latest", "pindoupic-mcp"] } } }
```

> 本地 stdio 服务只在 Claude Code 与 Claude Desktop 的本地会话里可用，claude.ai 上要连远程服务才行。

## 高级用法（给人看的）

- 命令行出图：`npx -y -p @javascribe/pindoupic@latest pindoupic --image 照片.jpg --width 80 --palette MARD/291`
- MCP 服务：`npx -y -p @javascribe/pindoupic@latest pindoupic-mcp`（10 个工具，含出图、色板清单、图纸读写、反馈）
- 接入自检：`npx -y -p @javascribe/pindoupic@latest pindoupic doctor`
- 全部开关与 MCP 入参对照：[references/cli-mcp-matrix.md](./skills/pindoupic-bead-pattern/references/cli-mcp-matrix.md)

## 许可与商用

- 本包为 `AGPL-3.0-only`（见 `LICENSE`）。
- 需要在闭源产品/服务里集成，走商业授权：[COMMERCIAL-LICENSE.md](./COMMERCIAL-LICENSE.md)。

## 反馈

用完觉得哪里不对，直接让 AI 助手提交（`submit_feedback` 工具），或到 <https://www.pindoupic.com/jiaoliu>。
默认每台设备每天 2 条，超出会被限流——批量生成时不必每条都提交。
