# yunn-skills

[English](./README.md) | 中文

一组用于结构化计划和长篇小说创作的可复用 Agent Skills，适用于 Codex 以及其他兼容 [Agent Skills 规范](https://agentskills.io/specification)的客户端。

## 前置要求

- 支持 Agent Skills 的 AI 编程助手，例如 Codex
- 使用推荐安装方式时，需要 Node.js 和 `npx`

## 安装

> **提示：** 建议只安装真正需要的 Skill。每个启用的 Skill 都会在 Agent 上下文中占用一部分元数据空间。

### 快速安装（推荐）

```bash
npx skills add Yunshiro/yunn-skills
```

根据安装界面的提示，选择 `html-plan`、`novel-writer` 或同时安装两者。

### 让 Codex 安装

在 Codex 中调用内置安装器：

```text
$skill-installer 请从 https://github.com/Yunshiro/yunn-skills 安装 html-plan 和 novel-writer
```

### Codex 项目级安装

如果只想在某个项目中使用，可以将完整的 Skill 目录复制到 `<project>/.agents/skills`：

```bash
git clone https://github.com/Yunshiro/yunn-skills.git
mkdir -p /path/to/your-project/.agents/skills
cp -R yunn-skills/skills/html-plan /path/to/your-project/.agents/skills/
cp -R yunn-skills/skills/novel-writer /path/to/your-project/.agents/skills/
```

安装后的目录结构应如下：

```text
<project>/.agents/skills/html-plan/SKILL.md
<project>/.agents/skills/novel-writer/SKILL.md
```

### Codex 用户级安装

如果希望在所有项目中使用，可以将 Skill 复制到 `~/.agents/skills`：

```bash
git clone https://github.com/Yunshiro/yunn-skills.git
mkdir -p ~/.agents/skills
cp -R yunn-skills/skills/html-plan ~/.agents/skills/
cp -R yunn-skills/skills/novel-writer ~/.agents/skills/
```

Codex 通常会自动识别 Skill 变更。如果安装后没有出现，请重启 Codex。

## 使用方法

Codex 支持两种 Skill 调用方式：

1. **显式调用：** 使用 `$skill-name` 指定 Skill。
2. **自动触发：** 直接描述任务，由 Codex 根据 Skill 的描述判断是否加载。

为了获得更稳定的结果，建议显式调用 Skill，并在提示词中写清目标、背景、范围、篇幅和输出要求。

## 可用 Skills

### html-plan

为技术或项目工作生成结构清晰、可直接交付的单文件 HTML 计划文档。

```text
$html-plan 请为一个 SaaS 项目的多租户改造生成实施计划，
包含范围、里程碑、负责人、依赖、风险、回滚策略和验收标准。
```

**适用场景：**

- 实施计划和技术执行方案
- 项目计划和路线图
- 迁移与上线计划
- 发布计划
- 研究与调查计划

**主要能力：**

- 输出一个自包含的 `.html` 文件
- 使用语义化 HTML 和稳定的标题层级
- 为较长文档生成目录
- 使用表格展示时间线、负责人、依赖、风险和状态
- 内置统一的中性文档主题与打印样式
- 无需外部字体、脚本或构建工具，可直接从本地打开

该 Skill 专门用于执行型文档，不适合制作营销页面、仪表盘或复杂交互应用。

### novel-writer

一套端到端小说创作工作流，覆盖故事构思、角色设计、大纲、章节创作、润色和长篇连续性管理。

```text
$novel-writer 我想写一篇适合番茄平台的都市异能小说，
预计 8 章、2 万字。主角是一个夜班代驾，能力是能看到物品最近一次易主的画面。
```

文学模式示例：

```text
$novel-writer 写一篇用于文学征文的现实主义中篇小说，
背景是正在消失的北方矿区，第三人称限知，语言克制，约 3 万字。
```

**两种创作模式：**

| 模式 | 适用场景 | 创作重点 |
| --- | --- | --- |
| 商业网文模式 | 番茄、七猫、起点等平台 | 黄金三章、爽点节拍、冲突升级、代入感和去 AI 味检查 |
| 文学模式 | 征文、文学刊物和个人创作 | 人物弧光、价值转折、潜台词、克制表达和留白 |

**完整流程：**

1. 采集需求并判断创作模式。
2. 生成角色 DNA 档案。
3. 设计故事大纲与冲突结构。
4. 生成逐章情绪节拍表。
5. 按章节或按卷创作正文。
6. 从代入感、AI 痕迹、节奏、对话、环境和心理活动等维度分层润色。

不超过 12 章且少于 5 万字的中短篇会逐章推进；达到 13 章或 5 万字的作品会切换为按卷规划，并维护进度、记忆和伏笔记录，以支持跨会话续写。

**默认输出：**

```text
output/
├── 章节正文/
│   ├── 第1章·章节标题.md
│   └── ...
└── 创作材料/
    ├── 书名-角色DNA档案.md
    ├── 书名-故事大纲.md
    ├── 书名-情绪节拍表.md
    └── ...
```

该 Skill 会在关键节点暂停确认，方便你在继续创作前检查平台、设定、人物、大纲和节拍。

## 仓库结构

```text
yunn-skills/
├── README.md
├── README.zh.md
└── skills/
    ├── html-plan/
    │   └── SKILL.md
    └── novel-writer/
        ├── SKILL.md
        ├── agents/
        │   └── openai.yaml
        ├── references/
        │   ├── commercial-mode.md
        │   ├── literary-mode.md
        │   ├── long-form-continuity.md
        │   ├── planning.md
        │   └── writing-and-revision.md
        └── scripts/
            └── check_novel.py
```

每个 Skill 都是独立目录，可以按需单独安装。`novel-writer` 只在对应创作阶段加载所需参考文件，并通过同一个确定性脚本完成单章与全稿检查。

## 更新

如果通过 `npx skills` 安装，重新运行安装命令即可获取最新版本：

```bash
npx skills add Yunshiro/yunn-skills
```

如果采用手动安装，请先拉取仓库更新，再重新复制需要的 Skill 目录。覆盖旧版本前，请先确认是否存在本地自定义修改。

## 参考

- [OpenAI：Build skills](https://learn.chatgpt.com/docs/build-skills)
- [Agent Skills 规范](https://agentskills.io/specification)
