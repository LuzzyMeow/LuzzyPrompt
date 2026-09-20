---
name: luzzy-bilibili-notes
description: >
  Use when the user provides a Bilibili video link or BV number and wants its
  content extracted: "提取B站视频", "B站视频转文档", "把这个B站视频整理成笔记",
  "B站视频转文字", "帮我把这个视频转成文字", "抓这个视频的字幕",
  "帮我看看这个视频讲了什么", "这个视频讲了啥", "这个视频说了什么",
  "读一下这个视频", "这个视频讲了啥，不用写文件", "帮我读读这个视频",
  "把视频和评论一起整理出来", "抓一下这个视频的评论",
  "看看这个视频的简介和评论都说了啥", "这个视频下面大家都在讨论什么",
  "帮我提取这个视频的标题简介字幕和全部评论", "bilibili 视频 笔记",
  "BV号 转 文章", "BV 号提取内容", "b23.tv 短链提取", "字幕转 markdown",
  "视频内容整理成文档".
  Handles four retrievals by default: title, description, AI subtitle (ai-zh),
  and all public comments including replies. Reports them in the conversation by
  default; writes a Markdown file only when the user asks for a note or document.
  Also handles SRT parsing, transcript restructuring, and collection (multi-part)
  directory extraction.
  Do NOT use for bulk or site-wide crawling, bypassing WBI signing or risk
  control, accessing paid/OGV/P-charge content, downloading the media file
  itself, reverse-looking-up danmaku senders' identities, rendering notes to
  HTML, or building player components.
license: MIT
compatibility: >
  requires: python>=3.10, yt-dlp (pip install -U yt-dlp).
  Bilibili subtitles and comments require a logged-in session; anonymous access
  returns danmaku only. Preferred cookie source is the Tabbit browser CLI (see
  Luzzy §14.16); fallback is yt-dlp --cookies-from-browser, which fails while
  that browser is running.
metadata:
  version: "1.2.0"
  author: "鹿溪 (LuzzyMeow)"
  category: "content-extraction"
  maturity: "L2"
---

# B 站视频内容提取与报告

把 B 站视频的标题、简介、AI 字幕与全部公开评论提取出来，默认**在对话里报告**；
用户明确要笔记 / 文档 / Markdown 时才落盘。

## Default Retrieval Scope

默认抓取**四项**，缺一不可：

| # | 内容 | 取法 |
|---|---|---|
| 1 | 视频标题 | `scripts/fetch_video_info.py` |
| 2 | 视频简介 | `scripts/fetch_video_info.py` |
| 3 | 视频字幕（`ai-zh`） | yt-dlp（步骤 4） |
| 4 | 全部公开评论（含二级回复） | `scripts/fetch_video_info.py` |

用户只问「讲了什么」也要按四项取——这是默认范围，不是可选增项。
用户明确说只要某一项时才收窄。

## Default Output

**默认产出是对话里的报告，不是文件。**

- 用户没说要文件 → **只在对话里报告**，不建任何文件（见 `references/report-shape.md`）
- 用户明确要笔记 / 文档 / 转成 Markdown / 存下来 → 落盘，结构照
  [references/note-template.md](references/note-template.md)
- **拿不准就问一句**，不要默认生成文件

这条是硬规定：**未经要求不新建成品文档**（提示词 §7.1）。四项信息一个字不少地
报告出来，就已经完成交付。

## Workflow

1. 归一化输入为 BV 号。

   从用户给的链接中提取 `BV` 号（`https://www.bilibili.com/video/BV...`、
   `b23.tv` 短链、`/list/watchlater?...&bvid=BV...` 都含 BV 号）。短链与列表链接
   一律改写成标准形式 `https://www.bilibili.com/video/<BV号>` 再使用。

   Verify: 拿到规范的视频 URL，且 BV 号长度为 12 位。

2. 确定 cookie 来源，拿到登录态。

   按顺序尝试，第一个成功的即采用：

   - **首选 Tabbit CLI**（不受浏览器文件锁影响，且能取到 HttpOnly cookie）：
     在 `nodejs` 程序里取 `await context.cookies("https://www.bilibili.com")`，
     写成 Netscape 格式文本文件，路径落在临时目录。写法见
     [references/cookie-sourcing.md](references/cookie-sourcing.md)。
   - **回退 `--cookies-from-browser <chrome|edge|firefox>`**：仅在目标浏览器
     **未运行**时可用。
   - 两者都不可用 → 走步骤 8 的失败路径，不要凭空继续。

   **取 cookie 不需要导航**——`context.cookies()` 直接读，不要为了它去打开
   B 站页面，那会平白多出一个标签页。

   cookie 文件必须落在临时目录（`%TEMP%` / `$TMPDIR`），并在步骤 7 删除。

   Verify: `yt-dlp --cookies <文件> --list-subs` 输出中出现 `ai-zh` 行，而不是
   只有 `danmaku`。只有 `danmaku` 说明没拿到登录态，回到本步换 cookie 来源。

