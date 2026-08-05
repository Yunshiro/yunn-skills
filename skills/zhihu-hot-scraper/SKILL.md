---
name: zhihu-hot-scraper
description: 抓取知乎热榜（https://www.zhihu.com/hot）话题及其高赞回答/评论，输出为按话题分文件的 Markdown 文档集合。当用户提出抓取知乎热榜、知乎热门话题评论、知乎榜单回答等内容时使用。执行前需要用户指定抓取话题数量和每个话题的回答数量；依赖 playwright MCP（mcp__playwright__* 工具），未安装时先引导用户安装。输出形式为一个"知乎热榜_日期"文件夹，内含 00_索引.md 和每个话题一个 md 文件，评论内容保持完整原文不截断。
agent_created: true
---

# 知乎热榜抓取 Skill（zhihu-hot-scraper）

抓取知乎热榜前 N 个话题，每个话题取前 M 条高赞回答（评论）的**完整原文**，输出为按话题分文件的 Markdown 文档集合。

## 执行前置：询问参数

执行前必须向用户确认两个参数（若用户已直接给出则可跳过）：

1. **抓取前几个话题**（N，默认 10）
2. **每个话题抓取前几条评论/回答**（M，默认 5）

示例提问：
> 好的，开始抓取知乎热榜。请问：
> 1. 要抓取热榜前几个话题？（默认 10）
> 2. 每个话题抓取前几条高赞回答？（默认 5）

## 第一步：检测 playwright MCP 是否可用（必须）

用 ToolSearch 检测 playwright 浏览器自动化工具是否已安装可用：

- 调用 `ToolSearch`，`queries: ["playwright browser automation", "browser"]` 或 `tool_names` 查找 `mcp__playwright__browser_snapshot`、`mcp__playwright__browser_run_code_unsafe`、`mcp__playwright__browser_navigate` 等工具。

**结果处理：**

- ✅ 找到 `mcp__playwright__*` 工具 → 继续执行抓取流程。
- ❌ 未找到 → **停止抓取**，告知用户：

> ⚠️ 未检测到 playwright MCP，无法执行抓取。
> 请先安装并启用 playwright MCP：
> 1. 在 WorkBuddy 的 MCP 配置（`~/.workbuddy/mcp.json`）中加入 playwright 服务器配置（官方推荐 `npx @playwright/mcp@latest`，配置 `headless: false` 以便使用已登录的浏览器会话）。
> 2. 保存后在连接器管理页信任/启用该 MCP。
> 3. 安装完成后重新发起抓取请求即可。

## 第二步：抓取热榜话题列表

1. 检查浏览器标签页状态（`browser_tabs`，action=list），若已有知乎热榜页（https://www.zhihu.com/hot）则选中该标签页（action=select, index=对应序号）；否则用 `browser_navigate` 打开 https://www.zhihu.com/hot 。
2. 用 `browser_snapshot` 获取页面结构，解析出热榜话题列表（`HotItem` 容器），按排名顺序取前 N 条，记录：**排名、标题、热度值、问题链接**（`/question/{id}` 格式）。注意结果可能很大，可考虑用 `browser_run_code_unsafe` 直接提取更紧凑的数据。

建议直接用 `browser_run_code_unsafe` 提取热榜数据（更省 token）：

```js
async (page) => {
  await page.waitForTimeout(2000);
  return await page.evaluate(() => {
    const items = [];
    document.querySelectorAll('.HotItem').forEach((el) => {
      const a = el.querySelector('.HotItem-title, h2 a, a');
      const hot = el.querySelector('.HotItem-hot, .HotItem-rank-hot');
      const href = el.querySelector('a')?.href || '';
      items.push({ title: a ? a.textContent.trim() : '', hot: hot ? hot.textContent.trim() : '', url: href });
    });
    return JSON.stringify(items);
  });
}
```

注意：知乎热榜是登录后可见完整榜单，未登录可能只显示部分内容。若用户浏览器已打开并登录知乎（有私信/通知提示），直接复用该会话。

## 第三步：逐个话题抓取前 M 条回答（完整原文，不截断）

对每个话题（共 N 个），用 `browser_run_code_unsafe` 打开问题页并提取前 M 条回答的**完整原文**：

