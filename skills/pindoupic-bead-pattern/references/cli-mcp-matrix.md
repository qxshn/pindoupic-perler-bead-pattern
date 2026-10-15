# CLI 与 MCP 参数对照矩阵（唯一真值源）

> 本表由 `npm run check:cli-data` 的 §CM 组**逐行对拍**：CLI 侧比对 `pindoupic --help` 枚举出的旗标集合，
> MCP 侧比对**活体 `tools/list`** 返回的 `inputSchema`。新增一个参数而没进本表 ⇒ 门禁直接红，
> 不靠人记得改文档。取值口径以被测件为准，本表只负责"把差异说出来"。

实测依据（2026-09-29，本机 `cli/dist`）：同一张渐变夹具、`width=32`，
- CLI 默认产物 MD5 == MCP 默认产物 MD5（10-02 白底默认翻成「绘出」后，两边同值）；CLI `--transparent-bg` == MCP `transparent_bg:true`；
- **同一张图分别从 `--image` / `--image-url` / `--image-base64` 喂进来，三份产物 MD5 全等**（MCP 的三个入口同理），
  因为两个出口用的是**同一份**来源解析实现（`cli/src/core/imageSource.ts`）。
⇒ **算法、统计与默认值都一致**，剩下的只有写法形态差异（下表逐行写明）。

09-29 补齐批：MCP 加了 `cell_size` + 四个 `show_*` + `offline` / `no_telemetry`，CLI 加了 `--image-url` / `--image-base64`，
原先"仅某一边能做"的 7 行差异已全部收敛为同一能力。
09-29 第二批（同日晚）：MCP 再加 `palette`（一段式别名）与 `agent`（宿主归因）、响应加 `param_source`、
离线/关用量时不再发 `feedback_request`，`submit_feedback` 接受 `agent` 别名，离线状态下反馈通道直接拒绝发送——
同日第三拍：**MCP 的白底默认也调成与 CLI 一致**（默认不绘纯白格，用户裁定）⇒ 两出口之间**不再有任何行为差异**，
只剩写法形态（否定旗标 vs 正向布尔、色板一段式与两段式并存）。实测：同图同参、两边都不传白底开关，
产物 **MD5 逐字节相同**。

## 一、生图参数（`generate_bead_pattern` ↔ `pindoupic`）

