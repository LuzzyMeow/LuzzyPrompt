# 首次接入：取二进制、初始化、接进宿主

只在仓库**没有**认知索引时用本文件。已有索引直接读，不必重装。

## 一、取二进制

两条路，**优先用签名 Release 包**——用预编译包不需要 Go 工具链：

```bash
# A. 官方 Release（推荐）：先完成校验层级再使用
gh auth login
gh release download v0.1.0-rc14 --repo aoci-spec/aoci-code
```

匿名下载：浏览器打开
<https://github.com/aoci-spec/aoci-code/releases> 取所需 archive 与校验资产。
校验分基础 / 推荐 / 完整三层，**按官方安装指南做完并如实说明完成了哪一层**。

```bash
# B. 从源码构建（无合适包、或用户明确要最新源码时）
git clone https://github.com/aoci-spec/aoci-code.git   # 克隆走 SSH 优先
cd aoci-code
make build
./build/aoci --version
```

Windows 从同一源构建 `aoci.exe`：

```powershell
git clone https://github.com/aoci-spec/aoci-code.git
Set-Location .\aoci-code
New-Item -ItemType Directory -Force .\build | Out-Null
make build
.\build\aoci.exe --version
```

**放稳定绝对路径**：二进制可以在源码工作区，也可以放统一工具目录，
只要宿主 MCP 配置指向正确的绝对路径。**路径不能变**——宿主配置记的是绝对路径。

Verify: `aoci --version` 有输出；记为长期稳定路径，不要放在会话临时目录。

## 二、初始化与扫描

```bash
AOCI=/absolute/path/to/aoci
"$AOCI" --repo . init --locale zh-CN --agent <codex|claude|opencode|cursor>
"$AOCI" --repo . scan
```

PowerShell：

```powershell
$Aoci = (Resolve-Path "C:\path\to\aoci.exe").Path
& $Aoci --repo . init --locale zh-CN --agent codex
& $Aoci --repo . scan
```

- `init` 写入：语言配置、受管的 `AGENTS.md` 规则区块、Git 边界、**语义为空**的认知骨架，
  以及宿主集成配置。它**不**根据文件名或语法树伪造业务语义
- `scan` 建立受管基线。新增仓库首次扫描即定基线；已有基线时改范围要走正式范围变更流程，
  **`--force` 不是重新定义治理事实的捷径**
- **只探不伪造**：`init` 后索引是空的，真实语义要等模型逐条创作（见 SKILL.md 步骤 4）

**关键约束**：`scan` 按 **Git 忽略权威**取文件清单。
`aoci.txt` / `aoci.meta.txt` / `aoci.code.txt` / `AGENTS.md` **不能**出现在
`.gitignore` 或 `.git/info/exclude` 里——被忽略的会被**静默跳过**，索引建不起来且不报错。
`init` 自己写的宿主配置忽略项保持原样。

Verify: `scan` 之后仓库根出现 `aoci.txt` 与 `.aoci/`；`.mcp.json` 等宿主配置在忽略清单里。

## 三、宿主集成差异

`init` 总会写受管的代理规则，但宿主集成各不相同：

| 宿主 | 集成方式 | 边界 |
|---|---|---|
| **Codex** | 项目级 stdio MCP；`--hooks` 另装压缩提示与 `SessionStart(compact)` | 钩子需经 `/hooks` 审阅信任；不装文件编辑钩子 |
| **Claude Code** | 项目级 MCP；可选轻量 `PreToolUse` 守卫 | 钩子只给写前提醒，不是运行时 |
| **OpenCode V1** | 经 `--agent opencode` 写严格的项目根配置 | 工具已加载即可继续，否则刷新会话 |
| **Cursor** | **只返回**参考配置片段，不写项目 | 需用户手工粘贴 |
| **其他宿主（含 DSH）** | 连标准 stdio 服务器 | 需手工配置与宿主侧验证 |

```bash
aoci --repo /abs/path/to/repo init --agent codex
aoci --repo /abs/path/to/repo init --agent codex --hooks
aoci --repo /abs/path/to/repo init --agent claude --hooks
aoci --repo /abs/path/to/repo init --agent opencode
aoci --repo /abs/path/to/repo init --agent cursor
```

**为什么接入后要重启宿主**：索引由 MCP 工具创作，而刚跑 `init` 的那个会话还没加载
新写入的 MCP 服务器。支持动态加载的宿主可能不必重启。

### DSH 等非原生宿主的手工配置

`init --agent cursor` 会返回可参考的片段；其他宿主按标准 stdio MCP 配置：

```json
{
  "mcpServers": {
    "aoci": {
      "command": "/absolute/path/to/aoci",
      "args": ["--repo", "/absolute/path/to/repository", "mcp"]
    }
  }
}
```

要点：`command` 与 `--repo` 都必须是**绝对路径**；`--repo` 指被治理的仓库根。
各家客户端配置文件位置**先探本机实际路径再写**，别按印象硬写。

Verify: 重启后会话里出现 AOCI 工具（九个：`aoci_rules` / `aoci_overview` /
`aoci_get_entries` / `aoci_search` / `aoci_maintain` / `aoci_update_entry` /
`aoci_remove_entry` / `aoci_header` / `aoci_report`）。

## 四、确认连的是哪一个 AOCI

**看服务端自报身份，不看磁盘文件**：任何 `aoci_overview` 的 `check_only` 响应、
或任何 `aoci_maintain` 响应里，`cognition_receipt.mcp_service_version` 是**正在运行**的版本，
`runtime_repository_root` 是它治理的仓库。

**替换磁盘上的字节不会改变已在运行的 MCP 进程**——升级或回滚后必须按这两个事实复核。

## 五、安装后会出现什么

```text
aoci.txt                    根：声明当前认知集合与参与卷
aoci.meta.txt               元：标签字典、条目规则与创作约束
aoci.code.txt               代码：代码与仓库资产的认知条目
aoci.database.txt           数据库：可选的表级认知，默认不存在
.aoci/
├── config.json             团队策略、语言、范围与预算
├── baseline.json           源码、认知与数据库绑定的治理基线
├── curation.json           可选的文件级纳入 / 排除决策
└── ...                     草稿、账本、事务与恢复证据，通常不进 Git
```

**宿主配置必须 gitignore**（`.mcp.json` / `.claude/settings.json` /
`.codex/config.toml` / `opencode.json`）——它们含本机绝对路径，提交到别的机器必然失效。

## 诊断命令

```bash
AOCI=/absolute/path/to/aoci
"$AOCI" --repo . capabilities                      # 当前二进制提供的能力
"$AOCI" --repo . doctor                            # 仓库与宿主集成诊断
"$AOCI" --repo . verify                            # 缺失 / 孤儿 / 过期 / 未定基线
"$AOCI" --repo . check                             # 聚合治理门禁
"$AOCI" --repo . index agent guide --agent codex --json   # 实时指南
```

**不要**把内部状态机复制进包装脚本——按运行中二进制返回的指南走。

## 数据库认知（可选）

先建代码索引，再建数据库索引。声明数据源的非敏感身份，凭据由数据库管理员在
**外部环境**提供（AOCI 不保存、不读取凭据值）：

```bash
aoci --repo . database source add \
  --source-id primary --engine postgresql --database-name app --namespace public
aoci --repo . database source access --source primary --json   # 只查引用是否已提供
```

只读基础表的**结构元数据**，不读业务行、不执行 DDL / DML。
支持 PostgreSQL、MySQL，以及受限的 openGauss 6.0.5。
