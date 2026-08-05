---
name: weibo-hot-scraper
description: 抓取微博热搜榜（https://s.weibo.com/top/summary?cate=realtimehot）前 N 条热搜及每条热搜下前 M 篇帖子的完整原文与配图，输出为按热搜分文件的 Markdown 文档集合 + 本地图片。当用户提出抓取微博热搜、微博热门话题帖子、微博榜单内容时使用。依赖 playwright MCP（mcp__playwright__* 工具）与本地 Node.js。输出形式为「微博热搜_日期」文件夹，内含 00_索引.md、每条热搜一个 md 文件、images/ 图片目录。
agent_created: true
---

# 微博热搜抓取 Skill（weibo-hot-scraper）

抓取微博实时热搜榜前 N 条，每条取搜索结果前 M 篇帖子的**完整原文 + 配图**，落盘为 Markdown 文档集合。

## 执行前置：确认参数

1. **抓取前几条热搜**（N，默认 10）
2. **每条热搜抓取前几篇帖子**（M，默认 20）

用户已明确给出则跳过询问。

## 第一步：检测 playwright MCP

用 `ToolSearch`（`tool_names: ["mcp__playwright__browser_navigate", "mcp__playwright__browser_run_code_unsafe", "mcp__playwright__browser_tabs"]`）确认可用。
未安装则停止，引导用户在 `~/.workbuddy/mcp.json` 配置 `npx @playwright/mcp@latest` 并在连接器管理页信任。

## 第二步：抓取热搜榜列表

先 `browser_tabs` (action=list) 看是否已有热搜页；否则 `browser_navigate` 到 `https://s.weibo.com/top/summary?cate=realtimehot`。

用 `browser_run_code_unsafe` 提取：

```js
async (page) => {
  await page.waitForTimeout(1500);
  return await page.evaluate(() => {
    const rows = [];
    document.querySelectorAll('#pl_top_realtimehot table tbody tr').forEach((tr) => {
      const rankEl = tr.querySelector('td.td-01');
      const a = tr.querySelector('td.td-02 a');
      const hot = tr.querySelector('td.td-02 span');
      if (!a) return;
      rows.push({
        rank: rankEl ? rankEl.textContent.trim() : '',
        title: a.textContent.trim(),
        href: a.getAttribute('href') || '',
        hot: hot ? hot.textContent.trim() : '',
        tag: (tr.querySelector('td.td-03 i')?.textContent || '').trim()
      });
    });
    return JSON.stringify({ count: rows.length, rows: rows.slice(0, 15) });
  });
}
```

**过滤规则（重要）**：
- 首行是**置顶广告/要闻**，`rank` 为空 → 剔除。
- 中间夹杂的**商业推广**行 `rank` 为 `•`、`href` 为 `javascript:void(0);` → 剔除。
- 只保留 `rank` 为纯数字的行，取前 N 条。
- 热度字段可能带前缀（如 `综艺 366450`、`剧集 357193`），格式化时保留前缀。

## 第三步：关键技巧——本地 HTTP 接收服务落盘（务必使用）

`browser_run_code_unsafe` 沙箱内 **`require` / `process` / 动态 `import` 全部不可用**，无法直接写文件。
若让脚本 `return` 完整数据，200 篇帖子会瞬间撑爆上下文。

**正确做法**：起一个本地 Node HTTP 接收服务，脚本内用 `page.request.post()` 把 JSON 直接 POST 到本地落盘，脚本只 return 一行统计日志。

`page.request` 是 Playwright 服务进程的 APIRequestContext，**不受 CORS 和混合内容限制**，比在页面里 `fetch` 更可靠。

接收服务（写到系统临时目录，后台运行）：

```js
const http = require('http'); const fs = require('fs'); const path = require('path');
const DATA_DIR = '<TEMP>/weibo_hot_<DATE>/data';
fs.mkdirSync(DATA_DIR, { recursive: true });
http.createServer((req, res) => {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, GET, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', '*');
  if (req.method === 'OPTIONS') { res.writeHead(204); res.end(); return; }
  if (req.url === '/ping') { res.writeHead(200); res.end('pong'); return; }
  if (req.method === 'POST' && req.url.startsWith('/save/')) {
    const name = decodeURIComponent(req.url.slice(6)).replace(/[^A-Za-z0-9_\-]/g, '');
    const chunks = []; req.on('data', c => chunks.push(c));
    req.on('end', () => { const buf = Buffer.concat(chunks);
      fs.writeFileSync(path.join(DATA_DIR, name + '.json'), buf);
      res.writeHead(200); res.end('saved ' + buf.length); });
    return;
  }
  res.writeHead(404); res.end('nope');
}).listen(8799, '127.0.0.1', () => console.log('listening'));
```

