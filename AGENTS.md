# Luzzy 维护指南

面向维护本仓库的 Agent。仓库身份与规则正文见 `README.md` 与 `prompt/Luzzy.md`；本文件只讲**怎么改、改哪里、什么不许动**，以及接入各家 harness 时要用的路径。

---

## 一、仓库是什么

Luzzy（鹿溪）给 Agent 用的一套行为契约——定位是**综合智能体**，编码是其中最常用的一类任务，但不是唯一。分两层：

- **规则层**：`prompt/Luzzy.md` —— 全部规则与红线的唯一载体，注入为 system prompt
- **细则层**：`skills/` —— 命中特定场景才加载的操作细则，不复制规则

**为什么是这两层而不是一层**：早先试过两个极端。先是「常驻提示词 + 22 个按需 skill」，但 §1.1 的必读清单与各 skill 正文里的清单是同一份数据的两个副本（实测 14 个 skill 重复了提示词里的链接），两处并存必然漂移。后来改成纯单一文件，清单只有一个事实源，但代价是全部细则常驻，提示词涨到 24k 且没有地方放长流程。

现在的分工是：**规则只住在提示词里，skill 只装「怎么做」**。清单仍然只有一个事实源（§1.1），skill 正文不重复它——所以不会回到漂移的老路。

**权威来源**：`https://github.com/LuzzyMeow/LuzzyPrompt`（本仓库；2026-09-16 前名为 Luzzy）。

## 二、目录与文件

```
LuzzyPrompt/
├── prompt/Luzzy.md            全部规则（唯一载体，注入为 system prompt）
├── skills/                    配套 skill（按需加载的操作细则，含 21 个 luzzy-roster-*）
│   ├── README.md                  索引：用途、安装、来源与许可
│   ├── luzzy-skill-architect/     创建 / 审计 / 融合 Agent Skills（Apache-2.0）
│   ├── luzzy-aoci-index/          项目认知索引：接手项目必过的认知门（MIT）
│   ├── luzzy-skill-meihuayishu/   梅花易数技能家族（MIT，自带维护宪章 AGENTS.md）
│   ├── luzzy-bilibili-notes/      B 站视频提取与报告（MIT，默认不落文件）
│   └── luzzy-zip-password-recovery/  ZIP 压缩包密码恢复（MIT，挂靠 roster-reverse）
├── evals/                    门评测层：§1.1.8 覆盖审计 + 回执核验（Python，仅标准库）
├── AGENTS.md                  本文件
├── README.md                  门面文档
├── LICENSE
└── .gitattributes
```

**没有** `package.json` / `pyproject.toml` 之类的清单文件，也**没有构建脚本**——本仓库不是软件项目，是提示词加一组 skill 文本。但 `skills/` 与 `evals/` 里**有可执行校验**（Python，仅用标准库）：改动对应技能或 §1.1 触发口径时必须跑通，见第四节。

## 三、改动流程

1. 读 `README.md` 与 `prompt/Luzzy.md`，确认要动的是哪一节；动 `skills/` 下的技能前，先读该技能的 `SKILL.md`（梅花易数还要读它的 `AGENTS.md` 宪章）
2. 改对应文件
3. 按第四节的清单人工核对（尤其 `§` 交叉引用与数字）；改了 §1.1.8 触发口径就跑 `python evals/eval-gate-coverage.py`（门槛 ≥80%）
4. 按实测更新 `README.md` 的「提示词预算」表与徽章数字（口径见第五节）
5. 推送走 SSH：`git remote -v` 两行都应是 `git@github.com:` 开头

改完最容易出的三类错是**交叉引用悬空**（改了章节编号没改引用）、**README 数字漂移**（改了提示词没重测）、**清单与 skill 路径不一致**（改了目录名没改 §1.1）——第四节给了对应的核对方法。