| 能力 | CLI | MCP | 默认值 | 差异 |
|---|---|---|---|---|
| 图片来源：本机路径 | `--image` / `-i` | `image_path` | 三入口选一 | 一致 |
| 图片来源：URL | `--image-url` | `image_url` | 三入口选一 | 一致（09-29 补齐 CLI）；**只在本机下载**，图片不上传 |
| 图片来源：base64 | `--image-base64` | `image_base64` | 三入口选一 | 一致（09-29 补齐 CLI）；带不带 `data:image/...;base64,` 前缀都行 |
| 三入口优先级 | 路径 > URL > base64 | 同左 | — | 一致（同一份实现，字段优先级写在 `imageSource.ts`） |
| 图纸宽度（格数） | `--width` / `-w` | `width` | 站点下发 `default_width`（离线兜底 80），区间 10-500 | 一致（09-29 起 CLI 也读下发值；10-14 起 npm 的上限等于算法层那根，不再自设 200） |
| 色板 | `--palette 品牌/型号` | `palette_brand` + `palette_sub`，**或** `palette: "品牌/型号"` | `MARD/291` | 一致（09-29 起 MCP 两种写法都吃；`palette` 与 CLI 同形，给了它就覆盖两段式。只写品牌不写型号会回一条 warnings 并按默认型号补齐——实测 `palette:"MARD/24"` 与 `palette_brand:MARD,palette_sub:24` 产物逐字节相同） |
| 图片类型预设 | `--type` / `-t` | `image_type` | **都不套预设**（与 CLI 一致，09-28 修） | 一致 |
| 镜像翻转 | `--mirror` | `mirror` | 关 | 一致 |
| 合并零散色号 | `--simplify-colors` | `simplify_colors` | **默认不开启**（产品拍板） | 一致 |
| 合并阈值 | `--simplify-threshold` | `simplify_threshold` | 站点下发（兜底 0.005），越界夹取 0.001-0.1 | 一致（同一解析函数） |
| **色号预算** | `--target-colors N` | `target_colors` | 不启用（不传＝不合并） | 一致（09-30 M4-1，同一份实现 `core/paletteIntent.ts`）；N 非法（非整数/小于 1）两边都是 ⚠/warnings **且不启用**；N 低于 8 出下限告警；与 `--simplify-colors` 同时给 ⇒ **预算优先**，阈值那步不跑并出读数（决-D4），文档 `params_used.threshold_overridden:true` |
| **锁色号（预算层）** | `--lock-colors A,B` | `lock_colors` | 空 | 一致；只在预算里起作用，没给预算时两边都点名"这 N 色不参与任何计算"。⚠ 与 `pattern set --lock`（**逐格**锁，写进 `lock_cells`）是两件事，不要混用 |
| **排除色号** | `--exclude-colors A,B` | `exclude_colors` | 空 | 一致；挂在**量化上游**（从候选色板里拿掉）⇒ 产物中该色号命中必然为 0；把整块色板排空 ⇒ 报错并列出可用色号，不回退内置 |
| **指定合并** | `--merge-colors A->B,C->D` | `merge_colors` | 空 | 一致；与 exclude 的本质区别写死在判据里：源色的格子 **100% 变目标色**（不是落到最近邻）。传递收敛（A->B,B->C ⇒ 到 C）、有环报错、同源两个目标报错、与 exclude 矛盾报错。实测"把 A 槽位换成 B 的颜色再量化"与"删掉 A 落最近邻"是同一个解，所以合并走**硬替换**而不是改色板（实现注释里有这条推导） |
| **抖动算法** | `--dither`（none 或 floyd_steinberg 或 atkinson 或 bayer） | `dither` | 沿用 `--type` 预设（8 个预设里 7 个本就是 none） | 一致；旗标与预设同时在场时**旗标优先**且两边都出"预设的 dither 已被显式旗标覆盖"读数。价值是照片质感的控制权，不是降色数（实测打开 fs 反而色号变多） |
| **抖动强度** | `--dither-strength 0~1` | `dither_strength` | 沿用预设 | 一致；实测极限如实写进读数：默认 `solidRegionProtection` 会把**平坦区**的强度归零 ⇒ floyd_steinberg/atkinson 在普通图上两档强度可能产物相同，只有 bayer 可观测地变化（`check:color-intervene` 的 DR2/DR3 一条钉可观测、一条钉极限） |
| **重采样（无旗标）** | — | — | `smooth`；`--type` 取 lineart 或 pixelart 时走 `nearest` | 一致（09-30 决-D5 选 b：**由 type 决定，不重新开放旗标**）。包内覆盖表 `RESIZE_BY_TYPE`（`core/localData.ts`），**不改站点预设** `imageTypePresets.ts`；生效值写进文档 `params_used.resize_mode` 并出"按 type 走 nearest"读数。被撤销的旧旗标 `--resize-mode` 仍在撤销状态（任何交付文档不得再出现它） |
| **背景策略** | `--bg-mode`（none 或 white 或 dark 或 auto） | `bg_mode` | `none`＝现状一字不动 | 一致；**边界连通**抠图（从四条边扩、生长容差 ΔE2000≤2），与边缘连通无关的同色描边不会被抠掉——全局阈值实现会（`check:color-intervene` BG2 用孤立暗格夹具区分两种实现）。auto 归族后深底/白底给不同的那句 |
| **裁掉背景带** | `--trim-bg` | `trim_bg` | 关 | 一致；必须与 `--bg-mode` 同时给，单独给时两边都点名"一格未裁"（不静默无事发生）。裁完 canvas 收缩到内容包围盒，读数给出前后格数 |
| **按库存出图** | `--palette-file <path>` | `palette_file` | 不传＝用内置套装 | 一致；与 `--palette` / `palette_brand`+`palette_sub` **互斥**（两个真值源直接报错，不玩"后者覆盖前者"）。非法文件点名第几条（空 colors／重复色号带两条各自 hex／hex 不是 `#RRGGBB`），不回退内置。文档 `palette` 记成文件里的 brand/型号 |
| 输出格式 svg | `--format svg` | `format: "svg"` | 默认 png | 一致；**两个出口用的是同一份 SVG 字符串**（位图本就是它栅格化的），实测同参 SVG 与 PNG 声明尺寸一致、两出口文件逐字节相同 |
| 采购清单 | `--shopping-list <path>` | `shopping_list_path` | 不传＝不写 | 一致（09-30 M5-2，同一份 `shoppingListCsv()`）；颗数合计与网格实算豆数互印，纯白那行按生效的 transparentBg 分型；名称里的逗号/引号转义、以 = 开头的值降级成文本 |
| 按豆板分块 | `--board-tiling <n>` | `board_tiling` | 不传＝不分板 | 一致（09-30 M5-1）；块数 == ceil(宽/n)×ceil(高/n)，各板拼回与原图逐格一致；产物是 boards.json（板号→1 起含边界的行列范围）＋每板一张带板号的图 |
| 分板产物目录 | `--boards-dir <dir>` | `boards_dir` | 与网格文档/输出同目录 | 一致；非法 n（0 或非数字）CLI 直接拒绝出图并点名，MCP 回 warnings（一个请求了分板却什么板都没给的静默交付比报错更糟） |
| 豆子形状 | `--bead-shape` | `bead_shape` | `square` | 一致 |
| 输出格式 | `--format` | `format` | `png`；无 `sips` 的宿主降级为 PNG 并出读数 | 一致 |
| 输出路径 | `--output` / `-o` | `output_path` | CLI 由输入名派生（base64 入口派生成 `pattern_bead.png`）；MCP 落临时目录并回 `png_file`；两者都建父目录 | 一致（09-29 补齐 CLI） |
| 白底是否绘出 | `--transparent-bg`＝纯白格**不**绘出 | `transparent_bg: true`＝纯白格**不**绘出 | 两出口默认都**绘出纯白格** | 一致（10-02 用户裁定两边一起翻成绘出；MCP 的 `transparent_bg` 默认 false。旧 CLI 旗标 `--keep-bg` 仍在，但等于默认值并会出一句点名读数） |
| 旧旗标（等于默认） | `--keep-bg`＝白色格子绘出＝**本次默认值**：生成时给不给产物相同，只出一句点名读数 | 无对应入参（MCP 侧不传即默认） | 不传 | 一致（10-02 默认翻转后它在**生成路径**上退化成"显式声明默认"）。⚠ 一条例外要说清：`pattern render` 重画一份记着「不绘出」的旧文档时，它会覆盖文档里那一项 ⇒ 产物会变，同批会打一句点名读数（10-02 审查实测补上，此前这条路径既改图又沉默）。`check:cli-exhaustive` 用 via=name＋expectSame 钉住生成路径不许静默、也不许偷偷改图 |
| 每格像素 | `--cell-size` | `cell_size` | 站点下发 `png_cell_size`（兜底 50）；画布长边受限时引擎自动降档并出读数 | 一致（09-29 补齐 MCP）；填了用不上（0/非数字）时**两边都说一句**，不静默降回默认 |
| 网格文档导出 | `--emit-pattern <path>` | `export_grid_path` | 关（不传就不写） | 一致（09-30 M1 同批加）；两边写的是**同一份实现**产出的同一结构（`core/patternFile.ts`），逐字节相同 |
| 读单格 / 读区域 | `pattern get --cell 行,列` / `--region 行1,列1,行2,列2` | `get_pattern_grid` 的 `cell` / `region` | 坐标 **1 起** | 一致；坐标语义与图纸四周标尺同序（引擎标尺打的是 `r+1`/`c+1`），越界两边都点名报错而不裁剪 |
| 从网格重画 | `pattern render <p.json>` | `render_pattern` | 未显式给的开关沿用文档 `render` 块 | 一致；**绝不重新量化**，实测原图与两条渲染路径 MD5 逐字节相同 |
| 重画时出交付物 | `pattern render` 的 `--shopping-list` / `--board-tiling` / `--boards-dir` / `--format svg` | `render_pattern` 的 `shopping_list_path` / `board_tiling` / `boards_dir` / `format:"svg"` | 不传＝不写 | 一致（09-30 M5 遗留清理批）：四处共用 `emitDelivery()` 这一份实现，实测同一份 p.json 出图那次与重画那次的 CSV 与 boards.json **逐字节相同**（判据在 check:delivery-m5 的 R2/R3/R4） |
| 不印色号文字 | `--no-color-codes` | `show_color_codes: false` | 印 | 一致（09-29 补齐 MCP）；形态相反：CLI 是否定旗标，MCP 是正向布尔、默认 true |
| 不画网格线 | `--no-grid` | `show_grid: false` | 画 | 一致（同上） |
| 不画四周标尺 | `--no-rulers` | `show_rulers: false` | 画 | 一致（同上） |
| 不要材料清单 | `--no-materials` | `show_materials: false` | 要 | 一致（同上） |
| `show_*` 传成字符串 | 不适用（旗标即开） | `show_grid: "false"` | — | MCP 不猜意图：**按默认保持显示**并回一条 `warnings`，产物与默认全等（实测 MD5 相同） |
| 内联 base64 返回 | —（不适用，CLI 只写文件） | `include_image` | 不内联 | ⚠ 仅 MCP（CLI 的载体是磁盘文件） |
| 色号用量表 | `--colors`（stdout 表格） | 响应 `color_stats` | — | 同一数据，两种载体 |