用 Bash `run_in_background: true` 启动。

## 第四步：逐话题抓取帖子

每次调用处理 **3-4 个话题**（更多容易超时），脚本内循环 + POST 落盘 + 只 return 统计行。

```js
async (page) => {
  const TOPICS = [ {id:'01', rank:1, title:'...', hot:'...', href:'/weibo?q=...'} /* 3-4 个 */ ];
  const LIMIT = 20;
  const extract = () => page.evaluate(() => {
    const out = [];
    document.querySelectorAll('#pl_feedlist_index div.card-wrap[action-type="feed_list_item"]').forEach(card => {
      const feed = card.querySelector('.card-feed'); if (!feed) return;
      const aName = feed.querySelector('.content .info a[nick-name]');
      const author = aName ? (aName.getAttribute('nick-name') || aName.textContent.trim()) : '';
      let authorUrl = aName ? (aName.getAttribute('href') || '') : ''; if (authorUrl.startsWith('//')) authorUrl = 'https:' + authorUrl;
      const fromAs = feed.querySelectorAll('.content .from a');
      const time = fromAs[0] ? fromAs[0].textContent.trim() : '';
      let postUrl = fromAs[0] ? (fromAs[0].getAttribute('href') || '') : ''; if (postUrl.startsWith('//')) postUrl = 'https:' + postUrl;
      const source = fromAs[1] ? fromAs[1].textContent.trim() : '';
      const full = feed.querySelector('p[node-type="feed_list_content_full"]');
      const brief = feed.querySelector('p[node-type="feed_list_content"]');
      let content = ((full && full.innerText.trim()) ? full.innerText : (brief ? brief.innerText : '')).trim();
      content = content.replace(/\s*收起\s*d\s*$/, '').replace(/\s*展开\s*c\s*$/, '').trim();
      const rt = card.querySelector('.card-comment'); let retweet = '';
      if (rt) { const rn = rt.querySelector('a[nick-name]');
        const rp = rt.querySelector('p[node-type="feed_list_content_full"],p[node-type="feed_list_content"]');
        retweet = (rn ? ('@' + (rn.getAttribute('nick-name') || '') + '：') : '') + (rp ? rp.innerText.trim() : ''); }
      const imgs = [];
      card.querySelectorAll('.media-piclist img, div[node-type="fl_pic_list"] img, .m3 img').forEach(im => {
        let s = im.getAttribute('src') || im.getAttribute('data-src') || ''; if (!s) return;
        if (s.startsWith('//')) s = 'https:' + s;
        s = s.replace(/\/(orj360|thumb150|square|orj480|bmiddle|thumbnail)\//, '/mw2000/');
        if (!imgs.includes(s)) imgs.push(s);
      });
      const vd = card.querySelector('div[node-type="fl_h5_video"], .thumbnail');
      const acts = [...card.querySelectorAll('.card-act ul li')].map(li => li.innerText.replace(/\s/g, ''));
      if (content || retweet) out.push({ author, authorUrl, time, source, postUrl, content, retweet, images: imgs, hasVideo: !!vd, acts });
    });
    return out;
  });
  const log = [];
  for (const t of TOPICS) {
    const BASE = 'https://s.weibo.com' + t.href;
    let posts = [];
    for (let pg = 1; pg <= 4 && posts.length < LIMIT; pg++) {
      const u = pg === 1 ? BASE : BASE + '&page=' + pg;
      try { await page.goto(u, { waitUntil: 'domcontentloaded', timeout: 60000 }); } catch (e) { break; }
      await page.waitForTimeout(2300);
      await page.evaluate(async () => { for (let i = 0; i < 8; i++) { window.scrollBy(0, 900); await new Promise(r => setTimeout(r, 280)); } });
      await page.waitForTimeout(900);
      const more = await extract(); if (!more.length) break;
      posts = posts.concat(more);
    }
    const seen = new Set();
    posts = posts.filter(p => { const k = p.author + '|' + p.content.slice(0, 40); if (seen.has(k)) return false; seen.add(k); return true; }).slice(0, LIMIT);
    await page.request.post('http://127.0.0.1:8799/save/' + t.id, { data: JSON.stringify({ ...t, url: BASE, total: posts.length, posts }), headers: { 'Content-Type': 'application/json' } });
    log.push(t.id + ':' + t.title + ' -> ' + posts.length + ' posts');
  }
  return log.join('\n');
}
```