**改动时要守的一条**：规则只有一个事实源。如果某条规则在提示词里出现两次，删掉一份，改成引用（`见 §N`）；如果某条规则同时出现在提示词和某个 skill 里，**规则留在提示词，skill 里删掉**。

### 更名类操作（改仓库名 / 文件名 / 目录名）

更名不是改一处就完事——名字散落在很多地方，漏掉任何一处都会留下死链或加载失败。**执行顺序**：

1. **先改权威处**：仓库名（`gh repo rename`）、主文件（`prompt/Luzzy.md`）
2. **立刻核验 remote**：`gh repo create` / `gh repo rename` 这类命令会把 remote 改成 HTTPS——执行后跑 `git remote -v`，出现 `https://` 就按 §1.2 纠正回 SSH
3. **在受影响范围内全量搜旧名**，逐个判定「该改」还是「该留」：

   | 该改 | 该留 |
   |---|---|
   | 文档正文、路径引用、安装命令 | **历史会话缓存**（`~/.dsh/storages/`）、变更登记里的历史记录 |
   | harness 配置、预设名与目录名 | 有意保留的演进说明（如「早先版本是……」） |
   | 仓库描述、topics、README 徽章链接 | 第三方 skill 自身的历史文件（如各技能的 `README.md` 说明其来源） |

4. **harness 侧同步**（本机 DSH 为例，其他 harness 见第七节）：预设目录名、`preset.yml` 的 `name` 与 `description`、`settings.yaml` 的 `agent-presets.default`、`agent.cordis.yml` 里的注释——四者都要跟。改完跑一遍第六节的 DSH 同步与回验流程。
5. **回归核对**：按第四节整表过一遍，重点是交叉引用、清单指向、README 数字。

> **`preset.yml` 的 description 是最容易漂移的一处**——它不在任何自动校验范围内，描述会长期停留在旧架构。改提示词时顺手看一眼它是否还说得对。

## 四、改完怎么核对

没有全仓库的自动门禁，改完按这张表人工过一遍。每项都给了「怎么快速验」：