## 二′、文字转图纸参数（10-13 新增；`generate_bead_pattern` ↔ `pindoupic --text`）

> 这一族把「打字出图纸」接到 npm 出口：引擎、字体表、板面量程都从网页那份真值源现取
> （`frontend/src/utils/textPatternGenerator.ts`、`frontend/src/utils/textFontLoader.ts`、`frontend/src/data/patternSize.ts`），
> CLI 与 MCP 共用 `cli/src/core/textPattern.ts` **一份**实现，所以同参两出口的网格文档除 `pattern_id` 外逐字节相同。

| 能力 | CLI | MCP | 默认值 | 差异 |
|---|---|---|---|---|
| 文字内容 | `--text <文字>` | `text` | 无默认（给了就进文字流） | 一致；**与三个图片入口互斥**，同时给会整次拒绝并点名（10-13 决-C＝乙） |
| 字体 | `--font <key>` | `font` | `shuhei` | 一致。可选键与各自下限**从 `textFontLoader.ts` 的 `TEXT_FONTS` 现取**（两出口与 `--help` 三处同源，写死清单就是下一个漂移点）；`sys`（系统黑体）在 npm 出口不可用并点名原因 |
| 自带字体文件 | `--font-file <path>` | `font_file` | 不给＝用内置那款 | 一致（同一份 `localFontFile()`）：给了它就走**零字体请求**那条路（不查内置表、不下载），文件解析不动就点名那份路径并整次拒绝，**绝不换成内置字体顶一张**；与 `--font`／`font` 同时给＝自带文件优先，但两出口都会点名被取代的那款内置字体（`advisories` 里那句「内置字体「seal」没有生效」）。字高下限按 16 格（内置轮廓档最宽松那档）——这是保守工程线，**不是对你这款字体的实测** |
| 字体清单（纯查询） | `--list-fonts` | `list_fonts: true` | — | 一致（同一份 `listTextFonts()`）：key／名称／点阵还是轮廓／最低字高／篆书可写字数／本机有没有留档，**不出图、不发任何请求**（留档状态只 stat 本地目录），断网照样答；与 `--text`／三个图片入口混给会被拒（纯查询不猜意图） |
| 预检（只跑判据不出图） | `--check-text` | `check_text: true` | — | 一致（**不是另写一份检查**，是同一条判据链跑完不渲染不落盘）：退出码 0／`success:true` 的语义是"这组参数能出图"，不是"图已经出了"；两出口都把出图那一发**会说的话**原样说一遍（同一个 `patternAdvisories`），并各说一句"预检模式没有写任何文件"。没给 text 单独给这一颗＝拒绝并指路，**不会**报成 "需要一张图片" |
| 字高（格/字） | `--char-height <n>` | `char_height` | 24 | 一致。低于该字体下限会拒绝并说出下限；与网页 DEFAULT_UI 的 24 由 `check:npm-textflow` 的 D 族逐字对拍 |
| 字距 | `--letter-gap <n>` | `letter_gap` | 7 | 一致 |
| 行距 | `--line-gap <n>` | `line_gap` | 7 | 一致 |
| 对齐 | `--align <side>`（left / center / right） | `align` | center | 一致 |
| 描边种类 | `--stroke <kind>`（none / char / whole / cloud） | `stroke` | cloud | 一致 |
| 描边宽度 | `--stroke-width <n>` | `stroke_width` | 1 | 一致（0-6 越界点名） |
| 字面色号 | `--face-color <code>` | `face_color` | H2 | 一致；色号不在本色板内会点名并给出可用形如 A1/A2/A3（网页端静默兜到第一颗，npm 不兜） |
| 描边色号 | `--stroke-color <code>` | `stroke_color` | R1 | 一致 |
| 云朵三色 | `--cloud-color a,b,c` | `cloud_color` | F14,E12,E6 | 一致（顺序＝内环/中环/外环，不是三个独立开关） |
| 底色 | `--bg-color <code>` 或 `--bg-color none` | `bg_color` | `none`＝透明底 | 一致；传 none 与不给同形（整张铺底与文字块描边是两件事） |
| 板宽 | `--width` / `-w` | `width` | 不给＝按文字块自动贴合 | **两个量，且 200 的身份变了**：文字流不再设人为上限（10-14 用户裁定「取消上限，只给建议」），`patternSize.ts` 的 `TEXT_BOARD_MAX_WIDTH=200` 现在只是**常见底板尺寸＝建议档**，拦人的是每边 1700 格那根防死机红线（实测档）；图片流仍是 500（算法层天花板） |
| 板高 | `--board-height <n>` | `board_height` | 不给＝自动贴合 | 一致（MCP 用 board_height 之名，免与图片流「高度按原图比例推导」混成一件事）；上限口径同上一行（10-14 起不设人为格数上限，红线每边 1700） |
| 锚点 | `--anchor <行,列>` | `grid_anchor` | 不给＝板内自动居中 | 一致；算式 `(W-bw)>>1` / `(H-bh)>>1` 与 iOS `pdGenerateText`、H5 `useTextPattern.ts` 同源 |

