# llms-install.md — install Pindoupic Bead Pattern (Agent Skill + MCP server)

中文：本文件与 README.md 等价，**命令逐字相同**，两份都由主仓的 `scripts/make-agent-discovery-artifacts.cjs`
从唯一身份真值源 `scripts/agent-discovery-identity.json` 生成 ⇒ 请在主仓改，别在这里手改。

**What it does.** 把图片变成能照着拼的拼豆图纸：本地出图、带色号与材料清单，Claude Code / Cursor / Codex / Qoder 等助手可直接调用。
Rendering happens **on your machine**; your image is not uploaded, and it still works with no network.

## Option 1 — one command (recommended)

Installs the skill into every assistant host it can detect (Claude Code / Cursor / Codex / Cline / Qoder …):

```bash
npx -y -p @javascribe/pindoupic@latest pindoupic install
```

## Option 2 — MCP server only (local stdio)

```bash
npx -y -p @javascribe/pindoupic@latest pindoupic-mcp
```

Same thing as a config block (key `pindoupic`):

```json
{
  "mcpServers": {
    "pindoupic": {
      "command": "npx",
      "args": [
        "-y",
        "-p",
        "@javascribe/pindoupic@latest",
        "pindoupic-mcp"
      ]
    }
  }
}
```

The server exposes 10 tools: analyze_input, diff_pattern_grids, edit_pattern_grid, generate_bead_pattern, get_pattern_grid, list_image_types, list_palettes, render_pattern, report_pattern, submit_feedback.

## Option 3 — skill only, straight from this repository

```bash
npx skills add qxshn/pindoupic-perler-bead-pattern
```

## Option 4 — Claude Code plugin

```
/plugin marketplace add qxshn/pindoupic-perler-bead-pattern
/plugin install pindoupic-bead-pattern@pindoupic
```

## Verify the install

```bash
npx -y -p @javascribe/pindoupic@latest pindoupic doctor
```

Then ask the assistant: "turn this photo into an 80 x 80 perler bead pattern with the MARD palette".
A headless one-shot also works:

```bash
npx -y -p @javascribe/pindoupic@latest pindoupic --image photo.jpg --width 80 --palette MARD/291
```

## Rules for whoever installs this

- **Keep the `@latest` version anchor** on every `npx -y -p` call in this repository's instructions.
  Dropping the anchor lets a cached older build answer instead — that older path was the cause of a real
  incident ("HTTP 404 无法生成图纸") even though the current version had been published.
- Do not call the site to fetch palettes or presets: the package ships them locally.
- Privacy switches: `--offline` (no requests at all) and `--no-telemetry` (no usage report) both exist.
- Licence: `AGPL-3.0-only` (see LICENSE). Closed-source integration: see COMMERCIAL-LICENSE.md.
