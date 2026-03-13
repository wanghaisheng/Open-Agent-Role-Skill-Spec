**OARSS 规范详细解析（v1.0）**  
**Open Agent Role-Skill Spec**  
**发布日期**：2026年3月  
**状态**：正式候选版（已落地 3 个高质量示例 + Registry Index Schema + CLI 伪代码）

### 1. 背景与核心目的（为什么需要 OARSS）

2026 年 3 月的 Agent 生态已经非常繁荣：
- **Anthropic Agent Skills** 提供了开放的 SKILL.md 文件夹格式（被 Claude Code、Cursor、OpenClaw、Gemini CLI 等 20+ runtime 原生支持）。
- **Vercel skills.sh** 提供了安装体验。
- 但存在**三大根本问题**：
  1. 技能碎片化：大部分是原子 tool 或孤立流程，缺少“完整的人设（Persona）”。
  2. 复用性差：同一个 React 最佳实践被重复写在多个角色里。
  3. Token 效率低 + 概念混乱：role、skill、tool 混为一谈，导致企业 mirror 难以管理。

**OARSS 的终极目标**：  
把“Agent 能力”从“工具集合”升级为**可切换的专业角色技能栈**（Role/Persona 第一公民）。  
同一个 runtime（Cursor / Claude Code / OpenClaw 等）可以随时切换“资深全栈工程师”“温柔丈夫”“AI 时代产品经理”等完整人设，自动加载对应的子技能栈。

一句话：**OARSS 是 Anthropic 标准的“角色化 + 模块化 + 可镜像”扩展版**，彻底解决概念混乱与复用问题。

### 2. 与现有标准的对比

| 维度               | Anthropic Agent Skills | Vercel skills.sh | OARSS（v1.0）                  |
|--------------------|------------------------|------------------|--------------------------------|
| 核心商品           | Skill 文件夹           | 命令式 skill     | **Persona Bundle（角色包）**   |
| 分层               | 扁平（全叫 skill）     | 无明确分层       | **严格四层**（Tool → Procedure → Skill → Persona） |
| 依赖管理           | 无                     | 简单             | **sub_skills + semver**（必填/可选） |
| mirror 支持        | 弱                     | 弱               | 原生支持（index.json + config.toml） |
| 搜索能力           | 仅 description         | 基本             | **语义搜索**（role_category + trigger_scenarios） |

### 3. 核心四层模型（OARSS 最重要创新）

| 层级              | metadata.type 值     | 职责与特点                                   | 是否可被其他角色复用 | 典型体积（brief → full） | 例子（我们已落地）                  |
|-------------------|----------------------|----------------------------------------------|----------------------|---------------------------|-------------------------------------|
| Tool              | `tool`               | 原子动作、脚本、API 调用                     | 是                   | 50–300 → 800              | browser-navigation                  |
| Procedure         | `procedure`          | 步骤/checklist/决策流程                      | 是                   | 200–600 → 1800            | code-review-checklist               |
| Skill             | `skill`              | 领域最佳实践 + 判断逻辑 + 模板               | 是（高复用）         | 300–800 → 2500            | react-best-practices                |
| Persona Bundle    | `persona-bundle`     | **第一公民**：人设 + backstory + 协调逻辑 + 子技能依赖 | 否（顶层）           | 150–400 → 3500            | senior-fullstack-engineer<br>loving-husband-role<br>product-manager-role |

**设计哲学**：Persona 只做“大脑与协调”，真实能力全部下沉到独立的 sub_skills，实现**模块化 + 渐进加载 + 高复用**。

### 4. 文件夹结构（必须严格遵守）

```text
角色或技能名/                  # kebab-case，与 name 字段完全一致
├── SKILL.md                   # 唯一必填文件
├── scripts/                   # 可选：辅助脚本（.ts / .py 等）
├── references/                # 可选：模板、文档
└── assets/                    # 可选：图片、配置文件
```

### 5. SKILL.md 格式详细字段解析（YAML frontmatter）

#### 5.1 所有类型必填字段
- `name`：与文件夹名一致（kebab-case）
- `description`：触发关键词（<100 字），runtime 靠此决定是否加载
- `metadata.type`：**核心区分字段**（tool / procedure / skill / persona-bundle）
- `version`、`license`、`compatible_runtimes`

#### 5.2 Persona Bundle 专用字段（重点解析）
```yaml
metadata:
  type: persona-bundle
  role_category: ["开发-全栈", "工程师"]          # 数组：分类标签，用于搜索
  persona_backstory: |                           # 多行字符串：完整人设描述
    你是一位 8–10 年经验的全栈工程师...
  trigger_scenarios:                             # 数组：触发条件（最重要！）
    - "React / Next.js / TypeScript / Supabase"
    - "代码 review / refactor"
  sub_skills:                                    # 依赖数组（核心机制）
    - name: react-best-practices
      version: "^1.3"
      required: true
  estimated_tokens: { brief: 220, full: 4800 }   # 帮助 runtime 做 token 预算
```

**每个字段的“为什么”**：
- `role_category`：让 registry 支持 `oar search "丈夫"` 或 `oar search "全栈"`
- `trigger_scenarios`：runtime 自动激活的关键（渐进加载基础）
- `sub_skills`：实现“同一个男人不同角色”——工程师角色依赖 5 个子技能，丈夫角色依赖另外 5 个，完全独立维护

### 6. sub_skills 依赖机制详细说明

- 支持 semver 范围（^1.0、>=2.1、any）
- `required: true/false`：必装 vs 可选
- **加载流程**（runtime / CLI 必须实现）：
  1. 加载 Persona（只读 brief + backstory）
  2. 解析 sub_skills
  3. 自动递归安装 required 子技能（CLI 已实现伪代码）
  4. 需要时才注入具体子技能内容（Progressive Disclosure）

**示例**：`senior-fullstack-engineer` 依赖 5 个子技能，其中 4 个 required，1 个 optional。

### 7. 与 Registry / Mirror 的深度配合

- **Registry Index**（index.json）：轻量元数据摘要（我们已生成包含 3 个角色的示例）
- **CLI 配置**（~/.oarss/config.toml）：支持多 mirror、默认路径、依赖策略、缓存
- **安装命令**：
  ```bash
  oar install senior-fullstack-engineer --with-deps --mirror internal
  oar search "产品经理"
  ```
- **企业 mirror**：复制 index.json + 修改 source.url 即可实现私有化 + 审核

### 8. 已落地 3 个高质量示例（可直接复制使用）

1. **senior-fullstack-engineer**（dev 场景，5 个 sub_skills）
2. **loving-husband-role**（非 dev，验证通用性，5 个 sub_skills）
3. **product-manager-role**（跨领域，5 个 sub_skills）

每个示例都严格遵循 OARSS v1.0，已包含完整的 SKILL.md + 子技能声明。

### 9. 总结：OARSS 的设计哲学

**“以终为始”**：我们不是在造另一个技能市场，而是在为 Agent 生态建立**角色化、模块化、可镜像**的标准。  
- Persona = “人”  
- sub_skills = “专业工具箱”  
- Runtime = “大脑执行器”  

这样设计后：
- 开发者写一次子技能 → 无数角色复用
- 企业 mirror 一键部署 → 安全、合规、零延迟
- 所有 runtime 无缝兼容 → 永不被锁定