实测依据（10-13，本机 `cli/dist`）：同一串文字参数分别从 CLI 与 MCP 喂进来，PNG **逐字节相同**、
网格文档 `content_fingerprint` 相同、采购清单 CSV 相同；图片流在改动前后 PNG／CSV／分板 16 件逐字节相同
（`p.json` 只有 `pattern_id` 变，那是每次运行的身份）。
⚠ 文字流的字体文件第一次用时从站点取一次（带 `--offline` 且未留档 ⇒ 点名失败），这是本包**唯一**一处
「断网做不到」的入口，由用户 10-13 决-B＝丙 拍定，不是实现上的偷懒；取到后存 `~/.pindoupic/fonts`，之后离线可用。

## 二、联网与反馈

| 能力 | CLI | MCP | 说明 |
|---|---|---|---|
| 提交意见 | `feedback --rating --note --agent` | `submit_feedback{rating,note,agent_name,context,offline}` | 规范名是 `agent_name`；**09-29 起 `agent` 作为别名同样生效**（旧行为：传 `agent` 不报错也不采纳，落库归属被静默换成 `pindoupic-mcp`＝假成功）。两者都给时以 `agent_name` 为准。`context`（机读上下文，服务端截 480 字、不得含图片内容或路径）**10-14 才补进 schema 声明**——handler 一直在读它并原样落库，而 10-14 把"未知入参"从"警告后继续"改成"整次拒绝"之后，漏声明会让正常的 `context` 被自己拒掉 |
| 完全离线 | `--offline` | `offline: true` | 一致（09-29 补齐 MCP）。语义＝**本站的配置/用量/反馈端点一个请求都不发**，渲染参数用包内默认值；用量仍记进本地队列，下次联网补报。⚠ 唯一保留的机制差异：CLI 是进程级（一条命令一个进程），MCP 是**本次调用级**（常驻进程不能设全局，否则一次 `offline:true` 会把之后所有调用永久锁在包内默认值上） |
| 不发送用量 | `--no-telemetry` | `no_telemetry: true` | 一致（09-29 补齐 MCP）；连本地待补报队列都不写 |
| 标注调用宿主 | `--agent <名>` | `agent`（生图入参） | 不传：CLI 记空、MCP 记 `mcp-host` | 一致（09-29 补齐 MCP；此前 MCP 侧永远是 `mcp-host`，多宿主共用一个包时分不清谁在调） |
| 生效参数来源 | stderr 的 `⚙ 渲染参数生效值来源` | 响应字段 `param_source`（`server`/`builtin`/`offline`） | — | 一致（09-29 补齐 MCP）：同版本同输入要能解释"为什么某天结果变了"，两个出口都得说清参数是谁给的 |
| 每次出图后的"征集意见" | 打 `💬 …` ＋ `FEEDBACK_REQUEST` | 响应字段 `feedback_request` | 默认都发 | 一致：`--offline`/`offline:true` 或 `--no-telemetry`/`no_telemetry:true` 时**两出口都不再发这条要求外发的指令**（09-29 补齐 MCP；旧行为是 CLI 抑制、MCP 照发） |
| 离线时提交意见 | `feedback --offline` | `submit_feedback{offline}` | 布尔；非布尔**拒绝执行** | 一致（09-29 进程级 `PINDOUPIC_OFFLINE`；**09-30 消费侧批补上调用级 `offline`**）：离线状态下反馈通道**在发之前就拒绝**并回一句人话（拦在唯一实现 `cli/src/core/feedback.ts` 里，两出口同语义）。旧形态是 MCP 只读进程级 ⇒ 在 `generate` 里用过 `offline:true` 的宿主把同一习惯带过来会**静默外发** |
| 环境变量（两出口通用） | `PINDOUPIC_OFFLINE=1`、`PINDOUPIC_TELEMETRY=0`、`PINDOUPIC_QUIET=1`、`PINDOUPIC_API=<url>` | 同左 | 注意关闭用量的是 `PINDOUPIC_TELEMETRY=0`，**不是** `PINDOUPIC_NO_TELEMETRY` |
| 取 URL 图片那次下载 | 算 | 算 | 不受 `--offline` / `offline:true` 约束——那是你自己指定的输入，不是本包的对外请求 |

