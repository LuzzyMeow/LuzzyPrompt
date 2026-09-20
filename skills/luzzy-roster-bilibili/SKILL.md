---
name: luzzy-roster-bilibili
description: >
  Use when the Luzzy checklist (必读清单) hits the B 站视频转笔记 / 字幕提取 category —
  the local companion skill for extracting Bilibili videos into structured Markdown notes.
  Answers "B站视频转笔记的清单子项" "BV号提取内容读什么" "抓字幕用哪个skill" "B站评论怎么整理成文档".
  Do NOT use for executing the task itself — discipline and red lines live in the Luzzy
  system prompt — nor for the reading gate, receipt, or triage logic, nor for other
  checklist categories or tasks unrelated to this one.
license: MIT
metadata:
  version: "1.0.0"
  author: "鹿溪 (LuzzyMeow)"
  category: "luzzy-roster"
  maturity: "L2"
---

# B 站视频转笔记 / 字幕提取 · 仓库子项明细

提示词 §1.1.6 命中「B 站视频转笔记 / 字幕提取」后，读本文件拿仓库子项明细。阅读口径（全读 / 折减 / 读 ≠ 装 ≠ 用 / 读取回执）见提示词 §1.1.3，本文件不重述。

## Workflow

1. 对照下表读子项：先本机后云端，读到正文才算（首页 / README 摘要不算）
   Verify: 读取回执里的子项名与本表条目名一致
2. 装之前核体积、许可与整仓要求；「读」不等于「装」，读满口径即完成
   Verify: 每条要装的子项都有体积与许可记录；缺失就按 §五 澄清
3. 链接失效（404 / 超时 / 已归档 / 内容为空）→ 按提示词 §1.1.9 上报用户，再自行寻找替代：抓取通道按提示词 §1.2（gh-proxy / ghfast 镜像、AnySearch `extract` 抓 raw 正文、Gitee 导入兜底，用前先探，逐级降级）；本 skill 内补齐同类型的按各家标注执行（如设计类「任意两条失效联网补齐」）
   Verify: 上报发生在任何替代动作之前；替代通道从头到尾可溯源；未静默跳过或悄悄顶替

## 子项（1 条 → 全读）

| # | 条目 | 读什么 |
|---|---|---|
| 1 | **luzzy-bilibili-notes**（本地配套 skill）`skills/luzzy-bilibili-notes/SKILL.md` | 完整正文；`references/` 六个文件与 `scripts/` 两个脚本按需加载 |

默认抓取四项：标题、简介、AI 字幕、全部公开评论（含二级回复）——用户未明确收窄时一律全取（触发口径见提示词 §1.1.8）。

**默认产出是对话里的报告，不是文件**：用户没说要不要文件时不新建任何文件；明确要笔记 / 文档 / Markdown 时才落盘（结构照 `references/note-template.md`）。未经要求不新建成品文档是提示词 §7.1 的硬规定。

**收尾要关掉自己的标签页**：走 Tabbit 取登录态时，任务结束执行一次 `finish --discard`——只关任务自有的页，**不碰用户的标签页、不杀浏览器进程**（提示词 §14.16 标签页纪律）。

风控与合规边界（不批量、不绕 WBI、不碰付费内容）以该 skill 正文与提示词红线为准。

## Examples

Input: 命中「B 站转笔记」
Output: 读本表 1 条 → 按 bilibili-notes 的七步工作流走 → 落回执。

Input: 用户只说「看看讲了啥」
Output: 仍按默认四项全取 → 落回执。



## Reference Files

| File | Load when |
|------|-----------|
| [../luzzy-roster-family/references/checklist-history.md](../luzzy-roster-family/references/checklist-history.md) | 维护场景：子项增删改登记（维护文档，非运行时上下文） |

执行纪律、红线与失败路径见提示词 该 skill 正文本身与提示词 §1.1.8；本 skill 只装清单明细。