| 类别 | 核对什么 | 怎么验 |
|---|---|---|
| 结构 | 一级标题齐全（正文「鹿溪 · Luzzy」+「导航」+ 〇 至 十四 + 附录 A/B） | 搜 `^# ` 列出所有一级标题对一遍 |
| **人设层** | `〇` 节含全部人设要素，且三条边界在位 | 搜「防漂移锚点」/「颜文字白名单」/「行为协议」/「做事协议」/「硬性禁忌」；再搜三条边界：「人设是语气层」/「限于红线之内」/「只管对话输出」。**条数写进了 README**（行为协议 6 / 做事协议 6 / 硬性禁忌 7 / 颜文字 8 类），改完要同步 |
| 清单 | §1.1 的十七类齐全（每类一个 roster 指向），条数列与各 roster skill 的明细对得上 | 数主表行数（应 17）；再逐个打开被指向的 `skills/luzzy-roster-*/SKILL.md`，核对「条数」列与 skill 内子项表行数一致；改条数时两侧同步 |
| **设计类子项** | 主清单仍是十七类（子项**往下分裂**，不占顶层）；基线与框架子项的**叠加**关系写清楚 | 数主清单表格行应仍为 17；确认 §1.1.6.1 三行各指向一个 `luzzy-roster-design-*` skill；搜「叠加」确认「基线照读」的说法在位；§14.1 的框架子项纪律与 §1.1 的说法一致 |
| **清单指向** | §1.1.6 每行指向的 roster skill 路径真实存在 | 逐个 `Test-Path` / `ls` 核对该路径（含 `luzzy-skill-architect`、`luzzy-bilibili-notes` 与 `luzzy-aoci-index`） |
| **上下文边界** | `prompt/Luzzy.md` **不引用本仓库的维护文档**（本文件、`README.md` 的维护章节） | 搜 `AGENTS.md`：只应出现在 §8.1「读**用户工作区**的规范文件」的语境里；出现「本仓库自带 AGENTS.md」「见 AGENTS.md 第七节」一类指向 → **违规**，必须改成工作区相对表述 |
| 规则语义 | 「全读 / 超过 4 条取 4」「读 ≠ 装 ≠ 用」「先读后做 + 读取回执」「反假读八条」在位 | 搜关键词 |
| **认知索引门** | §1.1.5.1 三段式（命中 / 未接入 / 不适用）+ 认知回执格式齐全；§14.18 的时机表、三个坑位、许可警示在位；导航与 §十二 自查都指向它 | 搜「认知回执」「1.1.5.1」，逐个跳过去看；确认 §1.1.8 那条过门口径**带限定语**（`**「项目接手」另算一道门**` 形式）——**去掉限定语会让 triage-01 误命中，门评测会 FAIL** |
| **门覆盖** | §1.1.8 触发口径对真实任务说法的词面覆盖 ≥80%，triage / chitchat 零误命中 | `python evals/eval-gate-coverage.py`；漏报优先在 §1.1.8 **只增不删**补同义词，补完重跑；靠改语料消掉漏报不算过 |
| **分诊条款** | §1.1.5「分诊与 skill 激活」四段（探 / 判 / 激活 / 回写）齐全且内部自洽 | 搜「分诊」，逐个跳过去看；应串成一条完整链路：§0.1 做事协议与快速参考 → 导航铁律与速查表 → §1.1.1 硬门 / §1.1.4 反假读 / §1.1.5 本体四段 / §1.1.8 触发口径 → §二 第 1 步与脚注 → §3.1 技能自寻 → §8.1 与 §8.3 → §十一 播报例外 → §十二 自查 → §十三 第 1 步。**任何一环缺失或章节号改了没跟，就是断链**。指向 §1.1 内部小节的引用一律用 §1.1.x 编号，不写全名定位 |
| **交叉引用** | 文中所有 `§N.N` 都能找到对应小节；**roster 的行内引用（`luzzy-roster-<slug>`）与实际目录、登记表三方一致** | 把所有 `§` 引用抄出来逐个跳过去看（改章节编号时最容易漏）；再把所有 `luzzy-roster-<slug>` 行内引用抄出来，与 `skills/` 目录名及 `skills/luzzy-roster-family/README.md` 的 slug 表三方比对——更名 roster 时最容易漏 §1.1.8 与 §十四 的行内引用和 family 登记表 |
| 残留 | 无指向旧机制的写法（`luzzycode-*`、被删的上游仓库链接、`§1.1a`、被撤的 `luzzy-skill-roster`） | 搜 `luzzy-skill-roster`，应无结果；搜 `Luzzy-Skill-Architect` / `Luzzy-Skill-MeiHuaYiShu`，应只在来源说明里出现 |
| 格式 | 无装饰性 emoji（✅ ⚠ 🚫 三档标记、✗ ✓ 正反例标记豁免）、无裸露分隔线 | 目视 + 搜 `^---$` |
| **模板语法** | 正文里没有**双花括号变量语法**（连续两个左花括号后接变量名）——DSH 会把 persona 里的它当 prompt 变量引用解析，变量名须匹配 `[a-z][a-z0-9_]*`；**全大写形式**会让预设直接报错 | 已由 `sync-persona.mjs` 自动校验并拒绝（部署前会失败）；手工自查时搜「连续两个左花括号」。占位符统一用 `${...}` |
| 安全 | 无硬编码密钥形态 | 搜 `sk-` / `ghp_` / `Bearer`（文档里的占位符写法不算） |
| **skill 校验** | 四个技能各自通过校验 | `validate-trigger.py`（architect、bilibili、aoci-index）；梅花易数跑三条回归命令（见下） |
| **数字** | README 的行数、token、徽章等于实测值 | 第五节命令重测，**每次必做** |

**skills 的校验命令**（改动对应技能后必跑）：

