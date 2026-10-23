# 共享目录与注册项目的访问、执行权限

本指南规定初始化 Agent 必须完成的权限配置。初始化时保存到 `~/.ai-global/PERMISSIONS.md`，补充本机实际客户端、配置位置和适配结果；后续维护使用共享目录内的版本，不依赖搭建图纸。

## 目标与范围

**访问权限与脚本执行权限都要配置，且对所有实际接入工具生效。** 不把“允许读文件”当成“允许执行脚本”，也不在日常使用时重复申请同一范围的授权。

| 授权对象 | 范围与要求 |
|----------|------------|
| 共享目录 | `~/.ai-global/**`，含 skill、规则、配置和后续创建的辅助脚本 |
| 已注册项目 | `registry.json` 中所有 `code_path`，位于共享目录外也同样覆盖 |
| 用户创建的独立辅助脚本 | 通用脚本进共享 `scripts/`，项目专属脚本进 `projects/<id>/scripts/` |
| 指针与目标 | 各工具指针文件及其明确指向的文件/目录，不扩大为整个主目录 |

访问包括读取、创建和修改文件；脚本执行包括 `python3 <脚本>`、`bash <脚本>` 和运行已有执行权限的入口文件。项目源码、构建脚本仍留在已注册仓库，skill 自带脚本仍随 skill 存放。

**用户注册项目，即将该项目纳入所有接入工具的访问、执行授权范围。** 不因它位于共享目录之外而再次询问是否允许访问。

权限按现有真实路径配置，不创建额外工作区、不要求切换工作区，也不为授权迁移已有脚本、skill 或项目。

## 初始化与后续同步

先复用本指南已确认的配置方法，只检查本机现有值、客户端版本及实际生效情况；不要在每次初始化时从头重复检索所有官方文档。只有原生校验失败、版本变化或适配缺失时，再针对对应问题核对，并把结论补回指南。

1. 从 `~/.ai-global/registry.json`、实际工具清单和指针目标汇总范围，展开为本机绝对路径，检查真实路径，不凭符号链接假定覆盖目录外目标。
2. 按实际客户端版本，分别配置文件访问、写入、脚本执行及适用的沙箱目录。修改已有配置时合并条目，保留无关设置、已有 deny 规则和组织策略。
3. 优先使用原生目录范围；若执行规则只支持精确脚本路径，则枚举授权范围内脚本生成规则。使用规范化的绝对脚本路径调用，不依赖 `~`、变量或目录通配被命令规则自动展开。
4. 新增项目时同步所有工具的目录和脚本规则；新建脚本时由创建它的 Agent 补齐需要精确路径的工具规则，不再逐个请求用户授权。
5. 接入新工具时覆盖全部既有范围；项目或脚本迁移时同步新路径，只清理本方案生成的失效条目。
6. 将实际配置位置、重载方法及尚未完成的适配记录在共享目录内的本指南中。需要客户端重载时执行其支持的重载流程，不假定修改文件会即时生效。

这套同步规则必须写进生成后的 BOOTSTRAP。图纸归档后，维护 Agent 从共享目录读取工具清单与本指南，不回到图纸目录查配置。

下面的 `/home/user`、`/work/example` 都是示例，初始化时替换为实际绝对路径。路径包含空格时，命令规则要与实际调用的引用形式一致。

## Codex

目录访问和命令执行分别配置。

在 `~/.codex/config.toml` 中合并共享目录与所有项目的可写范围；以下为支持该配置格式的客户端示例：

```toml
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
writable_roots = ["/home/user/.ai-global", "/work/example"]
```

本机 Codex 0.163 的离线沙箱验证已确认支持命名权限 profile；使用该版本时可选择以下格式，而不是把旧版配置与新格式混用：

```toml
default_permissions = "ai-shared"

[permissions.ai-shared.filesystem]
"/" = "read"
"/home/user/.ai-global" = "write"
"/work/example" = "write"
```

按本机路径补齐已注册项目、明确的指针文件与必要的系统临时目录。使用 `codex sandbox -P ai-shared -- <无副作用命令>` 验证；此版本不使用 `codex sandbox macos` 子命令。权限 profile 只授予列出的写入位置，新项目注册后同步补齐。

在 `~/.codex/rules/ai-shared.rules` 中生成脚本执行规则，避免覆盖用户已有的 `default.rules`：

```python
prefix_rule(
    pattern = [
        "python3",
        [
            "/home/user/.ai-global/scripts/example.py",
            "/work/example/scripts/example.py",
        ],
    ],
    decision = "allow",
    justification = "共享目录和已注册项目中的用户脚本",
)

prefix_rule(
    pattern = ["bash", "/home/user/.ai-global/scripts/example.sh"],
    decision = "allow",
    justification = "共享目录中的用户脚本",
)
```

