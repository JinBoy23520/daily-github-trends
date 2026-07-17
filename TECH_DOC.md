# Daily GitHub Trends — 技术文档

## 📋 项目概述

每日自动抓取 GitHub Trending 三榜（daily / weekly / monthly）数据，推送到 GitHub 仓库，并通过定时任务推送 Markdown 表格到聊天渠道。

- **GitHub 仓库**：`JinBoy23520/daily-github-trends`
- **在线页面**：https://jinboy23520.github.io/daily-github-trends/
- **本地路径**：`/Users/jin/.qclaw/workspace/daily-github-trends`
- **数据格式**：`data/YYYY-MM-DD.json`

---

## 🏗️ 架构

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  GitHub       │     │  QClaw Agent      │     │  GitHub Repo     │
│  Trending     │────▶│  (抓取+解析+推送)  │────▶│  (data/*.json)   │
│  Page         │     │                   │     │                  │
└──────────────┘     └────────┬──────────┘     └─────────────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │  聊天渠道推送      │
                     │  (Markdown 表格)  │
                     └──────────────────┘
```

---

## 📁 目录结构

```
daily-github-trends/
├── README.md              # 项目说明
├── TECH_DOC.md            # 本技术文档
├── index.html             # GitHub Pages 静态页面（内嵌数据）
├── data/                  # 每日数据存档
│   ├── 2026-03-21.json
│   ├── 2026-03-22.json
│   ├── ...
│   └── 2026-07-17.json
└── .git/                  # Git 仓库
```

---

## 📊 数据格式

### JSON 文件结构（`data/YYYY-MM-DD.json`）

```json
{
  "date": "2026-07-17",
  "daily": [
    {
      "repo": "OpenCut-app/OpenCut",
      "stars": "3,537",
      "lang": "TypeScript",
      "desc": "The open-source CapCut alternative"
    }
  ],
  "weekly": [ ... ],
  "monthly": [ ... ]
}
```

### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `date` | string | 日期，格式 `YYYY-MM-DD` |
| `daily` | array | 日榜，通常 10-25 条 |
| `weekly` | array | 周榜，通常 15-25 条 |
| `monthly` | array | 月榜，通常 15-25 条 |
| `repo` | string | 仓库全名 `owner/name` |
| `stars` | string | 周期内累计 Stars（含千位逗号） |
| `lang` | string | 主编程语言，可为空 |
| `desc` | string | 仓库描述，截断为 80 字符 |

---

## 🔧 核心逻辑

### 1. 抓取模块

**数据源**：`https://github.com/trending?since={daily|weekly|monthly}`

**抓取方式**：`subprocess.run(['curl', '-sL', ...])` 绕过 Python urllib SSL 问题

**User-Agent 轮换**（防限速）：
```python
UA_LIST = [
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:128.0) Gecko/20100101 Firefox/128.0',
    'Mozilla/5.0 (iPhone; CPU iPhone OS 17_5 like Mac OS X) AppleWebKit/605.1.15 ...'
]
```

**重试策略**：
- 每个榜单尝试最多 3 个 UA
- UA 之间 sleep 2 秒
- 榜单之间 sleep 2-5 秒
- 全部失败则使用昨日数据 fallback

### 2. 解析模块

**HTML 结构**：GitHub Trending 页面使用 `<article>` 包裹每个仓库条目

**解析流程**：
```
1. regex 提取所有 <article>...</article>
2. 每个 article 内找 <h2> 标签
3. <h2> 内的 <a href="/owner/name"> 提取仓库链接
4. 提取 stars（正则匹配 N stars）
5. 提取语言（itemprop="programmingLanguage"）
6. 提取描述（class="col-9" 或 color-fg-muted）
```

**过滤规则**：跳过以下 owner（非仓库）：
```python
SKIP = {'sponsors','trending','organizations','topics','collections',
        'explore','settings','apps','about','contact','pricing','site','blog'}
```

### 3. 推送模块

**方式 A — git push（当前使用）**：
```bash
git add data/2026-07-17.json
git commit -m "docs: 2026-07-17 GitHub trending data" --author="JinBoy23520 <905054549@qq.com>"
git push origin main
```

**方式 B — GitHub Contents API（备用）**：
```bash
curl -X PUT \
  -H "Authorization: Bearer {TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"message":"...", "content":"base64...", "sha":"...", "committer":{...}}' \
  https://api.github.com/repos/JinBoy23520/daily-github-trends/contents/data/2026-07-17.json
```

### 4. 消息推送模块

将三榜数据格式化为 Markdown 表格，推送到聊天渠道：

```
📅 GitHub 热门榜 · YYYY-MM-DD

🕐 今日热榜
| # | 仓库 | ⭐ | 语言 | 用途 |
...

📆 本周热榜
...

📆 本月热榜
...

🔥 今日亮点：（AI 生成摘要）
```

---

## ⚙️ 配置

### Git Remote

```
https://JinBoy23520:{TOKEN}@github.com/JinBoy23520/daily-github-trends.git
```

### Token 要求

- 类型：Classic Personal Access Token（`repo` scope）或 Fine-grained Token
- 权限：仓库读写（`contents:write`）
- 过期：建议 `No expiration`
- 生成地址：https://github.com/settings/tokens/new

### Committer 信息

```
name:  JinBoy23520
email: 905054549@qq.com
```

---

## ⏰ 定时任务

| 配置项 | 值 |
|--------|-----|
| Cron Job ID | `fecfe090-997d-4198-a7b7-ae6686648052` |
| 调度 | 每天 08:00 Asia/Shanghai |
| Session Target | isolated |
| Payload | agentTurn — 执行抓取+推送+格式化消息 |

---

## 🌐 前端页面（index.html）

### 特性

- 暗色主题（GitHub Dark 风格）
- 日期切换按钮（自动列出所有可用日期）
- 三榜表格展示
- 内嵌 JSON 数据（非动态加载，需手动更新）
- 部署在 GitHub Pages

### 数据内嵌方式

`index.html` 中的 `DATA` 对象包含所有日期的数据，需手动更新。当前只有初始数据，后续数据需重新生成 index.html 或改为动态加载 `data/*.json`。

---

## 🔄 完整执行流程

```
1. 触发：用户说"今日推送" 或定时任务触发
2. 抓取 daily/weekly/monthly 三榜
3. 解析 HTML → JSON 数据
4. 保存到本地 data/YYYY-MM-DD.json
5. git add + commit + push 到 GitHub
6. 格式化 Markdown 表格推送消息
7. 生成今日亮点摘要
```

### 异常处理

| 场景 | 处理 |
|------|------|
| Daily 为空 | 重试（换 UA），最终 fallback 到昨日数据 |
| Weekly/Monthly 为空 | 重试（换 UA），最终使用已有数据 |
| Git push 失败 | 尝试 git pull --rebase 后重推，或改用 Contents API |
| Token 过期 | 提示用户生成新 Token |
| Git 冲突 | `git reset --hard origin/main` 后重新提交 |

---

## 📝 已知限制

1. **GitHub 限速**：频繁请求会被限速，表现为返回空页面。需 UA 轮换 + 延时
2. **Token 过期**：Classic Token 会过期，需定期更新
3. **index.html 数据陈旧**：内嵌数据未自动更新，只有 data/ 目录的 JSON 持续更新
4. **无自动重试**：整体推送失败后不会自动重试整个流程
5. **GitHub Actions 已移除**：`github-actions[bot]` 提交不计入贡献图，已删除 workflow

---

## 🛠️ 待优化

- [ ] index.html 改为动态加载 `data/*.json`，不再内嵌
- [ ] 抓取脚本独立为 `fetch_trending.py`，不再内联在对话中
- [ ] Token 更新自动化（检测过期 → 通知用户 → 更新 remote）
- [ ] 增加历史趋势对比（某仓库连续 N 天上榜）
- [ ] 增加语言分布统计
- [ ] 支持 spoken_language 过滤
- [ ] 数据可视化图表

---

## 📞 运维信息

| 项目 | 值 |
|------|-----|
| GitHub 用户 | JinBoy23520 |
| 邮箱 | 905054549@qq.com |
| 仓库 | JinBoy23520/daily-github-trends |
| 本地路径 | /Users/jin/.qclaw/workspace/daily-github-trends |
| Pages URL | https://jinboy23520.github.io/daily-github-trends/ |
| Cron Job ID | fecfe090-997d-4198-a7b7-ae6686648052 |
| 运行环境 | QClaw Agent on macOS (Darwin arm64) |

---

*最后更新：2026-07-17*
