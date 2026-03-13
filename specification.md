# Open Agent Role-Skill Spec (OARSS) v1.0

**正式规范名称**：Open Agent Role-Skill Spec  
**版本**：1.0  
**发布日期**：2026 年 3 月  
**状态**：草案 → 1.0 候选（征求社区反馈后可标记稳定）  
**GitHub 建议仓库**：`open-agent-role-skill / oarss-spec`  
**网站建议**：`oarss.dev` 或 `spec.oarss.org`（未来可建）

## 1. 愿景与核心价值（为什么存在 OARSS）

在 2026 年的 Agent 生态中，我们已经拥有 Anthropic Agent Skills 的开放文件夹格式（SKILL.md + 资源），以及 Vercel skills.sh 的安装生态。但当前最大的缺失是：

- **缺乏以“人/角色”为中心的组织方式**：现有技能多为原子工具或孤立流程，缺少“完整专家人设”的封装。
- **复用性与维护性不足**：同一个最佳实践（如 React 性能优化）被重复写入多个 skill，升级困难。
- **Token 与加载效率问题**：大而全的 skill 导致 context 膨胀。
- **企业/社区 mirror 需求**：需要清晰的分层、依赖声明、版本约束与私有化支持。

**OARSS 的终局目标**：

构建一个**开放、不锁定任何 runtime** 的 Agent 能力分发标准，让开发者/企业能够：

- 搜索“资深全栈工程师” → 一键安装完整角色包（persona-bundle）
- 该角色包在 Cursor / Claude Code / OpenClaw / Gemini CLI / Copilot 等任意支持 SKILL.md 的环境中自动生效
- 角色包本身轻量，通过 **sub_skills** 依赖模块化子技能，实现高复用、低消耗、可独立迭代
- 支持企业私有 mirror（审核、签名、离线、加速），避免中心化单点风险

一句话：**从“工具集合”升级到“可切换的专业角色技能栈”**，真正实现“同一个 Runtime，不同人生/职业角色自由切换”。

## 2. 核心设计原则

1. **100% 兼容 Anthropic Agent Skills 开放标准**（SKILL.md 文件夹格式、progressive disclosure 机制）
2. **Role / Persona 第一公民**：registry 的核心商品是 persona-bundle
3. **模块化分层**：persona-bundle → sub_skills → procedure/skill/tool
4. **渐进加载友好**：persona 保持轻量（<300 tokens brief），子技能按需加载
5. **可镜像、可签名、可版本化**：借鉴 npm / cargo / apt 的依赖与分发模型
6. **Dev 场景优先**，但设计通用（家庭、教育、法律等角色未来可扩展）

## 3. 四层 artifact 类型（必须在 metadata.type 中声明）

| 类型标识              | metadata.type 值     | 主要职责                                   | 典型体积（brief → full） | 是否可作为 sub_skill | 典型使用场景示例（2026 dev 导向） |
|-----------------------|----------------------|--------------------------------------------|---------------------------|-----------------------|-----------------------------------|
| Tool                  | `tool`               | 原子、可执行接口 / 脚本                    | 50–300 → 800              | 是                    | browser-navigation, git-operations |
| Procedure             | `procedure`          | 结构化步骤、checklist、决策流程            | 200–600 → 1800            | 是                    | code-review-checklist, refactor-steps |
| Skill                 | `skill`              | 领域最佳实践 + 判断逻辑 + 模板            | 300–800 → 2500            | 是                    | react-best-practices, typescript-patterns |
| Persona Bundle        | `persona-bundle`     | 完整人设 + backstory + 协调逻辑 + 子技能依赖 | 150–400 → 3500            | 否（顶层）            | senior-fullstack-engineer, loving-husband-role |

## 4. 文件夹结构（必须遵守）