`prefix_rule.pattern` 匹配命令参数的字面量前缀，**不是文件系统 glob**，不能写成 `["python3", "~/.ai-global/**/*.py"]` 来表示全部脚本。按实际解释器、脚本路径生成规则；直接执行的入口文件也生成其精确命令规则。

新增脚本时更新规则并按客户端要求重载。不要用 `["python3"]`、`["bash"]` 或关闭全部审批和沙箱来代替目录范围授权。

## Claude Code

在用户级 `~/.claude/settings.json` 合并额外工作目录、文件权限和 Bash 执行规则。目录用于文件范围，Bash 规则另行覆盖脚本调用：

```json
{
  "permissions": {
    "additionalDirectories": [
      "/home/user/.ai-global",
      "/work/example"
    ],
    "allow": [
      "Read(~/.ai-global/**)",
      "Edit(~/.ai-global/**)",
      "Read(//work/example/**)",
      "Edit(//work/example/**)",
      "Bash(python3 /home/user/.ai-global/*)",
      "Bash(bash /home/user/.ai-global/*)",
      "Bash(python3 /work/example/*)",
      "Bash(bash /work/example/*)"
    ]
  }
}
```

`Read`、`Edit` 的绝对路径使用 `//` 前缀，`~/` 表示用户主目录；`Edit` 同时覆盖内置的文件编辑和创建工具，不使用无效的 `Write(path)` 文件规则。Bash 中的 `*` 是命令文本匹配，不与文件路径的 `**` 混用。

执行前将脚本路径规范化；已有直接执行入口或其他解释器时补齐对应规则。规则只覆盖批准便利性，不取代沙箱的文件系统约束。

## Qoder

### CLI

在 `~/.qoder/settings.json` 合并目录和工具规则。按当前客户端支持的格式生成，例如：

```json
{
  "permissions": {
    "trustDirectories": [
      "/home/user/.ai-global",
      "/work/example"
    ],
    "additionalDirectories": [
      "/home/user/.ai-global",
      "/work/example"
    ],
    "allow": [
      "Read(/home/user/.ai-global/**)",
      "Edit(/home/user/.ai-global/**)",
      "Write(/home/user/.ai-global/**)",
      "Read(/work/example/**)",
      "Edit(/work/example/**)",
      "Write(/work/example/**)",
      "Bash(python3 /home/user/.ai-global/scripts/example.py:*)",
      "Bash(bash /work/example/scripts/example.sh:*)"
    ]
  }
}
```

脚本规则覆盖实际脚本清单，新增脚本时同步补齐。`accept_edits` 不能代替 Bash 执行授权；仅会话级的 `--allowed-tools`、`--add-dir` 也不能代替持久配置。

### IDE 与插件

使用实际客户端的访问权限、工作目录和终端允许规则分别配置同一范围，记录持久化方式。不能把 CLI 配置直接当成 IDE 配置，也不能把 `bwrap` 当作所有平台和客户端统一的权限机制。

本机 Qoder IDE 已确认的独立配置位于 `~/Library/Application Support/Qoder/User/settings.json`：

```json
{
  "app": {
    "configChatTerminalRunMode": "askEveryTime",
    "configChatCommandAllowlist": "python3 /home/user/.ai-global/scripts/example.py,bash /work/example/scripts/example.sh"
  }
}
```

这是白名单模式：匹配命令自动执行，其他命令继续询问。保留原 `configChatCommandDenyList`，不改为 fullAccess。共享目录及注册项目的跨目录访问另用客户端原生目录权限配置，不能用命令白名单替代沙箱目录授权，也不能通过创建工作区或搬迁文件来回避权限配置。缺失能力必须明确记录。

原生安全检查可能高于 allow 规则；初始化时明确具体限制和未完成项，不用全局 Full access 或 YOLO 冒充目录范围配置。

## GitHub Copilot

### CLI

先用该版本的 `copilot help config`、`copilot help permissions` 确认格式。支持 `trustedFolders` 的版本可持久添加目录，例如：

```bash
copilot config trustedFolders /home/user/.ai-global
copilot config trustedFolders /work/example
```

目录信任与工具批准分开处理：继续配置文件写入和实际脚本命令的持久允许规则，不能只配置目录就标记执行权限完成。`--add-dir` 是启动时的目录范围，不等于所有写入和 shell 调用均已批准。