```bash
# 触发词校验（两个技能通用）
python skills/luzzy-skill-architect/scripts/validate-trigger.py skills/luzzy-bilibili-notes
python skills/luzzy-skill-architect/scripts/validate-trigger.py skills/luzzy-skill-architect

# 梅花易数的回归门槛（三条全过，缺一不可——其 AGENTS.md 的 R5 强制要求）
cd skills/luzzy-skill-meihuayishu
python evals/integrity_check.py    # 结构完整性 + 版本四处一致
python evals/run.py                # 16 组回归用例（含五个原书占例）
python lunarcal.py --selftest      # 34 项历表锚点
```

> 梅花的 `integrity_check.py` 把 `README.md` 与 `.gitignore` 列为必备文件——**不要改名或删除它们**，否则回归失败。

**代价要认**：早先的门禁是纯文字清单，从没被执行过，于是 22 个 skill 全部违反其中的「无装饰性格式」而无人发现。现在回到人工核对，风险最高的是**数字漂移**——所以第五节把它列为每次必做，其余各项靠交付前过一遍。

## 五、README 里的数字必须实测

token 数用 `tiktoken` 的 `o200k_base` 实测，不估算：

```bash
python -c "import tiktoken,pathlib; t=pathlib.Path('prompt/Luzzy.md').read_text(encoding='utf-8'); e=tiktoken.get_encoding('o200k_base'); n=len(e.encode(t)); print(len(t.splitlines()),'行', n,'token'); print('徽章', f'{n/1000:.1f}k')"
```

| 项 | 算法 |
|---|---|
| 行数 | `len(prompt.splitlines())` |
| token | `len(tiktoken.get_encoding("o200k_base").encode(prompt))` |
| 徽章 | `f"{token/1000:.1f}k"` |

改了提示词就要重测并更新 README 的预算表与徽章。这是**唯一无法靠肉眼发现**的漂移项，务必每次都做。

`skills/` 侧的两个数字同样按实测：行数用 `find skills -type f -name '*.md' | xargs cat | wc -l`（全部 `.md` 文件合计），技能数按含 `SKILL.md` 的目录数计——改了 `skills/` 也要重测。

`AGENTS.md` 自身的体量不进预算（它不必注入 system prompt）。

## 六、绝对不许做

- 硬编码密钥、令牌、账号
- `git push --force`
- 改自检脚本或 README 来让检查通过
- 把规则写成两份（同一条规则在两个地方各写一遍）
- 在仓库里散落无主的临时文件——本轮产物本轮清
- 未经要求新建成品文档

---

## 七、接入 harness：路径速查

要把本提示词用到某个 Agent、或找它的 MCP 配置时，按下表定位。**先探本机是否真有该目录，再动手**；下表路径以各项目官方文档为准，但版本会变，用前建议复核一次。

**官方文档链接固化在此**，供后续维护或使用的 Agent 直接阅读——不用再搜一遍。

### ZCode（Z.ai）

**官方文档**：插件市场与插件体系 —— https://zcode.z.ai/docs ；插件内 MCP 声明格式见本机插件缓存里的 `.mcp.json` 与 `.zcode-plugin/plugin.json`

| 项 | 路径 |
|---|---|
| 提示词注入 | 作为 system prompt / 项目指令加载；ZCode 无强制的 skill 目录要求 |
| 插件缓存 | `~/.zcode/cli/plugins/cache/<来源>/<插件名>/<版本>/` |
| 配置 | `~/.zcode/cli/config.json`（`plugins.enabledPlugins` 与 `plugins.options`） |
| MCP | 插件自带 `.mcp.json`；官方插件由 ZCode 插件宿主拉起（命令被改写成 `ZCode.exe … __zcode-plugin-host`），**第三方插件不被自动改写** |
| 工具命名 | 模型侧 `mcp__<server>__<tool>`（连字符转下划线） |