3. 抓取元数据与全部评论（标题 / 简介 / 评论三项）。

   ```bash
   python scripts/fetch_video_info.py "<BV号或视频URL>" \
     --cookies <cookie文件> --out "<临时目录>/bili_info.json"
   ```

   脚本一次取齐：视频标题、简介、UP 主、时长、分 P 目录、全部公开评论（含二级回复）。

   参数说明：
   - `--cookies` 必需，Netscape 格式 cookie 文件
   - `--out` 必需，输出 JSON 路径，落临时目录
   - `--max-pages` 可选，各接口翻页上限，默认 10
   - `--no-comments` 可选，只取元数据与分 P 目录

   接口契约、四个实测陷阱与完整性判据见
   [references/metadata-and-comments.md](references/metadata-and-comments.md)。

   Verify: JSON 里 `video.title`「`video.desc`」非空，`comments` 数组非空，
   且 `comments_summary` 给出了 `total_fetched` 与 `declared_count`。

4. 下载 AI 中文字幕。

   ```bash
   yt-dlp --cookies <cookie文件> "<视频URL>" \
     --write-subs --sub-lang ai-zh --skip-download --no-simulate \
     -o "<临时目录>/bili_output"
   ```

   `--no-simulate` 是必需的：带 `--print` 或单独用 `--skip-download` 时，
   yt-dlp 可能只打印而不落盘字幕文件。

   聚合（多分 P）视频默认会拉全部 P，**只处理用户指定的那一集**时加 `--no-playlist`
   并在 URL 上带 `?p=<集数>`。

   Verify: 临时目录出现 `bili_output.ai-zh.srt`，且非空。

5. 解析 SRT。

   用解析脚本把 SRT 转成纯文本：

   ```bash
   python scripts/parse_srt.py "<字幕.srt>" --timestamps
   ```

   - `--timestamps` 保留 `[mm:ss]` 标记，便于回视频核对；不要则省略
   - `--out <路径>` 写入文件；省略则打到标准输出

   读到文本后处理：

   - 合并被切碎的连续句为完整句子
   - 按语义划分章节（主题切换、步骤分节、概念定义、代码与配置）
   - 修正明显的语音识别错误（见 [references/asr-corrections.md](references/asr-corrections.md)）
   - 保留无法确定原词的表述，并标为存疑，不做推测性改写

   Verify: 拿到连贯的纯文本；专有名词的存疑项已记下。

6. 报告四项（默认）或落盘（仅在被明确要求时）。

   **默认路径——对话报告**：按 [references/report-shape.md](references/report-shape.md) 的结构，
   在对话里给出标题、简介、内容要点与评论情况，并带上抓取完整性说明。

   **按需路径——落盘笔记**：用户明确要笔记 / 文档时，
   结构照 [references/note-template.md](references/note-template.md)，
   文件名用 `{视频标题}.md`，落在用户指定位置（未指定则工作区）。

   Verify: 未要文件时**工作区没有新增文件**；要文件时笔记含标题、来源元信息、
   分节正文、评论汇总与「整理说明」段。

7. 收尾：关标签页 + 清临时文件。

   **关掉自己开的标签页**（用 Tabbit 时必须做）：

   ```bash
   <launcher> finish --task <任务名> --discard
   ```

   恰好一次，在任务结束时执行。它只关闭**任务自有**的标签页；用户的标签页不受影响。

   - **不要**去关用户自己的标签页——所有权已隔离，不要碰 `available` 状态的页
   - **不要**杀浏览器进程——CLI 没有退出命令，硬杀可能损坏 profile、丢用户会话。
     如果浏览器是**被本次任务拉起**的（任务开始前它没开着），如实报告这一情况并
     **询问用户**是否要关掉浏览器，由用户决定
   - **必须清掉** cookie 文件、临时 SRT 与临时 JSON——不管有没有落盘笔记

   Verify: cookie 文件已删除（`Test-Path` / `ls` 应为否）；用户的标签页仍在；
   任务清单里已无本任务（`diagnose` 的 `taskCount` 不含它）。