命令规则按本机版本支持的格式生成；不要臆造 `allowedTools` 等未确认字段。若某版本只支持解释器级命令允许，不把全局 `shell(python3:*)` 或 `shell(bash:*)` 宣称为目录级脚本授权；按原生能力补齐，无法表达的部分明确记入交付缺口。

已确认的另一种原生适配是用户级 `~/.copilot/hooks/*.json`：

```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "exec": "/usr/bin/python3",
        "args": ["/home/user/.ai-global/bin/copilot-permission-hook.py"],
        "timeoutSec": 10
      }
    ]
  }
}
```

初始化时生成处理器，按官方 `cwd/toolName/toolArgs` 输入校验注册目录和指针文件，仅对范围内的文件操作与简单脚本调用返回 `{"permissionDecision":"allow"}`；其他调用返回 `{}`，留给原生流程。必须校验实际路径和符号链接，不能放行整个解释器、解释器内联代码或沙箱绕过请求。处理器错误需显式报错，不能返回允许。

此方式已在本机 Copilot CLI 新会话中完成跨注册项目的创建文件与新脚本调用。验证时不使用 allow-all，且不要把可用文件工具硬限制成某一个名称，以免过滤掉当前模型实际使用的编辑工具。只写 trustedFolders 不等于完成工具批准。

### VS Code

按共享目录及已注册项目的现有路径配置原生文件访问范围，不生成新的工作区文件。仅配置 `chat.additionalReadAccessFolders` 只扩展读取，不能代替写入范围；跨目录写入受限时应明确其原生能力缺口。

在用户设置中合并文件编辑与终端允许规则。以下演示精确脚本规则，实际应覆盖范围内脚本清单：

```json
{
  "chat.tools.edits.autoApprove": {
    "/home/user/.ai-global/**": true,
    "/work/example/**": true
  },
  "chat.tools.terminal.enableAutoApprove": true,
  "chat.tools.terminal.autoApprove": {
    "/^python3 \\/home\\/user\\/\\.ai-global\\/scripts\\/example\\.py(?:\\s.*)?$/": true,
    "/^bash \\/work\\/example\\/scripts\\/example\\.sh(?:\\s.*)?$/": true
  }
}
```

使用默认的逐子命令审批，不通过整行匹配把未授权的复合命令一起放行；启用终端沙箱时也同步其目录范围。Workspace Trust 本身不是终端执行白名单。

### JetBrains

按实际插件版本的原生权限界面或公开配置接口，分别持久配置目录文件访问、文件操作与脚本终端执行。范围包括共享目录、全部已注册项目和明确的指针目标，不能照搬 VS Code 设置。

已在本机现有配置中确认的格式：`options/github-copilot.xml` 的 `github-copilot` component 内，`terminalAutoApprove/map/entry` 使用 `key="/正则/"`、`value="true"` 表示允许。只追加已授权脚本的精确匹配项，保留原有 `value="false"`，不启用 `autoApproveUnmatchedTerminal`。修改磁盘配置后需重新加载 IDE，并检查是否被内存配置覆盖。

若当前插件版本没有某项持久范围授权能力，记录具体版本、受限操作和缺口；不能把工作区信任当作已经完成执行授权，也不猜测内部配置字段或关闭所有安全检查。

## 交付标准

- 每个实际客户端分别确认：读取共享文件、创建/修改辅助文件、通过 `python3` 和 `bash` 执行无副作用脚本。
- 选择一个共享目录外的已注册项目，确认相同的文件访问和脚本执行范围已配置。
- 重开会话后配置仍生效；新增脚本及新增项目按生成后的同步规则补齐，不重复询问已有授权。
- 本指南记录各客户端原生配置位置、覆盖范围、重载方式和未完成项；只有访问与执行均已落地的客户端才标为完成。

已有 deny、组织策略及原生安全限制仍需遵守；不把权限配置扩大到整个主目录、整个文件系统或任意解释器调用。运行脚本的工具权限与具体业务操作的确认规则分别保留。

## 原生格式参考

- [Codex 命令规则](https://developers.openai.com/codex/rules/)
- [Codex 配置 schema](https://github.com/openai/codex/blob/main/codex-rs/core/config.schema.json)
- [Claude Code 权限](https://code.claude.com/docs/en/permissions)
- [Qoder CLI 权限](https://docs.qoder.com/cli/permissions.md)
- [Qoder 访问权限](https://docs.qoder.com/qoder/approval-and-sandbox.md)
- [VS Code 审批配置](https://code.visualstudio.com/docs/agents/run/approvals)
- [Copilot IDE Agent](https://docs.github.com/en/copilot/how-tos/copilot-in-your-ide/use-copilot-agents/use-agent-mode)