## 三、给 Agent 的用法结论

- 现在**两边都能做全部画面可调项**：关网格/色号/标尺/材料清单、指定每格像素、喂 URL 或 base64。
  形态差别只有两条：CLI 用否定旗标（`--no-grid`），MCP 用正向布尔默认显示（`show_grid: false`）；
  CLI 的色板是一段式（`--palette MARD/291`），MCP 是两段式（`palette_brand` + `palette_sub`）。
- 白底：**两边默认都绘出**（10-02 一起翻）。想让纯白格不绘出 → CLI 传 `--transparent-bg`、MCP 传 `transparent_bg: true`。
- 两出口的"这次到底绘不绘白底"都会写在 CLI 的 `ℹ` 行或 MCP 响应的 `advisories` 里，照原样转达用户即可。
- 隐私/离线：CLI 用 `--offline` / `--no-telemetry`，MCP 用同名入参（`offline` / `no_telemetry`），不必再去设环境变量。
- **名字不认识的开关/入参：两个出口都整次拒绝**（10-14 起，与 CLI 对齐；旧行为是"警告后继续"＝给了没生效还照样出图）。
  CLI 退出码非 0 且不写任何文件，MCP 回 `isError`；回执逐条点名哪个名字不存在、最接近的是哪一个、不给它少哪份交付物。
  最常见的一击是把对面出口的写法照抄过来（CLI 的 `--preview` ↔ MCP 的 `preview_path`）。
  布尔值发成字符串仍是"按默认执行＋写进 warnings"，不猜本意；下划线开头的协议保留位（`_meta`）不拒绝（否则带它的宿主每次调用都失败），照常忽略并写进 warnings。

