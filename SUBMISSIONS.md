# 目录提交物料 · 逐家现成文案（机器生成，别在这里手改）

由主仓 `scripts/make-agent-discovery-artifacts.cjs` 从 `scripts/agent-discovery-identity.json` 现算产出。
字段名与长度区间取自各目录一手表单取证，逐条记在 `scripts/agent-directory-registry.json`。
**这一份只是"可以直接复制的文字"，不含任何已执行的提交动作**；表单、issue、PR 都由人自己过一遍再交。

## 投递前必须由你确认的两处

1. **作者位**：本文件按 GitHub 现主 `@qxshn` 出值，而材料包 §1 那行格式示例字面写的是 `@qxsh1`（疑为笔误）。投中文清单前确认一次。
2. **contact email**：mcpservers.org 的表单要留联系邮箱 ⇒ **（待填：身份真值源里没有 contactEmail，这一项归你——别拿站点公开邮箱凑，表单会把它当成可联系的地址）**

## mcp.so（网页表单，需登录）

- 入口：<https://mcp.so/submit?type=server>，顶部切到 **MCP Server**。
- `Repository URL` = https://github.com/qxshn/pindoupic-perler-bead-pattern
- `Name` = Pindoupic Bead Pattern（该字段 2–120 字符，现 22）
- 没有 logo／分类／描述字段 ⇒ 展示内容取自仓库，README 首屏就是它的展示面。

## mcpservers.org（网页表单）

- 入口：<https://mcpservers.org/submit>
- `server-name` = Pindoupic Bead Pattern
- `category` = **design**（用户 2026-10-15 裁；候选表：development／productivity／design／memory／other…，取错类影响别人在什么场景搜到我们）
- `server-description` = 把图片变成能照着拼的拼豆图纸：本地出图、带色号与材料清单，Claude Code / Cursor / Codex / Qoder 等助手可直接调用。
- contact email ⇒ 见上面第 2 条

## Cline marketplace（GitHub issue 模板）

- 入口：<https://github.com/cline/mcp-marketplace/issues/new?template=mcp-server-submission.yml>
- `repo-url` = https://github.com/qxshn/pindoupic-perler-bead-pattern
- `logo` = 本仓 `./assets/logo-400x400.png`（400×400 PNG，由主仓母图 `./backend/static/logo-icon-color-512.png` 现缩；提交时按目录要求给可访问的 raw 链接）
- 两条 checkbox 的凭据：README.md **或** 本仓 `llms-install.md` 单独读一遍就能装成功——这条必须**真跑过**再勾。
  可复跑的实跑器在主仓：`node scripts/verify-agent-cleanroom-install-1015.cjs`（假 HOME＋仓外临时目录，命令从这两份文件里现抽再执行，收尾断两个仓没被弄脏）。
  它的读数文件**按仓库规矩不进版本库**（prd/ 证据只留本地副本），所以别在这份物料里找一个现成的读数文件——要凭据就现跑。
- "服务器 stable" 这条由你判断。本文件**不写版本号**：线上 latest 与本地 `cli/package.json` 的现值不是一回事
  （本地改了版本而按裁定未发包，那种时候把数字抄进这份物料就是说谎），投递前用 `npm view @javascribe/pindoupic version` 现读一次。

## skills.sh ／ skillsmp.com（自动索引，没有提交通道）

- `npx skills add qxshn/pindoupic-perler-bead-pattern` 是 skills.sh 的克隆入口；徽章 <https://skills.sh/b/qxshn/pindoupic-perler-bead-pattern.svg> 现在返回 resource not found ⇒ 还没进索引，**徽章 200 不等于收录**。
- skillsmp 的 FAQ 要求 GitHub topic 标签 `claude-skills` 或 `claude-code-skills`：前者已在，后者是账号动作（本机 gh 未登录，须你手动加）。

## awesome-mcp-servers（fork 加一行，按字母序）

```markdown
- [qxshn/pindoupic-perler-bead-pattern](https://github.com/qxshn/pindoupic-perler-bead-pattern) 📇 🏠 - 图片转拼豆图纸，本地出图不上传。Perler bead pattern from any image, offline.
```

⚠ 徽标语义以该仓 README 图例为准：材料包记的一手读数是 📇＝语言、🏠＝本地；提交前对着图例再核一眼，别照抄我这行就递。

## VoltAgent/awesome-agent-skills（PR 加一行）

```markdown
- **[qxshn/pindoupic-bead-pattern](https://github.com/qxshn/pindoupic-perler-bead-pattern/tree/main/skills/pindoupic-bead-pattern)** - 图片转拼豆图纸，本地出图不上传。Perler bead pattern from any image, offline.
```

## ComposioHQ/awesome-claude-skills（PR 加一行）

```markdown
- [Pindoupic Bead Pattern](https://github.com/qxshn/pindoupic-perler-bead-pattern) — 图片转拼豆图纸，本地出图不上传。Perler bead pattern from any image, offline.
```

⚠ 它要求先在已有条目里找到一条「指向外部仓库」的先例才跟着写；"描述 ≤60 字符"我只取到转述、非逐字。找不到先例就换投别的。

## shishirui/awesome-claude-skills-zh（PR 或 Issue）

```markdown
**[pindoupic-bead-pattern](https://github.com/qxshn/pindoupic-perler-bead-pattern)** — 把任意图片变成能照着拼的拼豆图纸，本机出图、带色号与材料清单。**什么时候用：** 手上有照片或像素图，想直接得到能一颗颗拼的格子图纸、各色用量与采购清单。**作者：** [@qxshn](https://github.com/qxshn) **标签：** 拼豆 拼豆图纸 像素画 MCP Agent Skill
```

它的收录标准里有一条「活跃维护（近 6 个月有更新）」——建仓第一天就去提会卡在这条上。

## 明确不做的三家（已裁）

- **官方 MCP Registry**：丁-1 选乙，不做单 bin shim 入口包、不提交（`registryPlanned=false`）。
- **Smithery**：不为它新造 `.mcpb` 打包件（`smitheryPlanned=false`）；那是一家目录换一份新发布物＋多一份维护。
- **glama.ai**：它的横幅原文要求"仓库里要有 server 实现本体，不能只有文档"，而本仓是技能包（实现在 npm 包里）⇒ 先不投，除非愿意改主包结构（不建议，且不许把私仓转公开）。
