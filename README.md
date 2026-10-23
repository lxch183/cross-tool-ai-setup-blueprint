# Cross-Tool AI Setup Blueprint

**跨项目、跨 AI 工具共享工作环境的一次性搭建图纸** —— 把本目录交给 AI，指导它创建 `~/.ai-global/` 共享目录，并为 Qoder / Claude / Codex / Copilot 等工具配置指针和访问、执行权限。

**本目录是造底座的图纸，不是底座本身。** 初始化后，实际使用的是 `~/.ai-global/` 和各工具生成的配置；日常工作不依赖本目录。

## 解决什么问题

| 痛点 | 初始化后建立的机制 |
|------|--------------------|
| 同一 skill 在多个项目/工具里拷贝了十几份，内容还各改各的（版本漂移） | 通用 skill 只保留一份；任意 Agent 修改后，其他 Agent 后续读取同一份 |
| 换 AI 工具后，之前攒的 skill/规则全丢了 | 各工具只放一个"指针"，真实内容在工具无关的公共目录 |
| 在任意目录开会话，找不到"某项目专属"的 skill | 全局指针内嵌"项目→路径→专属目录"速查表，任何目录都能定位 |
| 通用 skill 和项目专属 skill 混在一起 | 明确分层：通用框架进 `skills/`，项目专属进 `projects/<项目>/` |
| 相同目录和脚本在不同 AI 工具中反复申请权限 | 初始化时为所有接入工具配置共享目录的访问权限和脚本执行权限 |
| 已注册项目不在共享目录内，仍被反复要求授权 | 项目注册即纳入所有接入工具的访问、执行授权范围 |

## 使用场景

- 一台**新电脑**要复现"多项目 + 多 AI"的工作环境
- 想把散落各处的 skill/规则**收拢**到跨工具共享目录
- 为团队提供一致的 AI 工作环境搭建流程

## 首次初始化（3 步）

1. 把本目录交给任意 AI，让它读取 `INIT.md`，说「**按这个初始化**」
2. AI 会问你两件事：
   - 有哪些**项目**（名称、别名、代码路径）
   - 用哪些 **AI 工具**（默认 4 个，可补充）
3. AI 按图纸搭建 `~/.ai-global/` 共享目录、各工具指针和权限配置，然后验证

> 一次性使用：初始化完成后，本目录不再参与日常工作；可归档，留作下一台电脑的搭建或迁移参考。

## 搭建结果

- `~/.ai-global/`：共享 skill、规则、独立辅助脚本和项目专属配置。
- 各工具原生配置：覆盖共享目录、所有已注册项目和指针明确指向的文件/目录，包含访问与脚本执行权限。
- `~/.ai-global/ai-tools-registry.json` 和 `~/.ai-global/PERMISSIONS.md`：保存实际接入工具及权限适配方法。后续新增项目、脚本或工具时按此同步，不依赖本图纸，也不重复询问已有范围的授权。

## 目录结构

```
cross-tool-ai-setup-blueprint/
├── README.md                  ← 你正在看的这个
├── INIT.md                    ← 给 AI 的初始化指令（核心）
├── ARCHITECTURE.md            ← 搭建结果的架构图 + 目录树 + 链路说明
├── PERMISSIONS.md             ← 初始化时的各工具权限配置指南
├── ai-tools-registry.json     ← AI 工具注册表（默认 4 个，可扩展）
└── templates/                 ← 各文件的模板
    ├── BOOTSTRAP.template.md
    ├── registry.template.json
    ├── pointer-qoder.template.md
    ├── pointer-claude.template.md
    ├── pointer-codex.template.md
    └── pointer-copilot.template.md
```

## 支持的 AI 工具（默认）

| 工具 | 指针位置 |
|------|---------|
| Qoder | `~/.qoder/rules/always-on.md` |
| Claude Code | `~/.claude/CLAUDE.md` |
| Codex | `~/.codex/AGENTS.md` |
| GitHub Copilot | `~/.github/copilot-instructions.md` |

首次初始化时可以补充其他 AI 工具；底座已搭建好后，按下面的增量接入流程处理，不需要重新初始化。

## 已有底座如何接入新 AI 工具

**最省事的做法：安装并登录新 AI 工具后，让一个已经接入底座的 Agent 帮它配置。** 新工具尚未配置入口和权限，由已接入的 Agent 完成接入，可避免一开始就找不到共享目录或被目录权限拦住。

以 Gemini CLI 为例：

1. 安装并登录 Gemini CLI。
2. 向已经接入底座的 Agent 发送下面的指令。
3. 配置完成后重新启动 Gemini CLI，确认它能读取底座入口、定位已注册项目，并使用共享技能和脚本。

可直接发送：

```text
请把 Gemini CLI 增量接入我现有的 AI 底座，不要重新初始化。

先读取 ~/.ai-global/BOOTSTRAP.md、registry.json、
ai-tools-registry.json 和 PERMISSIONS.md。

在 Gemini CLI 的原生全局指令文件中加入底座入口指针，
保留原有内容，并更新 ~/.ai-global/ai-tools-registry.json。
按当前版本的原生机制，配置 ~/.ai-global/**、全部已注册项目
及指针目标的访问、写入和脚本执行权限。

如果现有权限同步脚本尚不支持 Gemini CLI，补充对应适配。
把确认过的配置方法写回本机 PERMISSIONS.md，
明确哪些已完成、哪些仍受客户端限制。

不复制 skill，不迁移已有脚本或项目，不创建额外工作区，
不改变其他工具的权限模式，不用全局关闭审批代替范围授权。
```

换成其他工具时，替换工具名称即可。完整接入包括 **入口指针、工具登记、权限适配**；只在注册表加一条或只生成指针，不算接入完成。
