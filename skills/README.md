# Luzzy 配套 skill

本目录是 [Luzzy](../README.md) 的配套技能层。提示词 [`prompt/Luzzy.md`](../prompt/Luzzy.md) 负责**每轮都生效的硬规定**，本目录负责**命中特定场景才需要的操作细则**——两层分工，规则不重复。

命中场景时，提示词 §1.1 的必读清单每类指向一个本地 roster skill；本机已有就直接读，不必联网。

## 技能一览

| 技能 | 用途 | 何时触发 | 许可 |
|---|---|---|---|
| [`luzzy-roster-*/`](luzzy-roster-backend/) | 十七类必读清单的仓库子项明细——每类一个 skill（21 个：17 类 + 设计三框架子项 + 学术/解题分立），装 URL、条数、读什么、安装坑位、体积与许可警示，以及该类目的操作性执行细则（工具面 / 命令族 / 失败路径，2026-09-18 从提示词 §十四 迁入） | 提示词 §1.1.6 命中任一类目后读对应 roster；维护子项明细或执行细则时改对应 roster | MIT |
| [`luzzy-roster-family/`](luzzy-roster-family/) | roster 家族的登记与规范目录（**非 skill**，无 SKILL.md）：生成规范 + 子项变更登记 | 维护 roster 时读，运行时不加载 | — |
| [`luzzy-skill-architect/`](luzzy-skill-architect/) | 创建、审计、诊断、融合 Agent Skills 的元框架：PPER 协议 + 五阶段生命周期 + L0–L5 成熟度 + 七设计模式 + 十反模式库 + 质量门禁 | 创建 / 改进 / 审计 skill，写 `SKILL.md`，校验触发词，评估成熟度，融合多个 skill | Apache-2.0 |
| [`luzzy-aoci-index/`](luzzy-aoci-index/) | 项目认知索引（AOCI-CODE）的接线与维护：三段式硬门（有索引必读 / 未接入必提议 / 非 git 跳过）、首次建索引、宿主接入、刷新时机（读在开头、写在收尾）、许可与写入面警示 | 接手新项目或旧项目、中途接手陌生仓库、「接着上次那个继续」、上下文压缩后重建认知 | MIT |
| [`luzzy-skill-meihuayishu/`](luzzy-skill-meihuayishu/) | 梅花易数技能家族（L5）：零依赖起卦引擎 + 《周易》《梅花易数》原文内置 + 原书占例回归 16/16 | 用户要算卦、起卦、占卜、用名字 / 时间 / 报数起卦 | MIT |
| [`luzzy-bilibili-notes/`](luzzy-bilibili-notes/) | 把 B 站视频提取成内容报告：默认抓标题、简介、AI 字幕、全部公开评论（含二级回复）四项，解析 SRT，**默认在对话里报告，不落文件**；用户明确要笔记时才按模板落盘 | 用户给 B 站链接要提取内容、转笔记、转文档、抓字幕、看简介与评论 | MIT |
| [`luzzy-zip-password-recovery/`](luzzy-zip-password-recovery/) | ZIP 压缩包密码恢复与免密解压：先分类 ZipCrypto / WinZip AES，再走字典 / 掩码 / 已知明文攻击，全量 CRC32 校验后解出每一项 | 用户要破解 zip 密码、忘了解压密码、加密压缩包打不开、点名 zip2john / hashcat；RAR / 7z / PDF 等其他格式不归它 | MIT |

## 安装到某个 Agent

三种方式，按需选一种：

```bash
# 1) 直接用（推荐）：把本仓库克隆到工作区，提示词里的清单会指向 skills/<名>/
git clone git@github.com:LuzzyMeow/LuzzyPrompt.git

# 2) 装进该 Agent 的用户级 skill 目录（以 Claude Code 为例）
cp -r LuzzyPrompt/skills/luzzy-skill-architect ~/.claude/skills/

# 3) 装进跨平台目录（Codex / OpenClaw 等认这个）
cp -r LuzzyPrompt/skills/luzzy-bilibili-notes ~/.agents/skills/
```

各家 harness 的 skill 目录与 MCP 配置路径速查，见 [`AGENTS.md`](../AGENTS.md) 第七节。

## 维护须知

- **`luzzy-skill-meihuayishu/` 自带维护宪章**：[`AGENTS.md`](luzzy-skill-meihuayishu/AGENTS.md)。它规定了十条红线（经典文本不可改写、计算一律走引擎、回归门槛不可放宽等）与四类变更流程。**改动该技能前必须先完整读它**，并跑通三条回归命令：

  ```bash
  cd skills/luzzy-skill-meihuayishu
  python evals/integrity_check.py    # 结构完整性 + 版本四处一致
  python evals/run.py                # 16 组回归用例（含五个原书占例）
  python lunarcal.py --selftest      # 34 项历表锚点
  ```

- **`luzzy-skill-architect/` 自带触发校验**：

  ```bash
  python skills/luzzy-skill-architect/scripts/validate-trigger.py skills/<技能目录>
  ```

- **新增配套 skill**：按 `luzzy-skill-architect` 的质量门禁产出（`name` 与目录名一致、`description` 只写触发条件含负面触发词、正文 ≤500 行、≥2 组 I/O 示例、含 `Verify` 段），并在本文件与提示词 §1.1 的清单里登记。

  > **前缀有别**：`luzzy-roster-*` 专指**提示词 §1.1.6 十七类清单的类目子项明细**（21 个），`luzzy-roster-family/` 是这一族的生成规范与登记目录。**不是**类目子项的配套 skill 用别的名字（如 `luzzy-aoci-index`、`luzzy-bilibili-notes`）——别为了整齐把非 roster 的东西塞进 roster 前缀，那会破坏「行内引用 × 目录名 × 家族表」的三方一致。

## 来源与许可

前两个技能原为独立仓库，2026 年迁入本仓库统一维护，原仓库已删除；迁移保持内容原样，仅改写指向旧仓库的路径引用，各技能的 `README.md` 保留其历史说明。

| 技能 | 原仓库 | 许可 | 要求 |
|---|---|---|---|
| luzzy-skill-architect | `LuzzyMeow/Luzzy-Skill-Architect` | Apache-2.0 | 保留 `LICENSE` 与版权声明 |
| luzzy-skill-meihuayishu | `LuzzyMeow/Luzzy-Skill-MeiHuaYiShu` | MIT | 保留 `LICENSE` 与版权声明 |

各技能的 `LICENSE` 随目录保留，未做改动。`luzzy-skill-meihuayishu` 内置的经典文本均为公有领域（作者逝世逾百年），历法数据表为社区公开成果，出处见其 `references/sources.md`。

`luzzy-bilibili-notes/` 与 `luzzy-zip-password-recovery/` 为本仓库自研（后者 2026-09-16 挂靠 `luzzy-roster-reverse` 第 2 条），许可均为 MIT。

`luzzy-aoci-index/` 亦为本仓库自研（MIT）。它指向的第三方项目 **AOCI-CODE**（[`aoci-spec/aoci-code`](https://github.com/aoci-spec/aoci-code)）采用 **FSL-1.1-MIT**（Fair Source / source-available，**不是** OSI 开源）——本 skill 只写接入与维护方法，**不分发该项目任何代码或二进制**，用户自行从官方来源获取。
