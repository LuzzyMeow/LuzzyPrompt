---
name: luzzy-roster-browser
description: >
  Use when the Luzzy checklist (必读清单) hits the 浏览器自动化 / 网页操作 category —
  reading sub-items for real-browser automation (Tabbit official skill, DSH plugin, DevTools CDP skill).
  Answers "浏览器自动化读什么" "tabbit的官方skill在哪" "接管已登录的浏览器" "网页操作的任务名怎么起".
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

# 浏览器自动化 / 网页操作 · 仓库子项明细

提示词 §1.1.6 命中「浏览器自动化 / 网页操作」后，读本文件拿仓库子项明细与执行细则。阅读口径（全读 / 折减 / 读 ≠ 装 ≠ 用 / 读取回执）见提示词 §1.1.3，本文件不重述。

## Workflow

1. 对照下表读子项：先本机后云端，读到正文才算（首页 / README 摘要不算）
   Verify: 读取回执里的子项名与本表条目名一致
2. 装之前核体积、许可与整仓要求；「读」不等于「装」，读满口径即完成
   Verify: 每条要装的子项都有体积与许可记录；缺失就按 §五 澄清
3. 链接失效（404 / 超时 / 已归档 / 内容为空）→ 按提示词 §1.1.9 上报用户，再自行寻找替代：抓取通道按提示词 §1.2（gh-proxy / ghfast 镜像、AnySearch `extract` 抓 raw 正文、Gitee 导入兜底，用前先探，逐级降级）；本 skill 内补齐同类型的按各家标注执行（如设计类「任意两条失效联网补齐」）
   Verify: 上报发生在任何替代动作之前；替代通道从头到尾可溯源；未静默跳过或悄悄顶替

## 子项（3 条 → 全读）

| # | 条目 | 读什么 |
|---|---|---|
| 1 | 本机官方 skill `~/.agents/skills/tabbit/`（`SKILL.md` + `references/recovery.md` + `references/host-routing.md`） | 权威正文——随浏览器 Runtime 同步 |
| 2 | **dsh-tabbit 官方插件** `https://github.com/Tabbit-Browser/dsh-tabbit` | 正文——DeepSeek Harness 集成 |
| 3 | **Tabbit-Devtools-Skill** `https://github.com/Tabbit-Browser/Tabbit-Devtools-Skill` | 正文——接管既有 Chrome / Edge |

缺 Tabbit 本体时先读官网 `https://www.tabbit.com/` 与引导（提示词 §14.16）；装完**必须启动一次**浏览器，CLI launcher 与官方 skill 才会注册。共享 cookie 的取用红线（绝不外发、用完即删、绝不入库）与登录 / 验证码 / 支付留给用户的边界见提示词 §14.16。

## 执行细则（提示词 §14.16 的操作明细）

**三条接入路径**：

| 路径 | 适用 | 入口 |
|---|---|---|
| **官方 skill**（首选） | 任何支持 Agent Skills 的 harness | `~/.agents/skills/tabbit/`（随浏览器 Runtime 同步，永远优先读它） |
| **dsh-tabbit 插件** | DeepSeek Harness | `dsh plugin --profile web add dsh-tabbit`；提供 `tabbit_browser` 工具、`/tabbit-info` 诊断命令 |
| **DevTools / CDP** | 已有 Chrome/Edge，或需要接管既有浏览器 | `Tabbit-Devtools-Skill` |

**稳定入口与命令族**：

```powershell
# Windows
& "$env:LOCALAPPDATA\Tabbit\LocalAgent\bin\tabbit-cli.exe" diagnose
```

```bash
# macOS / Linux
"$HOME/.local/bin/tabbit-cli" diagnose
```

**永远用稳定 launcher**，不要进应用包内部或版本化运行时目录找 CLI。

| 命令 | 用途 |
|---|---|
| `diagnose [--task N]` | 查能力、运行时限制、任务清单与占用 |
| `tabs --task N [--state available\|owned\|claimed]` | 列标签页清单（**清单不等于接管**） |
| `claim --task N --tab ID...` | 显式接管指定标签页 |
| `resume --task N --group ID` | 复用先前保留的标签组 |
| `nodejs --task N --request-id ID [--read-only]` | 提交 Playwright 程序（从 stdin 读 JS） |
| `resource --task N --resource ID --offset 0` | 读超过 16 KiB 的结果分片 |
| `finish --task N` | 收尾：释放占用、保留有用的标签组 |
| `finish --task N --discard` | 收尾并**关闭任务自有的全部标签页**（任务创建的 + 被 `claim` / `resume` 接管的） |

**执行纪律**：

