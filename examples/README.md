#### 推荐文件名：`install-examples.sh`

```bash
#!/usr/bin/env bash
# install-examples.sh
# 一键将 OARSS 示例角色包安装到本地 agent-skills 目录
# 支持常见 agent runtime（如 Claude Code、Cursor、OpenClaw 等）的 SKILL.md 加载路径
# 2026-03 使用说明：chmod +x install-examples.sh && ./install-examples.sh

set -euo pipefail

# ===================== 配置区域 =====================
REPO_URL="https://github.com/open-agent-role-skill/oarss-examples.git"
TARGET_DIR="${HOME}/.agent-skills"           # 最常用路径，可修改
# 其他常见路径备选（根据 runtime 不同）
# TARGET_DIR="${HOME}/.cursor/skills"
# TARGET_DIR="${HOME}/.claude-code/skills"
# TARGET_DIR="${HOME}/.openclaw/skills"

# ===================== 逻辑 =====================
echo "OARSS 示例角色包安装脚本 v1.0"
echo "目标目录：${TARGET_DIR}"
echo ""

if [ ! -d "$TARGET_DIR" ]; then
    echo "创建目标目录：$TARGET_DIR"
    mkdir -p "$TARGET_DIR"
fi

# 检查 git 是否已克隆过
if [ -d "${TARGET_DIR}/oarss-examples" ]; then
    echo "检测到已有克隆，正在更新..."
    cd "${TARGET_DIR}/oarss-examples"
    git pull --ff-only || {
        echo "更新失败，尝试重新克隆..."
        cd ..
        rm -rf oarss-examples
        git clone "$REPO_URL"
    }
else
    echo "克隆示例仓库..."
    git clone "$REPO_URL" "${TARGET_DIR}/oarss-examples"
fi

# 创建符号链接（推荐方式，便于后续更新）
echo "创建符号链接到示例角色..."
cd "$TARGET_DIR"

for role in senior-fullstack-engineer loving-husband-role product-manager-role; do
    if [ -d "oarss-examples/$role" ]; then
        ln -sf "oarss-examples/$role" "$role" 2>/dev/null || true
        echo "  已链接：$role"
    else
        echo "  警告：未找到 $role（可能仓库尚未包含）"
    fi
done

echo ""
echo "安装完成！"
echo ""
echo "支持的常见路径总结："
echo "  • 通用推荐：          ~/.agent-skills/"
echo "  • Cursor / Claude Code：~/.cursor/skills/ 或 ~/.claude-code/skills/"
echo "  • OpenClaw 等：        ~/.openclaw/skills/ 或自定义路径"
echo ""
echo "如果你的 agent runtime 使用其他路径，请手动复制或符号链接："
echo "  cp -r ${TARGET_DIR}/oarss-examples/* \$YOUR_SKILLS_DIR/"
echo ""
echo "验证方式：重新启动你的 agent IDE / runtime，应能自动识别这些 SKILL.md"
echo "有问题？欢迎 issue：https://github.com/open-agent-role-skill/oarss-examples"
```

#### README.md 中的对应说明片段（推荐放在 examples 仓库 README 开头之后）

```markdown
## 快速安装示例角色包

目前大多数 agent runtime 都会扫描特定目录下的 SKILL.md 文件夹来加载技能/角色。

### 方法一：一键安装脚本（推荐）

```bash
git clone https://github.com/open-agent-role-skill/oarss-examples.git
cd oarss-examples
chmod +x install-examples.sh
./install-examples.sh
```

脚本会默认安装到 `~/.agent-skills/`，并为每个角色创建符号链接，便于后续 git pull 更新。

### 方法二：手动放置（适用于自定义路径）

1. Clone 示例仓库：
   ```bash
   git clone https://github.com/open-agent-role-skill/oarss-examples.git
   ```

2. 复制或符号链接到你的 agent runtime 支持的 skills 目录：
   - 通用推荐（未来 CLI 会标准化）：`~/.agent-skills/`
   - Cursor 示例：`~/.cursor/skills/`
   - Claude Code 示例：`~/.claude-code/skills/`
   - OpenClaw 示例：`~/.openclaw/skills/`

   示例命令（符号链接方式，推荐）：
   ```bash
   ln -s ~/path/to/oarss-examples/senior-fullstack-engineer ~/.agent-skills/senior-fullstack-engineer
   ```

3. 重启你的 agent IDE 或 runtime，即可使用。

### 验证是否生效

- 在 agent 中输入类似：
  - “作为资深全栈工程师帮我 review 这段 React 代码”
  - “我今天和老婆吵架了，怎么办？”
  - “我们新 AI 功能的 PRD 应该怎么写？”

如果角色被正确激活（语气、结构、引用最佳实践等明显变化），说明安装成功。

后续我们会推出 `oar` CLI 来标准化安装、更新、搜索与依赖管理。