ZCode 插件可同时打包 skills、commands、MCP server。官方插件源为 `zcode-plugins-official`。典型范例：`android-emulator` 插件的 `README.md` 写明了 SEA 构建改写 manifest 的机制。

### DSH（DeepSeek Harness）

**官方文档**：项目主页 —— https://www.deepseek.com/harness/en/ ；官方 skill 与文档规范示例 —— https://github.com/deepseek-ai/deepseek-harness/blob/master/.agents/skills/dsh-doc/SKILL.md

| 项 | 路径 |
|---|---|
| 指令文件 | 仓库根 `AGENTS.md`；skill 目录 `~/.agents/skills/` 与仓库内 `.agents/skills/` |
| 插件安装 | `dsh plugin --profile <名> add <包>` |
| 插件形态 | bundle（`package.json` 声明 `dsh.bundle.patch`）／plugin（`cordis.yml` 挂载）／library（仅依赖，无安装路径） |

**装之前先分辨包形态**：读 `package.json` 与入口文件，别按文件夹名字猜。

**本机预设的部署链路（同步本提示词到 DSH 时的实测经验）**

DSH 的 agent 预设放在 `~/.dsh/.agent-presets/<预设名>/`（本仓库对应 `luzzy`）。提示词不是直接读文件，而是被**机械嵌入** YAML：

```
persona.md  --(node sync-persona.mjs)-->  agent.cordis.yml 的 config.prefix: |- 块
   ↑ 唯一真源，与 prompt/Luzzy.md 逐字节一致         ↑ 6 空格缩进，禁止手工编辑
```

同步流程（四条，缺一不可）：

1. **先备份 `persona.md`**（同步脚本只备份 YAML，不备份 persona 正本），命名沿用 `persona.md.bak-<原因>-<时间戳>`
2. **覆盖 `persona.md`**，内容与仓库 `prompt/Luzzy.md` **逐字节一致**
3. `cd ~/.dsh/.agent-presets/luzzy && node sync-persona.mjs`
4. **复核**：脚本会自行解析回验 + 逐字节比对 + 顶层条目数校验，任一不过自动回滚；稳妥起见再独立跑一次解析比对

**四条硬约束（踩过坑，别复发）**：

| 约束 | 原因 |
|---|---|
| **行尾必须 LF** | 从仓库复制来的文件在工作区可能是 CRLF（本仓库已实测发生过）。CRLF 会让回验在第 1 行就失败：`YAML 侧 "…" / 源文件 "…\r"`。复制后先规整 `\r\n` → `\n`，**不要指望 `.gitattributes` 在 checkout 时自动转换** |
| **字段名必须是 `prefix`** | DSH 2.0.9+ 的 persona schema 要求必填 `prefix`；用旧的 `text` 会导致 preset 组装无限失败重试（host 日志刷 `ValidationError: $.prefix missing required value`） |
| **禁止手改 YAML 标量** | 生成块由脚本产出并回验；手改会让下一次同步产生巨大且无法比对的 diff |
| **顶层条目数不能变** | 脚本会校验（本机为 17 条），变了说明误伤了块外结构 |
| **正文禁止双花括号变量语法** | DSH 把 persona 前缀里连续两个左花括号当作 prompt 变量引用，变量名必须匹配 `[a-z][a-z0-9_]*`；**全大写形式**会让预设报 `malformed prompt variable reference`，**整个预设加载失败**。占位符统一写 `${...}`；本文件自身也不写该字面序列。**已由 `sync-persona.mjs` 拦截**（该脚本在预设目录、不在本仓库）——它在生成前校验变量名，不合法直接退出 1，不会走到 YAML |

