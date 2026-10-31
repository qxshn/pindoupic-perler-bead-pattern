# pindoupic 拼豆图纸 Agent Skill

让 AI Agent（Claude Code、Qoder、Cursor 等支持 Agent Skills 的宿主）直接把图片生成拼豆图纸：
带色号/网格/材料清单的 PNG + 各色用量统计。

- 网站：https://www.pindoupic.com
- 生成引擎：npm 包 `@javascribe/pindoupic`（与网页版同一套算法的 Node CLI），Skill 首次运行时经 `npx` 自动下载
- 隐私：图片只在本机处理，不上传，文件名与路径也不上报。算法与色板都在包内 ⇒ **完全断网也能出图**；联网只做两件可选事：拉一次公开配置（拉不到就用包内默认值，不阻塞）与补报使用量（只含参数摘要）。`--offline` 一条请求都不发

## 安装

**所有宿主同一条命令**（复制给任何 AI 助手，或自己在终端跑，都一样这一条）：

```bash
npx -y -p @javascribe/pindoupic@latest pindoupic install
```

它做三件事，全是本机文件操作（秒级）：

1. **认出当前宿主**：优先读宿主自己给的环境变量（如 `QODER_CONFIG_DIR`），其次看进程链，最后才用目录存在性；认不出多家就不写、把该粘的配置打给你看（命令不用换，可加 `--config <路径>` 让它去写）。
2. **把 MCP 配置并进那家宿主自己的文件**：写入前先备份，只增改 `pindoupic` 这一个键，**别人已有的服务与未知顶层键逐字保留**；原文件不是合法 JSON 就拒绝动它并说清原因。
3. **把本技能放到位**：实体只有一份，住跨宿主中立位 `~/.agents/skills/pindoupic-bead-pattern/`，各宿主的技能目录放**软链**指过去（Windows 建不了链时退回复制，并当场说明"这会有第二份实体"）。

跑完它会点名**还差哪一步**——这一步归宿主，任何命令都替不了：Qoder 系会话内 `/mcp reload`，WorkBuddy 要重开会话并点一次「信任」，Claude Desktop 要完全退出再开。它不会宣称"已生效"。

配套两条：

- `npx -y -p @javascribe/pindoupic@latest pindoupic doctor` — 体检，把"包能跑 / 配置逐字段对不对 / 说明书在不在场"三类读数**分开**报（"包能跑"从来不等于"宿主已加载"）。
- `npx -y -p @javascribe/pindoupic@latest pindoupic install --list-hosts` — 打印宿主登记表全文（每家配置路径、顶层键名、生效方式、取证状态与来源 URL）。

> **各家的路径与字段不住在这份文档里**，住在 `cli/src/core/hostRegistry.ts` 这一处（唯一真值源）。
> 本文出现过的"装到某宿主某个目录"这类说法一律由它派生或与它对拍，门禁 `npm run check:host-install` 的 G12 会查。
> 已撤销的旧写法（往项目 `.ai/skills/` 复制、手抄 `node_modules/@javascribe/pindoupic/skill/...` 相对路径、Windows 上没有的 `cp -r`）在任何交付文档里都不得再出现。

> **没有技能目录的宿主**：同一本操作手册的精简版会随 MCP 握手的 `instructions` 字段送达（内容逐句照抄本 `SKILL.md`，门禁核对同源性），所以"只配了 MCP、装不了技能"的宿主同样拿得到选参纪律，不是只拿到工具。

> `cli/skill/` 是 `npm pack` 时由 `prepack` 从仓库根 `skill/` 复制出来的临时目录（已 gitignore），不要在那里改文件。

## 使用

安装后直接对话触发，例如：

> 把 ~/Downloads/cat.png 做成拼豆图纸，52 格，圆豆效果

Agent 会按 [SKILL.md](pindoupic-bead-pattern/SKILL.md) 的流程选参数、本地生成并回报各色号用量。
