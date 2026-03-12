# 发布你的第一个 Agent Skill：完整实战指南

> 本指南以我们刚创建的 `ai-engineering-toolkit` 为实际案例，从零走通 Skill 发布的完整流程。

---

## 目录

1. [发布前检查](#1-发布前检查)
2. [验证 Skill 是否符合规范](#2-验证-skill-是否符合规范)
3. [本地测试与质量评估](#3-本地测试与质量评估)
4. [初始化 Git 仓库并推送到 GitHub](#4-初始化-git-仓库并推送到-github)
5. [优化 GitHub 仓库的呈现](#5-优化-github-仓库的呈现)
6. [提交到社区集合](#6-提交到社区集合)
7. [推广策略](#7-推广策略)
8. [持续维护](#8-持续维护)

---

## 1. 发布前检查

在推送到 GitHub 之前，需要确保每个 Skill 都满足 [agentskills.io](https://agentskills.io/specification) 的开放标准。以下是一份检查清单：

### 1.1 目录结构

每个 Skill 必须是一个独立文件夹，文件夹名称必须与 `SKILL.md` 中 `name` 字段完全一致。标准结构如下：

```
skill-name/
├── SKILL.md          # 必需：包含元数据和指令
├── scripts/          # 可选：可执行辅助脚本
├── references/       # 可选：按需加载的文档
├── assets/           # 可选：模板、数据文件
└── examples/         # 可选：使用示例
```

以我们的项目为例，当前结构是：

```
ai-engineering-toolkit/
├── skills/
│   ├── prompt-evaluator/
│   │   └── SKILL.md
│   ├── context-budget-planner/
│   │   └── SKILL.md
│   ├── rag-pipeline-architect/
│   │   └── SKILL.md
│   ├── agent-safety-guard/
│   │   └── SKILL.md
│   └── eval-harness-builder/
│       └── SKILL.md
├── README.md
└── LICENSE
```

### 1.2 SKILL.md Frontmatter 规范

YAML Frontmatter 是 Skill 的"身份证"，**必须严格遵守以下规则**：

| 字段 | 是否必需 | 约束条件 |
|------|:--------:|----------|
| **name** | 是 | 最多 64 字符；仅限小写字母、数字和连字符 `-`；不能以连字符开头或结尾；不能有连续连字符 `--`；**必须与父目录名一致** |
| **description** | 是 | 最多 1024 字符；非空；应明确说明"做什么"和"什么时候用" |
| **license** | 否 | 许可证名称或引用文件 |
| **compatibility** | 否 | 最多 500 字符；环境要求（OS、依赖等） |
| **metadata** | 否 | 任意键值对（如 author、version） |

**合格示例**：

```yaml
---
name: prompt-evaluator
description: >
  Evaluate and optimize LLM prompts for quality, clarity, and effectiveness.
  Use this skill when reviewing system prompts, user prompt templates, or
  any instructional text destined for an LLM.
license: MIT
metadata:
  author: your-github-username
  version: "1.0.0"
---
```

**不合格示例**（常见错误）：

```yaml
---
name: Prompt-Evaluator        # ❌ 不能有大写字母
name: -prompt-evaluator        # ❌ 不能以连字符开头
name: prompt--evaluator        # ❌ 不能有连续连字符
description: Helps with prompts.  # ❌ 太模糊，缺少"什么时候用"
---
```

### 1.3 正文最佳实践

- **篇幅**：主 `SKILL.md` 正文建议控制在 **500 行以内**，更详细的内容放到 `references/` 目录
- **Token 效率**：遵循渐进式披露（Progressive Disclosure）原则
  - 第一层：元数据（约 100 tokens）— 启动时加载所有 Skill 的 name + description
  - 第二层：指令（< 5000 tokens）— 激活时加载完整的 SKILL.md 正文
  - 第三层：资源（按需）— 需要时才加载 scripts/、references/、assets/
- **必须包含**：分步执行说明、输入输出示例、边缘情况处理
- **建议包含**：决策树、检查清单、代码模板

---

## 2. 验证 Skill 是否符合规范

Anthropic 官方提供了 `skills-ref` 验证工具，可以自动检查你的 Skill 是否符合规范。

### 2.1 安装验证工具

```bash
# 克隆 agentskills 仓库
git clone https://github.com/agentskills/agentskills.git
cd agentskills/skills-ref

# 使用 pip 安装
python -m venv .venv
source .venv/bin/activate    # macOS / Linux
pip install -e .
```

如果你使用 `uv`（推荐，更快）：

```bash
cd agentskills/skills-ref
uv sync
source .venv/bin/activate
```

### 2.2 运行验证

```bash
# 验证单个 Skill
skills-ref validate path/to/prompt-evaluator

# 验证我们项目中的所有 Skills
for skill in skills/*/; do
  echo "=== Validating $skill ==="
  skills-ref validate "$skill"
done
```

验证通过会输出无错误；如果有问题，会告诉你具体哪个字段不符合要求。

### 2.3 查看属性与生成提示词

```bash
# 以 JSON 格式查看 Skill 的解析结果
skills-ref read-properties skills/prompt-evaluator

# 生成适合嵌入 Agent 系统提示词的 XML 格式
skills-ref to-prompt skills/prompt-evaluator skills/context-budget-planner
```

生成的 XML 格式如下，可直接嵌入 Agent 的 system prompt：

```xml
<available_skills>
  <skill>
    <name>prompt-evaluator</name>
    <description>Evaluate and optimize LLM prompts...</description>
    <location>/path/to/prompt-evaluator/SKILL.md</location>
  </skill>
</available_skills>
```

---

## 3. 本地测试与质量评估

发布前务必在本地进行测试。这是确保 Skill 质量的关键步骤。

### 3.1 在 Claude Code / WorkBuddy 中测试

**方式一：项目级安装**

将 Skill 文件夹放到项目的 `.claude/skills/` 或 `.github/skills/` 目录中：

```bash
# 在你的测试项目中
mkdir -p .claude/skills
cp -r ai-engineering-toolkit/skills/prompt-evaluator .claude/skills/
```

然后启动 Claude Code，输入 `/skills` 检查是否识别到你的 Skill。

**方式二：全局安装**

放到用户目录的全局 Skills 路径下，所有项目都能使用：

```bash
cp -r ai-engineering-toolkit/skills/prompt-evaluator ~/.claude/skills/
```

### 3.2 设计测试用例

根据 [agentskills.io 官方评估指南](https://agentskills.io/skill-creation/evaluating-skills)，每个测试用例由三部分组成：

| 组成部分 | 说明 | 示例 |
|----------|------|------|
| **Prompt** | 真实的用户消息 | "请评估这段系统提示词的质量：\<prompt 内容\>" |
| **Expected Output** | 描述成功结果 | "输出包含 8 个维度的评分，总分在 0-100 之间，每项有改进建议" |
| **Input Files** | Skill 运行所需的文件（可选） | prompt_to_evaluate.txt |

建议从 **2-3 个测试用例**开始，覆盖以下维度：

```text
测试用例 1：标准场景 — 评估一段中等质量的系统提示词
测试用例 2：边缘情况 — 输入一段极短或极模糊的提示词
测试用例 3：高级功能 — 使用 A/B 对比模式评估两段提示词
```

### 3.3 运行 A/B 评估

官方推荐的评估模式是"**双轨对比**"：同一个任务分别在 **使用 Skill** 和 **不使用 Skill** 的条件下运行，然后比较结果。

```text
workspace/iteration-1/
├── eval-test-case-1/
│   ├── with_skill/       # 加载 Skill 后的输出
│   │   ├── output.md
│   │   ├── timing.json   # Token 数和耗时
│   │   └── grading.json  # 评分结果
│   └── without_skill/    # 不加载 Skill 的输出
│       ├── output.md
│       ├── timing.json
│       └── grading.json
└── benchmark.json         # 汇总统计
```

评估维度：

- **pass_rate**：断言通过率（Skill 版 vs 无 Skill 版的差异）
- **time_seconds**：执行时长
- **tokens**：消耗的 Token 数量
- **delta**：使用 Skill 带来的质量提升 vs 额外成本

### 3.4 迭代优化

每次评估后形成反馈闭环：

1. 分析失败的断言和人工反馈
2. 修改 `SKILL.md` 的指令（泛化修复，不要针对特定测试打补丁）
3. 在新的 `iteration-<N+1>/` 目录中重跑所有测试
4. 重复直到通过率达到预期

---

## 4. 初始化 Git 仓库并推送到 GitHub

验证和测试通过后，就可以发布到 GitHub 了。

### 4.1 初始化 Git 仓库

```bash
cd ai-engineering-toolkit

# 创建 .gitignore
cat > .gitignore << 'EOF'
# Python
__pycache__/
*.py[cod]
.venv/

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/

# 评估临时文件
iteration-*/
EOF

# 初始化仓库
git init
git add .
git commit -m "feat: initial release of AI Engineering Toolkit with 5 production-ready skills"
```

### 4.2 创建 GitHub 仓库

**方式一：通过 GitHub CLI（推荐）**

```bash
# 如果没装 gh，先安装：brew install gh
gh auth login
gh repo create ai-engineering-toolkit --public --description "Production-ready AI Engineering Skills: prompt evaluation, context budget planning, RAG pipeline design, agent safety, and eval harness building" --source=. --push
```

**方式二：通过 Web 界面**

1. 打开 [github.com/new](https://github.com/new)
2. 仓库名：`ai-engineering-toolkit`
3. 描述：`Production-ready AI Engineering Skills for Claude Code, Cursor, Copilot, and any agent that supports the Agent Skills standard.`
4. 选择 **Public**
5. **不要** 勾选"Add a README"（我们已有）
6. 创建后按页面提示推送：

```bash
git remote add origin https://github.com/YOUR_USERNAME/ai-engineering-toolkit.git
git branch -M main
git push -u origin main
```

### 4.3 添加 Topic 标签

这一步非常关键，直接影响你的 Skill 被发现的概率。在 GitHub 仓库页面点击 **About** 旁边的齿轮图标，添加以下 Topics：

```
agent-skills
claude-code-skills
claude-skills
ai-engineering
context-engineering
prompt-engineering
rag
llm-evaluation
agent-safety
agentskills
```

---

## 5. 优化 GitHub 仓库的呈现

好的 README 和仓库结构能显著提升 Stars 增速。

### 5.1 README 必备要素

| 模块 | 说明 | 重要程度 |
|------|------|:--------:|
| **标题 + Badge** | 项目名、Stars Badge、License Badge | 必须 |
| **一句话描述** | 一行说清楚项目做什么 | 必须 |
| **安装方式** | 至少提供 2 种安装方法 | 必须 |
| **Skills 列表** | 每个 Skill 的名称、一句话功能、亮点 | 必须 |
| **快速开始** | 一个最简单的使用示例 | 必须 |
| **详细文档** | 每个 Skill 的详细用法和示例 | 推荐 |
| **贡献指南** | 如何参与贡献 | 推荐 |
| **许可证** | 开源许可声明 | 必须 |

### 5.2 添加 Badge

在 README 顶部添加 Shields.io Badge，让仓库看起来更专业：

```markdown
![GitHub Stars](https://img.shields.io/github/stars/YOUR_USERNAME/ai-engineering-toolkit?style=social)
![License](https://img.shields.io/github/license/YOUR_USERNAME/ai-engineering-toolkit)
![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-blue)
![Claude Code](https://img.shields.io/badge/Claude%20Code-ready-green)
```

### 5.3 创建 Release

打上版本标签并创建 Release，让用户能看到变更历史：

```bash
git tag -a v1.0.0 -m "🎉 Initial release: 5 production-ready AI engineering skills"
git push origin v1.0.0
```

然后在 GitHub 仓库页面，进入 **Releases** → **Draft a new release**，选择 `v1.0.0` 标签，填写 Release Notes：

```markdown
## AI Engineering Toolkit v1.0.0

### Included Skills

- **prompt-evaluator** — 8-dimension scoring system for LLM prompt quality
- **context-budget-planner** — Token budget planning with 5-zone model
- **rag-pipeline-architect** — End-to-end RAG pipeline design
- **agent-safety-guard** — 5-layer defense + 65-point red team checklist
- **eval-harness-builder** — 3-tier evaluation framework with CI/CD templates

### Compatibility

Compatible with Claude Code, Cursor, GitHub Copilot, Windsurf,
and any agent supporting the Agent Skills open standard.
```

---

## 6. 提交到社区集合

在 GitHub 上发布只是第一步，要获得更多关注需要提交到社区集合。

### 6.1 提交到 awesome-agent-skills（最大的集合）

[VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) 是目前最大的 Skills 集合。

**重要前提**：该集合有一条硬性要求 —— **刚创建的全新技能不会被接受。请等待技能成熟并积累了用户和 Stars 后再提交。** 建议至少等到获得 **10-20+ Stars** 后再提交 PR。

提交步骤：

1. **Fork 仓库**

```bash
gh repo fork VoltAgent/awesome-agent-skills --clone
cd awesome-agent-skills
```

2. **创建分支**

```bash
git checkout -b add-ai-engineering-toolkit
```

3. **在 README.md 中添加条目**

找到 "Community Skills" → "AI and Data" 子分类（或 "Context Engineering"），在末尾添加：

```markdown
- **[YOUR_USERNAME/ai-engineering-toolkit](https://github.com/YOUR_USERNAME/ai-engineering-toolkit)** - Production-ready AI engineering skills for prompt evaluation, RAG design, and agent safety
```

注意格式要求：
- 描述必须简短，**10 个单词以内**
- 名称中必须包含作者/组织前缀
- 必须是公开仓库
- 必须有文档（README 或 SKILL.md）

4. **提交 PR**

```bash
git add README.md
git commit -m "Add skill: YOUR_USERNAME/ai-engineering-toolkit"
git push origin add-ai-engineering-toolkit
gh pr create --title "Add skill: YOUR_USERNAME/ai-engineering-toolkit" --body "Adding 5 production-ready AI engineering skills covering prompt evaluation, context budget planning, RAG pipeline design, agent safety, and eval harness building."
```

### 6.2 提交到其他集合

| 集合 | 地址 | 特点 |
|------|------|------|
| **libukai/awesome-agent-skills** | [GitHub](https://github.com/libukai/awesome-agent-skills) | 中文社区主要集合，注重教程和实战 |
| **JackyST0/awesome-agent-skills** | [GitHub](https://github.com/JackyST0/awesome-agent-skills) | 面向 Cursor、Claude Code、Copilot 等多平台 |
| **anthropics/skills** | [GitHub](https://github.com/anthropics/skills) | Anthropic 官方仓库（接受社区贡献，但门槛较高） |

### 6.3 在 Claude 生态内发布

根据 [Claude 官方 Skills 博客](https://claude.com/blog/skills)，还可以通过以下方式让你的 Skill 被更多人使用：

**通过 Claude 插件市场**（如果支持）：

```bash
# 在 Claude Code 中执行
/plugin marketplace add YOUR_USERNAME/ai-engineering-toolkit
```

**通过 Claude Developer Platform API**：

如果你的 Skill 需要代码执行能力，可以通过 `/v1/skills` 端点进行版本管理和分发，面向 API 用户。

---

## 7. 推广策略

发布到 GitHub 和社区集合之后，还需要主动推广来获取初始关注度。

### 7.1 内容推广（中文社区）

| 平台 | 内容形式 | 建议标题 |
|------|----------|----------|
| **掘金** | 技术文章 | 《我用 Agent Skills 开源了一套 AI 工程化工具包》 |
| **知乎** | 回答/专栏 | 在"Claude Code"、"AI 工程"相关问题下回答 |
| **CSDN** | 博客 | 《Agent Skills 实战：5 个 AI 工程化技能的设计与发布全过程》 |
| **微信公众号** | 长文 | 面向你的目标读者群体定制内容 |
| **小红书** | 图文 | 轻量化介绍 + 使用效果截图 |

### 7.2 内容推广（英文社区）

| 平台 | 建议 |
|------|------|
| **X (Twitter)** | @anthropic, @alexalbert__ 并使用 #AgentSkills #ClaudeCode 标签 |
| **Reddit** | r/ClaudeAI, r/LocalLLaMA, r/MachineLearning 发帖 |
| **Hacker News** | Show HN 帖子 |
| **Dev.to** | 英文版技术博客 |

### 7.3 冷启动技巧

以下技巧可以帮助快速获得第一批 Stars：

1. **先 Star 别人的项目，参与讨论**，建立社区关系后再推广自己的项目
2. **在相关 Issue 中提供帮助**，自然引流到自己的 Skill
3. **写一篇"我是怎么做的"教程**，比单纯推广项目本身更容易获得关注
4. **找几个朋友先试用，收集反馈**，用真实用户的正面评价作为推广素材
5. **给你的 README 加上 GIF 演示图**，直观展示 Skill 的效果

---

## 8. 持续维护

发布不是终点。持续维护是积累信任和 Stars 的关键。

### 8.1 版本管理

使用语义化版本（Semantic Versioning）：

| 版本变更 | 格式 | 场景 |
|----------|------|------|
| 修复问题 | v1.0.1 | 修正错别字、修复边缘情况 |
| 新增功能 | v1.1.0 | 添加新的评估维度、新的工作流 |
| 重大变更 | v2.0.0 | 重构 Skill 结构、改变核心流程 |

### 8.2 Issue 和 PR 管理

- 及时回复 Issue（24 小时内首次回复为佳）
- 使用 Label 分类：`bug`、`enhancement`、`question`、`good first issue`
- 鼓励社区贡献：标记 `good first issue` 和 `help wanted`

### 8.3 持续改进循环

```
收集用户反馈 → 设计测试用例 → 运行 A/B 评估 → 优化 SKILL.md → 发布新版本
```

每次大版本更新都写一篇博客或帖子，保持项目的活跃度和曝光度。

---

## 快速回顾：发布检查清单

下面是一份 **一次性检查清单**，在你点击"Push"之前过一遍：

- [ ] 每个 Skill 的 `name` 字段是否全小写、用连字符分隔、与目录名一致
- [ ] 每个 `description` 是否清楚说明了"做什么"和"什么时候用"
- [ ] `SKILL.md` 正文是否包含分步说明、示例、边缘情况处理
- [ ] 正文是否控制在 500 行以内
- [ ] 运行 `skills-ref validate` 是否全部通过
- [ ] 在 Claude Code 中本地测试是否正常工作
- [ ] README.md 是否包含安装方式、Skills 列表、快速开始
- [ ] LICENSE 文件是否存在
- [ ] .gitignore 是否配置好
- [ ] GitHub Topics 是否添加了 `agent-skills`、`claude-code-skills` 等标签
- [ ] 是否创建了 Release 并打了版本标签

---

> **下一步行动**：如果你已经准备好了，我可以直接帮你在当前项目中执行 `git init`、创建 `.gitignore`，并完成第一次 commit。只需要告诉我你的 GitHub 用户名，就可以一键推送。