**为什么会有这条**：2026-09-14 从「常驻提示词 + skill 层」合并为单一提示词时，`skills/luzzycode-skills/SKILL.md` 里那句「占位符用双花括号大写的 PLACEHOLDER」被一并搬进了 persona。该文本原先只住在 skill 文件里、从不进 persona，所以 DSH 的变量解析器碰不到；合并后 persona 加载直接失败。**教训**：合并或搬运文本进 persona 前先扫一遍模板语法——skill 里安全的写法，进 persona 不一定安全；上面两处防护条款本身也刻意不写那个字面序列。

**验证命令**（脚本之外独立复核一遍，确认解析出的文本与源文件一致）：

```bash
cd ~/.dsh/.agent-presets/luzzy && node -e "
const fs=require('fs'), yaml=require('/Users/<你>/.dsh/profiles/desktop/node_modules/js-yaml');
const T=new yaml.Type('tag:yaml.org,2002:js',{kind:'scalar',construct:d=>d,resolve:()=>true});
const p=yaml.load(fs.readFileSync('agent.cordis.yml','utf8'),{schema:yaml.DEFAULT_SCHEMA.extend([T])}).find(e=>e.id==='persona');
const src=fs.readFileSync('persona.md','utf8').replace(/\n+$/,'');
console.log('逐字节一致:', p.config.prefix.replace(/\n$/,'')===src);
"
```

- **生效时机**：改动只对**新会话**生效；进行中的会话上下文里仍是旧文本
- **登记惯例**：每次改动在 `persona.changes.md` 追加一条（日期 + 性质 + 踩坑 + 校验结果 + 回滚命令），`preset.yml` 的 `description` 也要跟着改——它是预设列表里显示的说明，最容易被忘
- **回滚**：恢复 `persona.md.bak-*` 与 `agent.cordis.yml.bak-sync-*` 两个文件即可

**本机 skill 目录的处置（2026-09-14 清空，2026-09-15 部分恢复）**：`~/.dsh/skills/` 下 19 个 `luzzycode-*` 目录曾在 `b1833cc` 时代手工拷入（18 个子 skill + 编排器），已于 2026-09-14 删除——它们的清单与提示词重复，会让 DSH 继续扫描并暴露给模型。

**恢复时的注意**：本仓库现有 `skills/` 下 26 个技能（21 个 `luzzy-roster-*` + architect / aoci-index / meihuayishu / bilibili / zip-password-recovery），装进任何 harness 的 skill 目录（含 `~/.dsh/skills/`）都会被扫描并常驻在模型可见的技能列表里。**这是有意的**——roster skill 只在命中类目时被读正文，列表里的 description 开销可接受（每个约 100 token，26 个合计约 2.6k）。但它们**不得**被塞进 persona；`sync-persona.mjs` 只嵌 `persona.md`，与 skills 无关。

**恢复来源**：若要找回更早期的 `luzzycode-*` 版本，见本仓库 git 历史 `0a474c2`（`git show 0a474c2:skills/<名>/SKILL.md`）——那里有全部 **23** 项，比本机部署的 19 项还多（含 reverse / assets / android / mcp）。

### Tabbit（浏览器自动化）

**官方入口**：国内官网 —— https://www.tabbit.com/ ；下载页 —— https://www.tabbit.com/download ；国际官网 —— https://www.tabbit.ai ；GitHub 组织 —— https://github.com/Tabbit-Browser

| 项 | 路径 / 命令 |
|---|---|
| 官方 skill（权威正文） | `~/.agents/skills/tabbit/`（随浏览器 Runtime 同步；含 `references/recovery.md`、`references/host-routing.md`） |
| CLI launcher（Windows） | `& "$env:LOCALAPPDATA\Tabbit\LocalAgent\bin\tabbit-cli.exe"` |
| CLI launcher（macOS / Linux） | `"$HOME/.local/bin/tabbit-cli"` |
| DSH 插件 | `dsh plugin --profile web add dsh-tabbit`（提供 `tabbit_browser` 工具与 `/tabbit-info` 命令） |
| DevTools / CDP skill | https://github.com/Tabbit-Browser/Tabbit-Devtools-Skill |
| 实例固定 | 环境变量 `TABBIT_PLAYWRIGHT_INSTANCE`（16 位大写 hex） |
| 权限配置 | DSH Settings → tabbit，或 `$DSH_HOME/settings.yaml` 的 `tabbit.pageAccess` / `tabbit.intranetFetch` |