8. 失败路径（按症状处理）。

   | 症状 | 处理 |
   |---|---|
   | 字幕列表只有 `danmaku` | 登录态缺失。换 cookie 来源重试步骤 2；用户未登录 B 站时如实告知「该视频字幕需要登录态，暂无法提取」 |
   | 该视频无 `ai-zh` 字幕 | 视频未开启 AI 字幕。告知用户，并提议改用元数据与评论做简版报告（需用户同意） |
   | `Could not copy Chrome cookie database` | 浏览器正在运行、cookie 库被锁。改用步骤 2 的首选路径（Tabbit CLI） |
   | 实抓评论数少于页面显示 | **正常现象，不是遗漏**。差值来自已删除或隐藏的评论，它们仍计入总数但接口不再返回。如实报告 `comments_summary.gap`，不假装取全 |
   | 评论接口返回 `-352` / `-412` / `-509` 或 HTTP 403/412/429 | 触发风控。**立即停止请求**，告知用户，不重试、不换 IP、不换 UA、不调参数对抗 |
   | `--cookies-from-browser` 报文件锁 | 见上一行，改走 Tabbit CLI |
   | `yt-dlp` 未安装 | `python -m pip install -U yt-dlp` |
   | 聚合视频误抓全部 P | URL 加 `?p=<集数>`，命令加 `--no-playlist` |
   | 关标签页失败（finish 报错） | 不要反复重试、更不要改用杀进程。如实说明残留了标签页，让用户手动关 |
   | 浏览器本来是关着的、被本次任务拉起了 | **如实报告**并询问是否关掉浏览器；**不要自行结束进程** |

## Compliance Boundary

只处理**公开可见视频的标题、简介、字幕、公开评论与元数据**，且以用户自己的登录态访问。

禁止：批量或全站抓取、绕过 WBI 签名与风控、访问付费/OGV/充电专属内容、
下载视频本体、通过弹幕反查发送者身份。这些是被平台明确主张侵权的行为，
已有律师函先例（见 [references/compliance.md](references/compliance.md)）。

用户要求上述任一行为 → 拒绝并说明理由，改为提供公开数据的合法替代方案。

## Examples

Input: 「读一下这个视频 https://www.bilibili.com/video/BV1jHbP61EGH」
Output: 归一化 → 从 Tabbit CLI 取 cookie（不导航）→ 取到标题、简介、评论 31 条
（声明 31，差值 0）→ yt-dlp 取 `ai-zh` 字幕 → **只在对话里报告四项**，
不建文件 → `finish --discard` 关掉自己的标签页 → 删除 cookie 与临时文件。

Input: 「把这个B站视频整理成笔记 https://www.bilibili.com/video/BV11q8J6CEZS」
Output: 同上取到四项 → 用户**明确要笔记**，走落盘路径 →
重组为分十节、含元数据区与「整理说明」段的 Markdown → 落盘 `{视频标题}.md` →
关标签页、删 cookie 与临时文件。

Input: 「帮我看看这个视频讲了什么 https://b23.tv/xxxxx」
Output: 短链归一化为标准 URL → 因 Edge 正在运行导致 `--cookies-from-browser`
失败 → 改从 Tabbit CLI 取 cookie 成功 → `--list-subs` 确认有 `ai-zh` →
**对话报告**（不落盘）→ 收尾关标签页、清临时文件。

## Verify

- [ ] 输入已归一化为标准视频 URL，BV 号正确？
- [ ] 四项都取到了：标题、简介、字幕、全部评论（含二级回复）？
- [ ] `--list-subs` 确认到 `ai-zh`（而非只有 `danmaku`）才开始下载？
- [ ] 下载命令带 `--no-simulate`？
- [ ] 评论逐条核对过（每个 `rcount` == 实取子回复数）？差值如实报告？
- [ ] **默认路径下工作区没有新增文件**？要文件时笔记含元数据区、评论汇总与「整理说明」段？
- [ ] cookie 文件与临时 SRT、临时 JSON 已删除？
- [ ] **任务自有的标签页已关闭，且用户的标签页未被动过**？
- [ ] 浏览器是被本次任务拉起的时，已如实报告并询问（而不是杀进程）？
- [ ] 全程只读公开数据，未触碰 Compliance Boundary 的禁止项？

## Reference Files

| File | Load when |
|------|-----------|
| [references/report-shape.md](references/report-shape.md) | 默认路径：在对话里报告四项，需要结构与措辞 |
| [references/metadata-and-comments.md](references/metadata-and-comments.md) | 抓元数据与评论，或需要弄清接口契约、完整性判据、风控处置 |
| [references/cookie-sourcing.md](references/cookie-sourcing.md) | 需要取 B 站 cookie，或 `--cookies-from-browser` 报文件锁错误 |
| [references/note-template.md](references/note-template.md) | **仅当用户明确要笔记文件**，需要标准结构与措辞 |
| [references/asr-corrections.md](references/asr-corrections.md) | 重组字幕时，需要判断某个词是否为识别错误 |
| [references/compliance.md](references/compliance.md) | 用户要求批量抓取、绕过风控、付费内容，或需要说明边界依据 |

## Scripts

| Script | Run | Purpose |
|------|-----|---------|
| [scripts/fetch_video_info.py](scripts/fetch_video_info.py) | `python scripts/fetch_video_info.py <BV号> --cookies <文件> --out <JSON>` | 取标题、简介、UP 主、分 P 目录与全部公开评论 |
| [scripts/parse_srt.py](scripts/parse_srt.py) | `python scripts/parse_srt.py <字幕.srt> [--timestamps]` | SRT 转纯文本，合并碎片句 |
