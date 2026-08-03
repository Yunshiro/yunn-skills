---
name: novel-writer
description: Create, continue, revise, or audit Chinese fiction through a staged workflow covering requirements, character DNA, outlining, emotional beats, chapter drafting, deterministic manuscript checks, layered revision, and long-form continuity. Use when the user asks to write a novel or story, develop a commercial web-fiction project for platforms such as Fanqie, Qimao, or Qidian, create literary fiction or contest submissions, continue an existing manuscript, remove AI-like prose patterns, or inspect novel chapters for immersion and pacing.
---

# 小说创作工坊

将小说创作拆成可确认、可恢复、可验证的六个阶段。不要一次性跳过规划直接生成全稿，除非用户明确要求跳过确认。

## 核心优先级

按以下顺序解决冲突：

1. 保证人物行为、故事因果和设定一致。
2. 保证视角贴身和读者代入感。
3. 移除模具句、机械排比与编排痕迹。
4. 调整节奏、字数和格式。

商业网文中，代入感高于去 AI 味。不要为了减少心理活动而删掉主角的即时感知、身体反应、危险判断和明确欲望。

## 资源路由

只读取当前阶段需要的参考文件：

| 当前任务 | 必须读取 |
| --- | --- |
| 需求采集、角色 DNA、大纲准备、情绪节拍 | [references/planning.md](references/planning.md) |
| 商业网文的平台结构、黄金三章、爽点、冲突升级、代入感 | [references/commercial-mode.md](references/commercial-mode.md) |
| 纯文学、征文、自留或文学模式 | [references/literary-mode.md](references/literary-mode.md) |
| 正文创作、单章检查、全稿体检或分层润色 | [references/writing-and-revision.md](references/writing-and-revision.md) |
| 达到 13 章或 5 万字、按卷创作、跨会话续写 | [references/long-form-continuity.md](references/long-form-continuity.md) |

解析本 Skill 所在目录为 `<skill-dir>`。运行检查时调用 `<skill-dir>/scripts/check_novel.py`，不要把脚本重新写进提示词或项目目录。

## 模式判断

阶段一必须确认平台或用途，再决定模式。两套结构规则不可混用。

| 用户目标 | 模式 | 结构重点 |
| --- | --- | --- |
| 番茄、七猫、得间、疯读等免费阅读平台 | 商业网文 | 黄金三章、密集反馈、锯齿上行情绪 |
| 起点、掌阅、QQ 阅读等付费订阅平台 | 商业网文 | 同样执行商业结构，允许略慢铺垫 |
| 纯文学、征文、文学刊物、个人自留 | 文学 | 人物选择、价值转折、潜台词与余韵 |

用户无法判断时，说明假设并默认商业网文模式。

篇幅判断：

- 不超过 12 章且少于 5 万字：中短篇，逐章确认后创作。
- 达到 13 章或 5 万字：长篇，按每卷 3 至 5 章规划；卷前确认一次，卷内连续创作。

## 输出目录

所有创作文件保存到当前项目的 `output` 目录。需要时创建目录：

```text
output/
├── 章节正文/
│   ├── 第1章·章节标题.md
│   └── ...
└── 创作材料/
    ├── 书名-角色DNA档案.md
    ├── 书名-故事大纲.md
    ├── 书名-情绪节拍表.md
    ├── 书名-创作进度汇总.md    # 长篇
    └── ...
```

正文文件除章节标题外不使用 Markdown 语法。角色、大纲、节拍和进度文档可以使用 Markdown 表格与列表。

## 六阶段工作流

### 阶段一：需求采集

读取 `references/planning.md`。只询问用户尚未提供的信息，至少明确：

- 平台或用途、类型、故事内核。
- 商业网文的金手指机制、限制和代价。
- 背景、视角、人物数量和参考作品。
- 总字数、章节数和单章目标字数。

输出《故事规划确认表》，注明模式、篇幅模式、已知事实、默认假设和待确认项。用户确认后进入阶段二。