**装完必须启动一次浏览器**，CLI launcher 与官方 skill 才会注册。红线见 `prompt/Luzzy.md` §14.16；操作细则（接入路径 / 命令族 / 执行纪律 / 失败路径）见 `skills/luzzy-roster-browser/SKILL.md` 执行细则节。

### Claude Code

**官方文档**：Skills —— https://code.claude.com/docs/en/skills ；MCP 接入（本地 stdio / 远端 HTTP / SSE） —— https://code.claude.com/docs/en/claude_code_docs_map ；插件内 MCP 集成范例 —— https://github.com/anthropics/claude-plugins-official/blob/main/plugins/plugin-dev/skills/mcp-integration/SKILL.md

| 项 | 路径 |
|---|---|
| 指令文件 | 项目根 `CLAUDE.md`（本仓库的 `AGENTS.md` 亦可读） |
| MCP | 全局 `~/.claude.json`；项目级 `.mcp.json` |
| skill 目录 | `~/.claude/skills/`、`<项目>/.claude/skills/`（若你另有 skill） |

### Codex CLI

**官方文档**：AGENTS.md 自定义指令 —— https://learn.chatgpt.com/docs/agent-configuration/agents-md ；配置参考（`config.toml`、`mcp_servers`） —— https://learn.chatgpt.com/docs/config-file/config-reference ；开放格式说明 —— https://agents.md/

| 项 | 路径 |
|---|---|
| 指令文件 | 仓库根 `AGENTS.md`（逐级向下读；存在 `AGENTS.override.md` 时优先读它）；全局 `~/.codex/AGENTS.md` |
| MCP | `~/.codex/config.toml` 的 `mcp_servers` |

Codex 在动手前读 `AGENTS.md`——所以本文件的维护约束对它直接生效。

### OpenClaw

**官方文档**：Skills 配置参考 —— https://docs.openclaw.ai/tools/skills-config ；Skills 概念与加载顺序 —— https://docs.openclaw.ai/tools/skills ；创建自定义 skill —— https://docs.openclaw.ai/tools/creating-skills

| 项 | 路径 |
|---|---|
| 指令文件 | `~/.openclaw/openclaw.json`；项目级 `<workspace>/` |
| skill 目录 | `~/.openclaw/skills`、`~/.agents/skills`、`<workspace>/skills`、`<workspace>/.agents/skills` |
| 额外扫描目录 | `skills.load.extraDirs`（优先级最低） |
| MCP 客户端注册表 | `~/.openclaw/skills/config/mcporter.json` |
| 安装策略 | `skills.install`；操作员审批 `security.installPolicy` |

注意：`agents.entries.*.skills` 的显式列表**替换**默认值而非合并；写成 `[]` 等于该 agent 看不到任何 skill。

### Hermes Agent（Nous Research）

**官方文档**：MCP 配置参考（键位、过滤、OAuth） —— https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference ；Skills 系统 —— https://hermes-agent.nousresearch.com/docs/user-guide/features/skills ；官方 skill 范例 —— https://github.com/NousResearch/hermes-agent/blob/main/optional-skills/mcp/fastmcp/SKILL.md

| 项 | 路径 |
|---|---|
| 指令文件 | 项目 `AGENTS.md`；skill 目录 `~/.hermes/skills/`（唯一事实源） |
| MCP 配置 | `~/.hermes/config.yaml` 的 `mcp_servers:` |
| 密钥 | `~/.hermes/.env`（配置里用 `${VAR}` 或 `${env:VAR}` 引用） |
| OAuth 令牌 | `~/.hermes/mcp-tokens/<server>.json` |
| 重载 | `/reload-mcp` |
| 工具命名 | `mcp__<server>__<tool>`，非字母数字下划线字符会被替换为 `_` |