```js
async (page) => {
  // 注意：将 questionUrl 替换为目标话题链接，limit 替换为 M
  const q = { url: 'https://www.zhihu.com/question/XXXXXXXX', limit: 5 };
  await page.goto(q.url, { waitUntil: 'domcontentloaded', timeout: 60000 });
  await page.waitForTimeout(2500);
  // 滚动触发懒加载
  await page.evaluate(async () => { for (let i=0;i<12;i++){ window.scrollBy(0,700); await new Promise(r=>setTimeout(r,350)); } });
  await page.waitForTimeout(1500);
  const data = await page.evaluate((lim) => {
    const items = [];
    document.querySelectorAll('.List-item').forEach((el, i) => {
      if (i >= lim) return;
      // 作者名：从 meta itemprop="name" 读取（.AuthorInfo-name 可能取不到）
      const meta = el.querySelector('.AuthorInfo meta[itemprop="name"]');
      const author = meta ? meta.content : ((el.querySelector('.AuthorInfo')?.textContent || '').trim().slice(0, 30) || '匿名用户');
      // 赞同数：.VoteButton--up 文本，含零宽字符需清理
      const voteEl = el.querySelector('.VoteButton--up, .VoteButton');
      const vote = voteEl ? voteEl.textContent.replace(/[\s\u200b]/g, '') : '';
      // 完整原文：.RichContent-inner innerText，不截断！
      const content = (el.querySelector('.RichContent-inner')?.innerText || '').trim();
      if (content) items.push({ author, vote, content });
    });
    return items;
  }, q.limit);
  return JSON.stringify({ title: document.title, url: q.url, items: data });
}
```

**关键经验（务必遵守）：**

- **作者名**：知乎新版页面 `.AuthorInfo-name` 选择器取不到作者，必须从 `.AuthorInfo meta[itemprop="name"]` 的 `content` 属性读取。
- **赞同数**：`.VoteButton--up` 文本中可能含零宽字符，用 `replace(/[\s\u200b]/g, '')` 清理后再记录。
- **内容必须完整**：取 `.RichContent-inner` 的 `innerText`，**严禁截断**（用户明确要求完整原文）。
- **懒加载**：知乎问题页回答为懒加载，必须滚动页面（12 次 scrollBy 700px + 间隔 350ms）后再提取，否则只能拿到前 1-2 条。
- **无法直接写文件**：`browser_run_code_unsafe` 的沙箱内 `require()` 和动态 `import()` 均不可用，不能直接写本地文件。正确做法是：每次脚本返回 JSON 数据 → 在本地用 Write 工具落盘。
- 为控制单次输出大小，建议**每个话题单独调用一次**脚本（一次返回完整 JSON），不要在一条脚本里循环全部话题（完整原文量大，容易超限）。

## 第四步：写入文件（一个话题一个文件）

在工作目录下创建文件夹：`知乎热榜_YYYY-MM-DD/`（YYYY-MM-DD 为当天日期）。

文件夹内文件结构：

```
知乎热榜_YYYY-MM-DD/
├── 00_索引.md                  ← 总览：抓取时间、话题列表（排名/标题/热度/文件链接）
├── 01_话题简短标题.md
├── 02_话题简短标题.md
├── ...
└── NN_话题简短标题.md
```

**每个话题文件的内容格式：**

```markdown
# 话题 NN：{完整话题标题}

- **热度**：{热度值}
- **链接**：{问题链接}
- **抓取时间**：{YYYY-MM-DD HH:MM}

---

## 评论 1｜{作者名}（赞同 {数字}）

{完整原文，一字不改，保留原有段落换行}

---

## 评论 2｜{作者名}（赞同 {数字}）

{完整原文}
```

**00_索引.md 格式：**

```markdown
# 知乎热榜 TOP{NN} 完整评论合集（{日期}）

> **抓取时间**：...
> **数据来源**：知乎热榜（https://www.zhihu.com/hot）

## 话题列表

| # | 话题 | 热度 | 文件 |
|---|------|------|------|
| 1 | [标题](链接) | 热度 | [01_xxx.md](01_xxx.md) |
```

- 话题文件命名规则：`序号_话题核心关键词.md`（如 `01_光收发模块禁令.md`），文件名简短、避免超长。
- 注意：旧版文件若存在（如同一日期重复抓取），可先询问用户是否覆盖，或使用带时间戳的文件夹名区分。

## 第五步：收尾

- 用 `present_files` 展示输出：第一个传 `00_索引.md`，随后依次传入所有话题文件（一个调用传全部路径）。
- 用简洁文字告知用户抓取完成、文件位置、以及"数据为抓取时刻实时值"等注意事项。

## 常见问题

- **热榜列表解析不到**：可能是未登录/页面未加载完成，先等待 2-3 秒再提取；必要时提示用户先在浏览器登录知乎。
- **回答只有 1-2 条**：懒加载未触发，增加滚动次数和等待时间。
- **作者全是"匿名用户"**：meta 选择器失效，改回检查 `.AuthorInfo` 的 textContent 或 `.AuthorInfo-name` 后代元素。
- **MCP 输出被截断**：一次只抓一个话题；若单个话题内容仍超限，可把 limit 临时调小分两批抓。