```text
skill-or-persona-name/               # kebab-case，全小写，连字符分隔
├── SKILL.md                         # 必须，YAML frontmatter + Markdown
├── scripts/                         # 可选，可执行代码（.py / .js / .sh 等）
├── references/                      # 可选，额外文档、模板、示例
├── assets/                          # 可选，图片、配置文件模板等
└── ...                              # 其他辅助文件（不做强制约定）
```

## 5. SKILL.md 格式要求（YAML frontmatter + Markdown 正文）

### 5.1 必填字段（所有类型）

```yaml
---
name: senior-fullstack-engineer                  # 必填，与文件夹名一致，kebab-case
description: >-                                  # 必填，简短触发描述，<100 字
  当涉及 React/Next.js + Node/Supabase 全栈开发时激活。扮演 8 年经验全栈工程师。
metadata:
  type: persona-bundle                           # 必填，四种之一
  version: 1.0.0                                 # 推荐，semver
  license: MIT                                   # 推荐
---
# 正文（Markdown，支持标题、列表、代码块）
当触发时，你应该……
```

### 5.2 Persona Bundle 专用推荐字段

```yaml
metadata:
  type: persona-bundle
  role_category:                                 # 数组，分类标签，便于搜索
    - 开发-全栈
    - 工程师
  persona_backstory: |                           # 多行字符串，人设描述
    你是一位有 8 年经验的全栈工程师，擅长……
  trigger_scenarios:                             # 数组，触发提示词/场景
    - "React / Next.js / TypeScript / Supabase"
    - "写组件 / API / 数据库 / review / refactor"
  sub_skills:                                    # 核心！依赖声明
    - name: react-best-practices
      version: ^1.2
      required: true
    - name: supabase-postgres-best-practices
      version: ">=2.0"
      required: true
    - name: code-review-checklist
      version: any
      optional: true
  compatible_runtimes:                           # 兼容声明
    - "*"
    - claude-code
    - cursor
  estimated_tokens:
    brief: 180                                   # 轻量描述加载量
    full: 4200                                   # 完整加载量（含子技能建议）
```

### 5.3 子技能（skill / procedure / tool）常用字段

```yaml
metadata:
  type: skill
  category:                                      # 辅助分类
    - frontend
    - react
  trigger_scenarios:
    - "写 React 组件"
    - "性能优化"
    - "re-render 问题"
```

## 6. 依赖管理与加载规则（规范强制约定）

1. **persona-bundle 必须声明 sub_skills**（至少 1 个推荐）
2. **runtime / client 应支持递归加载**：加载 persona → 解析 sub_skills → 按需加载 required 子技能
3. **版本约束**：支持 semver 范围（^1.0, ~1.2, >=2.0, any）
4. **optional vs required**：optional 子技能可提示用户是否安装
5. **冲突处理**：同名不同版本 → 优先使用已安装最高兼容版本，或提示用户

## 7. 兼容性声明

- **必须兼容**：Anthropic Agent Skills spec（https://agentskills.io/specification）
- **已验证兼容 runtime**（截至 2026-03）：Claude Code、Cursor、Vercel-supported agents、OpenClaw 等支持 SKILL.md 的环境
- **未来扩展**：可通过 metadata.compatible_runtimes 声明特定优化

## 8. 安全与 mirror 考虑

- **推荐**：所有发布的 skill 使用 cosign / sigstore 签名
- **mirror 友好**：persona-bundle 只需存储 metadata + 轻量正文，子技能可独立缓存
- **私有化**：企业 mirror 可强制审核 sub_skills，覆盖官方版本

## 9. 示例（简要）

**persona-bundle 示例**：senior-fullstack-engineer  
（完整文件夹结构与内容见后续 examples 仓库）

**子技能示例**：react-best-practices  
（type: skill，包含性能规则、hooks 模式、代码模板等）

## 10. Roadmap（v1.x → v2.0）

- v1.1：CLI 参考实现规范（install / deps / mirror）
- v1.2：多语言支持（SKILL.zh.md 等）
- v2.0：经济模型字段（tip / bounty）、P2P 分发（IPFS CID）