`trust: untrusted` 会让所有写操作走审批面；`tools.include` 优先于 `tools.exclude`。

### QwenPaw（AgentScope）

**官方文档**：配置与工作目录 —— https://qwenpaw.agentscope.io/docs/config/ ；MCP 客户端配置 —— https://qwenpaw.agentscope.io/docs/mcp ；项目仓库 —— https://github.com/agentscope-ai/QwenPaw

| 项 | 路径 |
|---|---|
| 工作目录 | `~/.qwenpaw`（环境变量 `QWENPAW_WORKING_DIR`） |
| 全局配置 | `~/.qwenpaw/config.json` |
| Agent 配置 | `~/.qwenpaw/workspaces/<agent_id>/agent.json`（含 `mcp.clients`、`tools`、`skills`） |
| 提示词 / 人设 | 工作区内的 `AGENTS.md`、`SOUL.md`、`PROFILE.md`（由 `system_prompt_files` 控制加载） |
| 密钥目录 | `~/.qwenpaw.secret/`（`providers.json`、`envs.json`） |

`agent.json` 优先级高于全局 `config.json`；文件改动每 2 秒自动热重载。

### OpenCode

**官方文档**：配置参考（优先级、`mcp`、`instructions`、变量替换） —— https://opencode.ai/docs/config/ ；MCP 服务器配置 —— https://opencode.ai/docs/mcp-servers ；插件 —— https://opencode.ai/docs/plugins

| 项 | 路径 |
|---|---|
| 全局配置 | `~/.config/opencode/opencode.json` |
| 项目配置 | `<项目>/opencode.json`（优先级高于全局） |
| 指令文件 | 配置里的 `instructions` 数组（可指向 `AGENTS.md`） |
| MCP | 配置里的 `mcp` 段 |
| 自定义路径 | 环境变量 `OPENCODE_CONFIG`（配置文件）、`OPENCODE_CONFIG_DIR`（目录） |
| 管理端强制配置 | Linux `/etc/opencode/`、macOS `/Library/Application Support/opencode/`、Windows `%ProgramData%\opencode` |

优先级（低到高）：远端 `.well-known/opencode` → 全局 → 自定义 → 项目 → `.opencode/` → 内联 → 管理端。

### Cherry Studio

**官方文档**：内置工具、知识库、技能与 MCP —— https://docs.cherryai.com.cn/docs/en-us/advanced-basic/agent-workspace/tools-knowledge-skills-mcp ；项目仓库 —— https://github.com/CherryHQ/cherry-studio

| 项 | 路径 |
|---|---|
| MCP 配置 | 图形界面「设置 → MCP 服务器」（也支持直接改 `config.json`） |
| 已安装 MCP 落点 | 用户目录下的 `.cherrystudio/`（Windows 典型为 `C:\Users\<用户>\.cherrystudio\`） |
| 提示词 | 以 Agent 工作区的「技能」或系统提示形式配置 |

**Cherry Studio 是图形界面的桌面应用**：MCP 依赖模型的函数调用能力，且需要先在设置里连接并启动服务器，再回到 Agent 编辑窗口绑定。它是 GUI 优先，路径随版本变动较多，**以界面实际显示为准**。

---

## 八、给其他 Agent 的提示

- 本仓库的规则**以 `prompt/Luzzy.md` 为准**；本文件是维护说明，不替代它
- 要接入某台机器时：先确认该 harness 的真实配置位置（上表 + 本机勘探），再写；**不要凭表硬写路径**
- 上表路径来自各项目官方文档（2026-09 核对）。harness 迭代快，路径可能变——发现不符就报告并更新本文件
- 改动本仓库前先跑一次自检拿基线；改完再跑，对比差异
