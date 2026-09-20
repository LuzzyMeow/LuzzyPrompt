---
name: luzzy-aoci-index
description: >
  Use when a project is being taken over and its global cognition must be
  established: "接手这个项目看看该怎么弄" "接手一个陌生仓库" "接着上次那个继续，换我来接手"
  "这个项目交给你了，你先读一遍" "旧项目接手怎么上手" "接手一个新项目该做什么"
  "项目认知索引怎么用" "给这个项目建一份代码索引地图" "给项目建索引"
  "上下文压缩了，AI 把项目忘光了，重新理解一下" "怎么让 AI 记住整个项目不用每次重读"
  "换对话以后 AI 忘了项目" "aoci-code 的刷新时机是什么时候" "AOCI" "代码索引地图".
  Handles the three-branch gate for an existing index, a missing integration, and a
  non-git repository; reading a Whole-Index as the system-level prior; setup and host
  integration; and index maintenance at task closeout.
  Do NOT use for executing the coding work itself — discipline and red lines live in
  the Luzzy system prompt — nor for the reading gate, receipt, or triage logic, nor
  for authoring an MCP server (see the MCP category), nor for project planning or
  requirement breakdown, nor for reviewing a diff or pull request.
license: MIT
compatibility: >
  requires: git (the index inventory is taken from Git ignore authority; a non-git
  repository cannot be indexed). The aoci binary is a CGO-free single executable
  obtained from GitHub Releases or built from source; no daemon, no network, no
  Go toolchain needed for a verified release package. stdio MCP host required —
  Codex / Claude Code / OpenCode / Cursor are natively supported, other hosts
  (including DSH) need manual MCP configuration.
metadata:
  version: "1.0.0"
  author: "鹿溪 (LuzzyMeow)"
  category: "cognition-infrastructure"
  maturity: "L2"
---

# 项目认知索引（AOCI-CODE）

接手任何项目时的固定动作：先拿到这个项目的**全局认知**，再动手改代码。索引是一份随仓库保存、由 Git 版本化的纯文本认知地图——每个受管对象一条，记职责、强关系、公开契约，以及从代码里推断不出来的约束。