## 二·补 逐格改写与观察面（M2/M3，2026-09-30）

| 能力 | CLI | MCP | 默认值 | 差异 |
|---|---|---|---|---|
| 出图前诊断 | `analyze <图>` / `--image` | `analyze_input` 的 `image_path` / `image_url` / `image_base64` | 按 `width`（默认 80）探针采样 | 一致（同一份 `cli/src/core/analyze.ts`）；四组读数逐字段全等 |
| 诊断的格数与色板 | `--width` `--palette` | `width` `palette` | 同上 | 一致 |
| 报告出口 | `--report <path>` 或 `report p.json` | `generate_bead_pattern` 的 `export_report_path`；单独出报告用 `report_pattern` | 不传＝不写 | 一致（同一份 `reportFile.ts`）。**必须有源图**：CLI 的 `--image`、MCP 的 `image_path`，缺了就拒绝而不是拿网格自己当参照 |
| 报告核对（旧报告配新网格） | `report p.json --check report.json` | —（CLI 专属：这是回归/实验用的核对通道） | — | MCP 没有对应入参；需要核对时走 CLI |
| 单格选择器 | `pattern set --cell 3,4` | `edit_pattern_grid` 的 `cell` | 四选一，互斥 | 一致（坐标 1 起） |
| 区域选择器 | `pattern set --region 1,1,4,5` | `region` | 同上 | 一致 |
| 整色号选择器 | `pattern set --where A13` | `where` | 同上 | 一致 |
| 边界连通块 | `pattern set --flood 3,4` | `flood` | 同上 | 一致（**4-邻域同色号**，不是全局色键阈值——全局实现会把深色描边一起抠走） |
| 目标色号 | `--to B22`（擦成空格传 `--to ""`） | `to` | 必填（除规则模式） | 一致：表外色号一律**报错并列出可用色号** |
| 规则链 | `--rules "A->B,B->C"` | `rules` | 无 | 一致（传递收敛、顺序无关、有环与一源多目标都报错） |
| 锁格 | `--lock "30,20;31,20"` / `--unlock` | `lock` / `unlock` | 无锁 | 一致：锁集合序列化进 p.json，后续任何重算逐格不动 |
| 只算不写 | `--dry-run` | `dry_run: true` | false | 一致：判据是"产物文件未被创建"，不是"没报错" |
| 改写落盘 | `-o / --output`（不传＝就地写回） | `output_path`（不传＝就地写回） | 就地 | 一致 |
| 动作回执 | stdout 的 JSON＋`--receipt <path>` | 响应里的 `receipt` 字段 | 总是给 | 一致（除落盘路径外逐字节相同）。无源图时 `de:null` 并说明原因，**不给 0** |
| 逐格比对 | `pattern diff a.json b.json` | `diff_pattern_grids` 的 `pattern_path` / `other_pattern_path` | 变更清单前 500 格 | 一致；格数不同都**拒绝比较** |
| 读回网格 | `pattern get` | `get_pattern_grid` | — | 一致 |
| 重画 | `pattern render` | `render_pattern` | — | 一致 |
| 缩小预览 | `--preview <path>` ＋ `--compact-preview <px>` | `preview_path` ＋ `preview_max_side`；`pattern render` 侧同名 | 长边 800 | 一致。**预览只用于观察**，逐格数据一律走 p.json（反向判据：删掉预览文件后报告数字一字不改） |
| 源图与图纸并排 | `--compare <path>` | `compare_path`；`pattern render` 侧 `compare_path` ＋ `compare_source_path` | 不传＝不写 | 一致（宽度=两图宽之和+16px，几何可量） |
| 网格文档出口 | `--emit-pattern <path>` | `export_grid_path` | 不传＝不写 | 一致（09-29 起） |

