# OARSS 项目术语表  
**Open Agent Role-Skill Spec 术语文档**  
**版本**：1.0（2026年3月）  
**维护**：OARSS 核心团队  
**目的**：统一项目内及社区对 Agent 生态核心概念的理解，避免歧义，提高沟通效率。

## 核心层级概念（由底层到顶层）

| 中文术语          | 英文术语                  | 定义（简）                                                                 | 详细说明与上下文                                                                 | 常见混淆 / 错误用法                              | 对应 metadata.type 值 |
|-------------------|---------------------------|----------------------------------------------------------------------------|----------------------------------------------------------------------------------|---------------------------------------------------|-----------------------|
| 执行环境 / 运行时 | Runtime / Agent Runtime   | 负责观察-思考-行动循环、工具调度、内存管理、状态持久的“空壳”执行框架       | Claude Code、Cursor、OpenClaw、LangGraph executor、Gemini CLI 等都属于这一层     | 把带有人设的完整实体也叫 runtime                  | （无）                |
| 工具 / 原子能力   | Tool / Function / Action  | 最底层的、可被模型调用的单个动作或接口，通常是函数调用或脚本执行           | browser-navigation、git-commit、terminal-exec 等                                 | 把带流程判断的 skill 也叫 tool                    | tool                  |
| 流程 / 步骤       | Procedure / Workflow      | 带有明确步骤、检查清单、决策分支的结构化工作流程                           | code-review-checklist、refactor-patterns、security-audit-flow                   | 与 skill 边界模糊                                 | procedure             |
| 技能 / 专家技能   | Skill                     | 领域特定的最佳实践、判断逻辑、模板 + 指导说明的模块化能力包                 | react-best-practices、typescript-patterns、supabase-postgres-best-practices      | 把完整角色也叫 skill（最常见混淆）               | skill                 |
| 角色 / 人设 / 角色包 | Role / Persona / Persona Bundle | 完整的人设封装，包括性格、目标、行为准则、背景故事 + 一组子技能依赖       | senior-fullstack-engineer、loving-husband、security-auditor                     | 把 runtime 直接叫 agent，把 persona 叫 agent      | persona-bundle        |

## 其他重要术语

| 中文术语              | 英文术语                        | 定义（简）                                                                 | 详细说明与上下文                                                                 | 常见误用示例                                      |
|-----------------------|---------------------------------|----------------------------------------------------------------------------|----------------------------------------------------------------------------------|---------------------------------------------------|
| 子技能                | sub_skill / sub-skills          | 一个 Persona Bundle 所依赖的、可独立存在的 Skill / Procedure / Tool         | 通过 metadata.sub_skills 数组声明，支持版本约束                                 | 把所有东西都塞进 persona 里，不做子技能拆分      |
| 元数据扩展            | metadata extension              | 在 SKILL.md YAML frontmatter 中自定义的字段，用于实现 OARSS 层级区分        | type、role_category、persona_backstory、sub_skills、trigger_scenarios 等         | 直接修改 Anthropic 标准核心字段（会破坏兼容性）   |
| 触发场景              | trigger_scenarios               | 当满足某些关键词、上下文或意图时，runtime 应激活该 Skill / Persona 的条件   | 用于渐进加载和自动切换                                                           | 只写 description，不写 trigger_scenarios          |
| 渐进加载              | Progressive Disclosure          | runtime 先加载轻量信息（name + description），需要时才加载完整内容          | Anthropic Agent Skills 标准核心机制，OARSS 强烈推荐遵守                         | 所有内容都写在 SKILL.md 开头，导致 token 爆炸     |
| 镜像 / 私有仓库       | Mirror / Private Registry       | 对官方或社区技能源的缓存、审核、签名、私有化部署版本                        | 支持企业内网离线使用、合规审查、加速拉取                                         | 直接 fork 所有内容，而不是只镜像 metadata + CID   |
| 内容寻址              | Content Addressing              | 使用哈希（如 IPFS CID）标识内容，确保不可篡改、版本唯一                     | 未来 v2.0 方向，用于去中心化分发                                                 | —                                                 |
| 兼容运行时            | compatible_runtimes             | 该 Skill / Persona 可被哪些 agent 执行环境正确加载和执行                     | 值示例：["*", "claude-code", "cursor", "openclaw"]                               | 写死特定运行时名称，导致跨平台兼容性下降          |

## 推荐的命名约定（kebab-case）

- 文件夹名 / name 字段：全部小写、连字符分隔  
  正确：senior-fullstack-engineer、react-best-practices  
  错误：SeniorFullstackEngineer、reactBestPractices

- 分类标签（role_category 等）：中文或英文均可，但建议保持一致  
  推荐：["开发-全栈", "前端", "工程师"] 或 ["fullstack", "frontend", "engineering"]

## 常见错误用法对照表（高频踩坑）

| 错误说法 / 错误用法                          | 正确 / 推荐说法                                      | 原因说明                                                                 |
|-----------------------------------------------|-------------------------------------------------------|--------------------------------------------------------------------------|
| “这个 agent 很会写 React 代码”                | “这个 persona 在写 React 时会加载 react-best-practices” | agent/runtime 是中性执行器，能力来自 persona/skill                      |
| “我写了一个 skill 叫 loving-husband”          | “我写了一个 persona-bundle 叫 loving-husband-role”   | skill 是模块，persona 才是完整角色                                       |
| “把所有最佳实践都写到一个 SKILL.md 里”        | “拆分成多个 sub_skill，由 persona 协调调用”           | 违背模块化 + 渐进加载原则，token 效率低下                                 |
| “runtime 自己知道什么时候用什么 skill”        | “靠 trigger_scenarios + description 提示 runtime”    | 大多数 runtime 目前依赖自然语言匹配或显式触发                             |

## 版本历史

- 2026-03：v1.0 初稿，建立四层结构 + sub_skills 概念
- 后续计划：根据社区反馈补充更多领域术语（非 dev 场景，如教育、医疗、家庭等）

欢迎在 issues 或讨论区提出新增术语、修改定义、补充反例。

最后更新：2026年3月