**官方仓库**：[`aoci-spec/aoci-code`](https://github.com/aoci-spec/aoci-code)（canonical source）｜当前 `v0.1.0-rc14`｜许可 FSL-1.1-MIT｜论文 [arXiv:2605.02421](https://arxiv.org/abs/2605.02421)

**版本会变**：以运行中二进制的 `--version` 与 `capabilities` 为准，不以本文写的版本号为准。

## Workflow

1. 判定三段式分支，落认知回执。

   在仓库根探 `.aoci/` 与认知资产（`aoci.txt` / `aoci.meta.txt` / `aoci.code.txt` /
   `aoci.database.txt`），按结果走：

   | 情形 | 动作 |
   |---|---|
   | 有索引 | 读 Rules 与 Whole-Index，作系统级先验；确认是否 `aligned` |
   | 没接（且是 git 仓库） | 向用户提议接入，一次说清耗时 / 许可 / 写入面；拒绝就转普通探索 |
   | 非 git 仓库 | 跳过并说明——`scan` 依赖 Git 忽略权威，没有 git 建不起索引 |

   回执格式（与提示词的读取回执并列，不混用）：

   ```
   认知索引：<命中 / 未接入 / 不适用> | 索引版本：<…> | 覆盖：<受管文件数 / 规模> | 结论：<可作系统级先验 / 本次按普通探索推进>
   ```

   Verify: 回执落在动手前的可见输出里，三个分支之一有明确结论。

2. 读认知（命中分支）。

   先读 Rules，再读 Whole-Index。索引超过交付块预算时会分块返回，**沿游标读到
   最后一块**才算读完——局部搜索、旧记忆、直接读源码都不能冒充完整交付。

   读法与交付协议见 [references/cognition-lifecycle.md](references/cognition-lifecycle.md)。

   Verify: 报告里能说出索引声明的版本与覆盖范围；未读完分块时不声称已有系统认知。

3. 首次接入（未接入分支，拿到用户同意后）。

   按官方安装说明取二进制、校验、放稳定绝对路径，然后接入并扫描：

   ```bash
   AOCI=/absolute/path/to/aoci          # Windows: C:\path\to\aoci.exe
   "$AOCI" --repo . init --locale zh-CN --agent <codex|claude|opencode|cursor>
   "$AOCI" --repo . scan
   ```

   接入写的宿主配置含**本机绝对路径**，必须 gitignore。宿主差异与 DSH 手工配置见
   [references/install-and-hosts.md](references/install-and-hosts.md)。

   Verify: `"$AOCI" --version` 有输出；会话里出现了 AOCI 工具（没出现就重启宿主）。

4. 建立索引（首次）。

   索引由宿主模型逐文件创作，不是程序自动生成。20 万行约 1 小时，按批、可中断续做。
   大仓库先与用户确认时间成本再开始。

   Verify: `verify` 与 `check` 收敛到 `aligned`。

5. 收尾维护（每次开发任务结束时，一次）。

   代码与测试稳定之后调**一次**维护，解决待修项，用校验与指南收敛回 `aligned`。
   **开发中途不要触发**——仓库不干净时会被顶回。时机表与理由见
   [references/maintenance-timing.md](references/maintenance-timing.md)。

   Verify: 维护后状态是 `aligned`；出现 `stopped` 时不当作成功。

6. 上下文压缩后强制重载。

   已知发生过压缩时，完整重读认知（声明压缩事件 + 新事件 ID），**便宜的检查点不能替代**。
   协议见 [references/cognition-lifecycle.md](references/cognition-lifecycle.md)。

   Verify: 重载后落一次完整交付确认，而不是靠旧记忆答话。

7. 失败路径（按症状处理）。

   | 症状 | 处理 |
   |---|---|
   | 索引建不起来、条目为空 | 认知资产可能进了 `.gitignore`——被忽略会被**静默跳过**且不报错。移出忽略清单后重扫 |
   | 宿主重跑接入后路径仍是坏的 | 接入器按条目是否存在判幂等，会**静默保留坏路径**。删掉那条配置再重建 |
   | 维护被顶回、状态是「等稳定」 | 正常：仓库不干净。先完成当前工作单元 + 格式化 / lint / 测试，再维护 |
   | 维护结果是 `stopped` | **不是成功**，也不等于没写入。看失败步骤与恢复证据，按指南恢复 |
   | 索引跟不上代码 | 不是实时索引——检测是拉取式，必须调一次维护。见 [references/maintenance-timing.md](references/maintenance-timing.md) |
   | 会话里没有 AOCI 工具 | 宿主没加载 MCP。重启宿主；DSH 等非原生宿主需手工配置 |
   | 二进制路径换了但行为没变 | 磁盘换字节不影响已在运行的 MCP 进程。用运行时报的版本与仓库根复核 |
   | 数据库连不上 | 只读表结构需要管理员在外部提供凭据引用；AOCI 不保存、不读取凭据值 |

## Compliance Boundary

- **接入是写操作**：会在仓库内建认知资产、写宿主配置、动 Git 边界——先列改动面拿用户确认
- **不代替用户接受许可**、不代替用户输入数据库凭据；凭据只走环境变量引用
- **不把 Fair Source 说成开源**：FSL-1.1-MIT 带 Competing Use 限制，两年后才转 MIT
- **只对目标项目读写**：索引在项目目录内，不向外发送任何内容

许可与坑位细节见 [references/traps-and-license.md](references/traps-and-license.md)。

## Examples

Input: 「接手这个项目看看该怎么弄」（仓库根有 `.aoci/`）
Output: 落认知回执「命中 | 版本 v1 | 覆盖 480 文件 | 可作系统级先验」→ 读 Rules + Whole-Index
（分块则读到末块）→ 报告掌握范围 → 再进具体任务。

Input: 「接手一个陌生仓库，帮我加个导出功能」（无 `.aoci/`，是 git 仓库）
Output: 落认知回执「未接入 | — | — | 已提议接入」→ 说明三件事（20 万行约 1 小时 /
FSL-1.1-MIT 不是开源 / 写入面含仓库资产与宿主配置）→ 用户同意则接入、建索引；
拒绝则按普通探索推进，不反复索取。

## Verify

- [ ] 动手前落了认知回执，三个分支之一有明确结论？
- [ ] 有索引时，Whole-Index 读完整（分块读到末块），没有用搜索或旧记忆冒充？
- [ ] 未接入时提议里说清了耗时、许可、写入面三件事？
- [ ] 非 git 仓库被如实跳过，没有硬试？
- [ ] 接入前拿到了用户对改动面的确认？
- [ ] 认知资产没有被加进 `.gitignore`？
- [ ] 宿主配置（含本机绝对路径）已 gitignore、未提交？
- [ ] 索引维护发生在代码与测试稳定之后，且只调一次？
- [ ] 上下文压缩后走了完整重载，而不是便宜检查点？
- [ ] 没有把 FSL-1.1-MIT 说成开源？

## Reference Files

| File | Load when |
|------|-----------|
| [references/install-and-hosts.md](references/install-and-hosts.md) | 首次接入：取二进制、校验、init/scan、四家宿主差异、DSH 手工配置 |
| [references/cognition-lifecycle.md](references/cognition-lifecycle.md) | 读认知、分块交付、确认与校验、压缩后的强制重载 |
| [references/maintenance-timing.md](references/maintenance-timing.md) | 判断何时维护、为什么不能中途、阈值与状态含义 |
| [references/traps-and-license.md](references/traps-and-license.md) | 三个静默坑位、许可边界、非 git 降级、数据库访问边界 |