## 四、接入安装链（CLI 专属；MCP 出口没有对应入参，也不该有）

`install` / `doctor` 是"把本包接进宿主"的动作，执行者是人和 agent 的终端，不是 MCP 会话——
MCP 进程不能自己配置宿主自己，所以这一族旗标**只有 CLI 有**，下表逐条写明为什么 MCP 侧不适用。
命令串只有一条（`oneCommand()` 住在 `cli/src/core/hostRegistry.ts`），差异由包内的宿主登记表消化。

| 能力 | CLI | MCP | 默认值 | 差异 |
|---|---|---|---|---|
| 一条命令装好接入（认宿主＋并配置＋放技能） | `install` | —（不适用：MCP 进程不配宿主） | 默认执行前三步 | CLI 专属；判据见 `check:host-install` 的 G1~G15 |
| 体检：包能跑／配置逐字段对不对／说明书在不在场 | `doctor` | —（不适用，同上） | 只读，不写盘 | CLI 专属 |
| 点名要配哪家宿主 | `--host <id>` | — | 不传＝自动探测 | CLI 专属；`--list-hosts` 可查全部 id |
| 未知宿主的配置位置 | `--config <路径>` | — | 不传＝兜底只打印 | CLI 专属（决-C：命令不变，加参数即可） |
| 一次配全检测到的多家 | `--all` | — | 关（多家命中且不点名⇒一个都不写） | CLI 专属 |
| 只装一半 | `--only mcp` 或 `--only skill` | — | 两样都做 | CLI 专属；非法取值当场退出，不猜意图 |
| 只算计划不落盘 | `--dry-run` | — | 关 | 与 `pattern set --dry-run` 同名同义（判据是"目标文件未被创建"） |
| 机读结论 | `--json` | — | 关（人读走 stderr） | CLI 专属；给 agent 用，别猜退出码 |
| 撤销 | `--undo` | — | 关 | CLI 专属；按备份字节还原，备份不在就不动并点名 |
| 打印宿主登记表 | `--list-hosts` | — | 关 | CLI 专属；分母从这里取，文档不许手抄 |
| 首跑是否活体自检 | `--verify` | — | **默认不跑**（决-G：首跑要下载约 83MB，实测 185 秒，会被宿主掐断） | CLI 专属；跑则真起 `tools/list` |
| 只问"我该复制哪一条" | `--print-command` | — | 关 | CLI 专属；一个字节都不写 |
| 逐组合枚举命令（给门禁打靶用） | `--all-combos` | — | 关 | CLI 专属；配 `--print-command` 用，去重必须=1 |
| 上下文参数（只为证明命令不随它变） | `--os` 与 `--shell` | — | 不影响输出 | CLI 专属；`--os` 取 darwin、win32、linux 三值，`--shell` 取 bash、zsh、powershell、cmd 四值。`oneCommand(_ctx)` 故意接收却不使用，G1 与 M1/M10 各打一种分叉形态 |
