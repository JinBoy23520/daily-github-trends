# 📅 GitHub Daily Trending Report

> 每日 GitHub 热门项目追踪，支持按日期切换查看

**🌐 在线页面：** https://github.com/JinBoy23520/daily-github-trends

---

## 💻 本地代码位置

```
/Users/jin/.qclaw/workspace/daily-github-trends
```

> 修改后记得 `git add . && git commit -m "your change" && git push` 推送到 GitHub

---

## 📌 使用说明

打开上方链接，点击顶部日期按钮即可切换查看不同日期的热门榜单。

每日数据存档位于 `data/` 目录，格式为 `YYYY-MM-DD.json`。

---

## 📊 数据格式

每条记录包含以下字段：

| 字段 | 说明 |
|------|------|
| `repo` | 仓库全名（owner/name） |
| `desc` | 原始英文描述 |
| `stars` | 周期内累计 Stars |
| `lang` | 编程语言 |
| `use` | 中文用途说明 |

---

## ⚙️ 自动更新

- **QClaw 推送**：每天 08:00（北京时区）推送到 Web 聊天
- **GitHub Actions**：每天 00:00 UTC 自动抓取数据并提交到 `data/` 目录

---

*由 QClaw 自动生成与维护*