- **任务名唯一并复用**：一个用户目标用一个短 `NAME`，全程不变——它同时是标签组标题
- **`--request-id` 每次唯一**：执行状态不明时先核对回执，**不要重跑可能已发生的填写或提交**
- **`--read-only` 只声明不改状态**：读标题、读文本算只读；导航、点击、填表都不算
- **一次程序内完成**：导航 + 提取 + 验证写在一个程序里，不要一次一个字段地反复探测
- **临时标签页在 `finally` 里关**：只关本任务创建的；**绝不关用户的已接管标签页**
- **收尾 `finish` 恰好一次**——要清干净就用 `--discard`
- **Windows 传多行 JS**：写 UTF-8 临时文件 + `cmd /d /c "... < 文件"` 重定向；**不用** PowerShell 管道与 here-string（会改写换行与编码）

### 标签页与进程生命周期（实测结论）

**问题**：Agent 用浏览器时会打开一个标签页，用户希望任务结束后它自己消失，且不碰用户原有的标签页。

**实测条件**：Tabbit 正在运行，用户自己有 2 个标签页（`state=available`，`group=null`）。

| 阶段 | 观测 |
|---|---|
| 任务运行中 | 清单 3 页：用户 2 页（`available`）+ 任务 1 页（`owned`，属任务组）；`oneGroupPerTask: true` |
| `finish --task N --discard` 后 | 用户两页**原样保留**（tabId / windowId 均未变）；`taskCount: 0`、`activeTaskCount: 0`、`occupiedTaskCount: 0` |

**三条结论**：

1. **能关掉自己开的标签页**——用 `finish --task N --discard`。所有权模型保证只关任务自有的
2. **能不碰用户标签页**——只有 `claimed` / `resumed` / `task-created` 的页进入 Agent 可见范围
   （`contextContainsOnlyOwnedTabs: true`）；`finish` 只释放本任务所有权
3. **不能让浏览器进程消失**——launcher 的语义是「需要时拉起浏览器」；CLI 命令面只有
   `finish, tabs, claim, resume, nodejs, receipt, resource, diagnose`，**没有退出命令**。
   硬杀进程可能损坏 profile、丢用户会话，官方也要求浏览器生命周期由浏览器自己管

**因此收尾按这三步**：

```text
1. finish --task N --discard          # 关掉任务自有的标签页（恰好一次）
2. 清掉 cookie / 截图 / 下载物等临时产物
3. 浏览器是被本次任务拉起的（任务开始前没开着）→ 如实报告并询问用户是否关掉浏览器
   否则 → 什么都不说，用户自己的浏览器维持原样
```

**四条禁止**：

- 不要去关用户自己的标签页——所有权已隔离，`available` 状态的页不许动
- 不要因为某页 URL 是 `about:blank` 就当垃圾关掉——用户可能正要它
- 不要杀浏览器进程或结束 Runtime 服务
- `finish` 报错时不要反复重试、更不要改用杀进程——如实说明残留，让用户手动关

**一个省标签页的技巧**：读登录态 cookie **不需要导航**——`context.cookies(origin)` 直接读。
为空任务自动创建的隐式页是空白的，正常 `finish` 也会关掉它。
不要为了读 cookie 去打开目标网站，那会平白多出一个真正的页面标签页。

**未实测分支**：「浏览器原本没开、被本次任务拉起」这一支**没有实测**——上面的处置按官方
文档语义写（"launching it when necessary"）。遇到这一支时如实告诉用户这是文档依据。

**失败路径**：

| 症状 | 处置 |
|---|---|
| Agent 找不到 tabbit skill | 先确认 Tabbit 已更新、且浏览器与 Agent 都启动过；重启 Agent 会话再试 |
| `diagnose` 无响应 | 检查 launcher 是否已注册（浏览器是否启动过至少一次）；未装则走提示词 §14.16 的前置引导 |
| 退出码 69 | 路由不可用或有歧义：列出实例并按提示词 §五 询问用户选哪个 |
| 实例冲突（装了多个版本） | 用 `TABBIT_PLAYWRIGHT_INSTANCE` 固定一个 16 位大写 hex 实例 ID，全程不变 |
| 环境预检缺项 | 用 `tabbit_browser_install`（DSH 插件）或引导用户手动装；**不要**自己去下载安装浏览器 |
| 页面需要登录 | 把登录交回用户完成，然后复用已登录的会话继续 |

## Examples

Input: 命中「浏览器自动化」
Output: 读本表 3 条 → 永远用稳定 launcher → 任务名唯一并复用 → 落回执。

Input: 页面要登录
Output: 登录交回用户完成 → 复用已登录会话继续 → cookie 落临时目录用完即删。



## Reference Files

| File | Load when |
|------|-----------|
| [../luzzy-roster-family/references/checklist-history.md](../luzzy-roster-family/references/checklist-history.md) | 维护场景：子项增删改登记（维护文档，非运行时上下文） |

纪律、红线与验收标准见提示词 §14.16；本 skill 装清单明细与操作性执行细则。