### 阶段二：角色 DNA

继续使用 `references/planning.md`。为主要人物建立能从过去经历推导到恐惧、渴望和行为的档案。

商业网文额外指定：

- 反派的轻蔑表达习惯。
- 配角的明确反馈功能。
- 1 至 2 名具名情绪代理人。
- 主角固定的身体感知清单和内心独白语气。

保存角色 DNA，向用户展示摘要并确认后进入阶段三。

### 阶段三：故事大纲

先读取 `references/planning.md`，再按模式读取：

- 商业网文：`references/commercial-mode.md`。
- 文学模式：`references/literary-mode.md`。

商业网文不得使用“激励事件可以拖到全书四分之一处”的慢结构替代黄金三章。文学模式不得硬塞每 2000 字爽点或当面打脸。

保存故事大纲，完成对应模式的自检。用户确认后进入阶段四。

### 阶段四：情绪节拍

读取 `references/planning.md` 的节拍规则。为每章写出事件目标、情绪变化、价值转折、代价、信息变化、伏笔和具体章末事件。

长篇先确认卷级节拍，再只展开当前卷。保存节拍表并等待确认。

### 阶段五：正文创作

读取 `references/writing-and-revision.md`，并保持对应模式参考文件在上下文中。长篇还要读取 `references/long-form-continuity.md` 和当前进度文件。

每章按以下顺序完成：

1. 读取角色 DNA、大纲和当前章节拍。
2. 按目标字数反推场景拍子，一稿写入目标上下 200 字。
3. 保存到 `output/章节正文/第N章·[标题].md`。
4. 运行单章检查：

```bash
python3 <skill-dir>/scripts/check_novel.py chapter \
  "./output/章节正文/第N章·标题.md" \
  --target 2500 \
  --hero "主角名"
```

5. 修复脚本报告的问题并重新运行，直到机械指标通过。
6. 人工通读视角、因果、人物利益、数字常识、对话声音和动作复用。
7. 中短篇向用户报告本章摘要、字数、检查结果和下一章目标，等待确认。
8. 长篇更新进度、人物状态、能力、伏笔和精确续写入口；卷内继续下一章。

将示例命令中的目标字数和主角名替换为当前项目值。脚本非零退出表示仍有机械指标未通过，不要忽略。

### 阶段六：分层润色

读取 `references/writing-and-revision.md`，严格按顺序执行：

1. 代入感润色：补视角感知、即时情绪、身体感官和人味。
2. 去 AI 味润色：删模具、重复解释、机械排比和编排痕迹。
3. 节奏压缩：删除无功能段落和不改变局势的对话。
4. 对话优化。
5. 环境与心理活动优化。

商业网文必须执行前三层；其余层根据章节重点执行。文学模式也执行去 AI 味检查，但不强制商业爽点与章末悬念。

全稿完成后运行：

```bash
python3 <skill-dir>/scripts/check_novel.py manuscript \
  "./output/章节正文" \
  --hero "主角名"
```

所有章节目标相同时添加 `--target 2500`。脚本通过后仍执行参考文件中的人工终检。

## 长篇续写

长篇模式读取 `references/long-form-continuity.md`。每章立即更新进度，不等卷末凭记忆补写。

新会话恢复时依次读取快速记忆、进度汇总、当前卷节拍、相关角色 DNA 和上一章全文。先向用户报告上一章停点、当前状态和下一章目标，再继续写作。不要重新规划已经确认的内容。

## 完成标准

交付前确认：

- 所有必要确认节点均已通过，或用户明确要求跳过。
- 角色、大纲、节拍和正文文件路径符合规范。
- 每章机械检查通过，且完成无法脚本化的人工通读。
- 伏笔、人物状态、能力限制、时间、距离和数量前后一致。
- 长篇的进度与续写入口已经更新到最后完成的章节。
- 最终报告列出已完成文件、总字数、检查结果和仍需用户决定的问题。
