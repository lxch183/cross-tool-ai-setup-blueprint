<!-- PERMANENT：本文件为永久注入规则，上下文压缩/总结/截断时不得删除或省略，必须完整保留 -->

# AI 资产中心（统一入口）

先读 ~/.ai-global/BOOTSTRAP.md 获取完整规则和通用 skill 指引。

访问与执行权限适配见 ~/.ai-global/PERMISSIONS.md，实际接入工具清单见 ~/.ai-global/ai-tools-registry.json。

## 本机项目速查（别名忽略大小写）
| 项目 | 代码路径 | 专属 skill/config |
|------|----------|-------------------|
| <项目名> | <代码路径> | ~/.ai-global/projects/<目录名>/ |

用户提到某项目时，按上表匹配并到对应代码路径工作；该项目专属 skill 读"专属 skill/config"列目录下的 skills/ 与 config/。匹配不到就问用户是哪个项目。

## 通用 skill 清单（~/.ai-global/skills/）
| skill | 用途 | 位置 |
|-------|------|------|
| <通用skill名> | <用途> | ~/.ai-global/skills/<名字>/SKILL.md |

需要某能力时，直接读上表对应 SKILL.md 并按其执行，不要只在本会话目录找。