**关键经验（务必遵守）：**

- **作者名**：从 `.content .info a[nick-name]` 的 `nick-name` 属性读取。
- **完整原文**：优先 `p[node-type="feed_list_content_full"]`（「展开全文」后的完整内容，默认隐藏但 DOM 里有），为空才回退 `p[node-type="feed_list_content"]`；末尾要清掉 `收起 d` / `展开 c`。
- **互动数**：`.card-act ul li` 三项依次为转发/评论/赞；数量为 0 时文本是「转发」「评论」「赞」而非数字，需转成 `0`。
- **图片**：搜索页给的是 `orj360` 等缩略图，把路径段替换成 `mw2000` 可拿到高清原图（约 500KB～1MB/张）。
- **单页帖子数不足**：搜索页一页只有 ~15 条卡片，要抓 20 条必须翻页（URL 追加 `&page=2`），并按「作者+正文前 40 字」去重。
- **实时流**：热搜搜索结果每分钟都在变，同一话题两次抓取结果会不同，属正常现象，需在文档里注明。

## 第五步：下载图片 + 生成 Markdown

用本地 Node 脚本读取 JSON → 下载图片 → 写 md。

图片下载必须带 header，否则可能 403：

```js
const res = await fetch(url, { headers: {
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36',
  'Referer': 'https://weibo.com/'
}});
```

- `mw2000` 若 404 则回退 `orj360`；下载体积 < 500 字节视为失败。
- 已存在同名文件则跳过下载（便于重跑）。
- **写文件用 Node `fs.writeFileSync(file, str, { encoding: 'utf8' })`**，不要用 Python（用户机器上 Python 默认 locale 编码会写出乱码）。写完必须用 Read 工具抽查验证中文正常。

**⚠️ JS 陷阱**：不要用 `{'01':'x', ..., '10':'y'}` 这种对象存话题顺序——`'10'` 是合法数组索引字符串，`Object.keys()` 会把它排到最前面，导致索引表顺序错乱。**用二维数组** `[['01','x'], ...]` 保序，需要时再 `Object.fromEntries()`。

## 第六步：输出结构

```
微博热搜_YYYY-MM-DD/
├── 00_索引.md                ← 总览：抓取时间、热搜表格（排名/话题/热度/帖子数/文件）
├── 01_短标题.md
├── 02_短标题.md
├── ...
├── 10_短标题.md
└── images/
    ├── 01/ p01_1.jpg p02_1.jpg ...
    ├── 02/ ...
    └── 10/ ...
```

单个热搜文件格式：

```markdown
# 热搜 01：{完整热搜标题}

- **热搜排名**：第 1 名
- **热度值**：2,236,243
- **搜索链接**：{url}
- **抓取时间**：{YYYY-MM-DD HH:MM}
- **收录帖子**：20 条

---

## 帖子 1｜{作者名}

- **发布时间**：{time}　·　**来源**：{source}
- **互动**：转发 {n}　·　评论 {n}　·　点赞 {n}
- **作者主页**：{authorUrl}
- **原文链接**：{postUrl}

{完整正文原文，一字不改}

> **转发内容**：
> @原作者：...

**配图：**

![{作者} 配图1](images/01/p01_1.jpg)

---
```

md 文件命名：`序号_核心关键词.md`，短标题避免过长。索引表格里的文件链接要 `encodeURI()`，否则中文文件名在部分 Markdown 渲染器里点不开。

## 第七步：收尾

- 停掉后台接收服务，临时 JSON 留在系统临时目录即可（不要写进工作目录）。
- `present_files` 展示：第一个传 `00_索引.md`，随后依次传所有热搜 md 文件。
- 告知用户：帖子数、图片数、总体积，以及「数据为抓取时刻实时值」。

## 常见问题

- **热搜榜解析不到**：确认 `#pl_top_realtimehot` 存在；未登录也能看榜单，但搜索结果会受限，建议用户先在浏览器登录微博。
- **搜索结果为空**：话题带 `#` 的 URL 已 encode，直接用榜单里的 `href` 拼 `https://s.weibo.com` 即可，不要自己重新构造。
- **帖子数凑不够 M**：冷门话题实时流本身就少，翻到第 4 页仍不足则按实际数量收录，并在索引里如实标注。
- **图片 403**：检查是否带了 `Referer: https://weibo.com/`。
- **上下文爆掉**：说明没走 HTTP 落盘方案，改回第三步的做法。
