# 🚀 AI新闻聚合项目 - 完整部署指南

## 📖 目录

1. [项目架构](#项目架构)
2. [运行策略](#运行策略)
3. [技术栈](#技术栈)
4. [完整部署流程](#完整部署流程)
5. [GitHub Actions自动化](#github-actions自动化)
6. [访问和验证](#访问和验证)
7. [常见问题](#常见问题)

---

## 📐 项目架构

### 核心组件

```
ai-news-radar/
├── index.html                    # 前端展示页面
├── scripts/
│   └── update_news.py           # Python爬虫脚本（核心）
├── data/                        # 数据存储目录（自动生成）
│   ├── latest-24h.json         # 最新24小时新闻
│   ├── archive.json            # 历史归档数据
│   ├── source-status.json      # 数据源状态
│   ├── waytoagi-7d.json        # WaytoAGI专属数据
│   └── title-zh-cache.json     # 中文标题缓存
├── feeds/
│   ├── follow.opml             # 私有RSS订阅（不提交到仓库）
│   └── follow.example.opml     # OPML模板
├── .github/workflows/
│   └── update-news.yml         # GitHub Actions配置
└── requirements.txt            # Python依赖
```

### 数据流

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Actions（云端）                     │
│                                                              │
│  1. 定时触发（每30分钟）                                       │
│  2. 安装Python环境                                           │
│  3. 解码OPML订阅（从Secret）                                  │
│  4. 运行爬虫脚本 ──────────┐                                  │
│                          ↓                                  │
│  ┌─────────────────────────────────────────┐                │
│  │  update_news.py 爬虫引擎                │                │
│  ├─────────────────────────────────────────┤                │
│  │  • 抓取10+网页源（TechURLs, Buzzing等）  │                │
│  │  • 解析RSS订阅（OPML）                   │                │
│  │  • 数据清洗和去重                        │                │
│  │  • 生成JSON数据文件                      │                │
│  └─────────────────────────────────────────┘                │
│                          ↓                                  │
│  5. 提交数据到GitHub仓库（data/*.json）                       │
│  6. 触发GitHub Pages更新                                     │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                  GitHub Pages（静态托管）                     │
│                                                              │
│  index.html 读取 data/*.json → 渲染新闻列表                   │
└─────────────────────────────────────────────────────────────┘
                          ↓
                  用户浏览器访问网站
```

---

## ⚙️ 运行策略

### 1. 自动化触发机制

项目使用 **GitHub Actions** 实现全自动运行，无需本地服务器：

```yaml
# .github/workflows/update-news.yml
on:
  workflow_dispatch:      # 手动触发
  schedule:
    - cron: "*/30 * * * *"  # 每30分钟自动触发
```

**触发时间**：
- 自动：每小时的 **0分** 和 **30分**（例如：10:00, 10:30, 11:00, 11:30...）
- 手动：随时在GitHub Actions页面点击 "Run workflow"

### 2. 数据更新策略

#### 窗口期设置
```python
--window-hours 24  # 抓取最近24小时的新闻
```

#### 数据源优先级
1. **网页爬虫源**（高优先级）
   - TechURLs (https://techurls.com)
   - Buzzing.cc
   - InfoFlow
   - 其他AI资讯网站

2. **RSS订阅源**（可选）
   - 从 `feeds/follow.opml` 读取
   - 支持任意RSS/Atom feeds

#### 去重和缓存
- 使用 `title-zh-cache.json` 缓存已处理标题
- 根据URL和标题进行去重
- 归档历史数据到 `archive.json`（最多保存64,000+条记录）

### 3. 权限和安全

```yaml
permissions:
  contents: write  # 允许提交代码
```

**安全机制**：
- OPML订阅使用 **GitHub Secrets** 加密存储（Base64编码）
- 不会将私有订阅暴露到公开仓库
- Actions运行在隔离的GitHub云环境

---

## 🛠 技术栈

### 后端（Python爬虫）
- **requests** - HTTP请求
- **beautifulsoup4** - HTML解析
- **feedparser** - RSS/Atom解析
- **python-dateutil** - 时间处理
- **tzdata** - 时区数据（Windows兼容）

### 前端（静态网页）
- **HTML/CSS/JavaScript** - 纯静态页面
- **JSON** - 数据格式
- **GitHub Pages** - 免费静态托管

### 自动化
- **GitHub Actions** - CI/CD
- **Cron表达式** - 定时任务

---

## 🚀 完整部署流程

### 阶段1：本地准备（一次性）

#### 1.1 Fork项目
```
访问：https://github.com/waytoagi/ai-news-radar
点击右上角 Fork 按钮
```

#### 1.2 Clone到本地
```powershell
cd D:\application\illegal
git clone https://github.com/YOUR_USERNAME/ai-news-radar.git
cd ai-news-radar
```

#### 1.3 配置Python环境
```powershell
# 创建虚拟环境
python -m venv .venv

# 激活虚拟环境（Windows PowerShell）
.\.venv\Scripts\Activate.ps1

# 安装依赖
pip install -r requirements.txt
```

**依赖列表**：
```
requests==2.32.3
beautifulsoup4==4.12.3
feedparser==6.0.11
python-dateutil==2.9.0.post0
tzdata==2025.3  # Windows必需
```

#### 1.4 配置RSS订阅（可选）
```powershell
# 复制模板
Copy-Item feeds/follow.example.opml feeds/follow.opml

# 编辑 feeds/follow.opml 添加你的RSS源
notepad feeds/follow.opml
```

OPML格式示例：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<opml version="2.0">
  <head>
    <title>My AI News Feeds</title>
  </head>
  <body>
    <outline text="AI新闻源1" 
             xmlUrl="https://example.com/feed.xml" 
             htmlUrl="https://example.com"/>
    <outline text="AI新闻源2" 
             xmlUrl="https://another.com/rss" 
             htmlUrl="https://another.com"/>
  </body>
</opml>
```

---

### 阶段2：本地测试

#### 2.1 手动运行爬虫
```powershell
# 激活虚拟环境
.\.venv\Scripts\Activate.ps1

# 运行爬虫（有OPML订阅）
python scripts/update_news.py --output-dir data --window-hours 24 --rss-opml feeds/follow.opml

# 运行爬虫（无OPML订阅）
python scripts/update_news.py --output-dir data --window-hours 24
```

**预期输出**：
```
Wrote: data\latest-24h.json (1998 items)
Wrote: data\archive.json (64752 items)
Wrote: data\source-status.json
Wrote: data\waytoagi-7d.json (36 items)
Wrote: data\title-zh-cache.json (1571 entries)
```

#### 2.2 启动本地预览
```powershell
python -m http.server 8080
```

访问：http://localhost:8080

**验证页面功能**：
- ✅ 显示最新24小时AI新闻
- ✅ 中英双语切换
- ✅ AI强相关/全量切换
- ✅ 时间戳显示正确

---

### 阶段3：GitHub配置（关键）

#### 3.1 生成OPML Base64（如果使用RSS订阅）

**Windows PowerShell**：
```powershell
# 读取OPML并转换为Base64
$opmlContent = Get-Content feeds/follow.opml -Raw
$bytes = [System.Text.Encoding]::UTF8.GetBytes($opmlContent)
$base64 = [Convert]::ToBase64String($bytes)

# 复制到剪贴板
$base64 | Set-Clipboard

# 显示确认
Write-Host "✅ OPML Base64已复制到剪贴板！" -ForegroundColor Green
```

#### 3.2 配置GitHub Secrets

1. **访问**：`https://github.com/YOUR_USERNAME/ai-news-radar/settings/secrets/actions`

2. **添加Secret**：
   - 点击 **New repository secret**
   - **Name**: `FOLLOW_OPML_B64`
   - **Value**: 粘贴刚才复制的Base64字符串
   - 点击 **Add secret**

**为什么用Base64？**
- GitHub Secrets不支持多行文本
- Base64编码将XML转换为单行字符串
- Actions运行时会自动解码

#### 3.3 配置Actions权限

1. **访问**：`https://github.com/YOUR_USERNAME/ai-news-radar/settings/actions`

2. **设置权限**：
   - **Actions permissions**: 
     - ✅ Allow all actions and reusable workflows
   
   - **Workflow permissions**: 
     - ✅ Read and write permissions
     - ✅ Allow GitHub Actions to create and approve pull requests
   
   - 点击 **Save**

**为什么需要写权限？**
- Actions需要提交更新的数据文件到仓库
- 自动更新 `data/*.json` 文件

#### 3.4 提交初始数据

```powershell
# 配置Git用户
git config user.name "YOUR_USERNAME"
git config user.email "YOUR_EMAIL@example.com"

# 添加数据文件
git add data/
git commit -m "Initial news data"

# 推送到GitHub
git push origin master
```

---

### 阶段4：启动自动化

#### 4.1 手动触发第一次运行

1. **访问Actions页面**：
   ```
   https://github.com/YOUR_USERNAME/ai-news-radar/actions
   ```

2. **触发workflow**：
   - 左侧点击 **"Update AI News Snapshot"**
   - 右侧点击 **"Run workflow"** 按钮
   - 选择分支 **master**
   - 再次点击绿色的 **"Run workflow"**

3. **观察运行状态**（2-3分钟）：
   - 🟡 黄色圆点 = 正在运行
   - 🟢 绿色勾 = 成功
   - 🔴 红色叉 = 失败（点击查看日志）

#### 4.2 验证Actions日志

点击运行中的workflow，查看各步骤：

```
✅ Checkout                    # 拉取代码
✅ Setup Python                # 安装Python 3.11
✅ Install dependencies        # 安装pip包
✅ Prepare OPML                # 解码OPML Secret
✅ Update data                 # 运行爬虫脚本
✅ Commit and push changes     # 提交数据
```

**如果失败**，点击红色步骤查看详细错误：
- 检查Secret是否正确配置
- 检查权限是否开启
- 查看Python脚本错误日志

---

### 阶段5：部署GitHub Pages

#### 5.1 启用Pages

1. **访问**：`https://github.com/YOUR_USERNAME/ai-news-radar/settings/pages`

2. **配置**：
   - **Source**: Deploy from a branch
   - **Branch**: master
   - **Directory**: / (root)
   - 点击 **Save**

#### 5.2 等待部署（1-2分钟）

页面会显示：
```
Your site is live at https://YOUR_USERNAME.github.io/ai-news-radar/
```

#### 5.3 访问网站

访问：`https://YOUR_USERNAME.github.io/ai-news-radar/`

**验证功能**：
- ✅ 页面正常加载
- ✅ 显示最新新闻
- ✅ 数据来自 `data/latest-24h.json`
- ✅ 时间戳为最近时间

---

## 🔄 GitHub Actions自动化详解

### Workflow配置解析

```yaml
name: Update AI News Snapshot

on:
  workflow_dispatch:           # 允许手动触发
  schedule:
    - cron: "*/30 * * * *"    # 每30分钟触发一次

permissions:
  contents: write              # 写权限（提交数据）

jobs:
  update:
    runs-on: ubuntu-latest     # 运行环境：Ubuntu虚拟机
    
    steps:
      # 步骤1：拉取代码
      - name: Checkout
        uses: actions/checkout@v4
      
      # 步骤2：安装Python 3.11
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      
      # 步骤3：安装依赖
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      
      # 步骤4：准备OPML（从Secret解码）
      - name: Prepare OPML (optional)
        env:
          FOLLOW_OPML_B64: ${{ secrets.FOLLOW_OPML_B64 }}
        run: |
          mkdir -p feeds
          if [ -n "$FOLLOW_OPML_B64" ]; then
            echo "$FOLLOW_OPML_B64" | base64 --decode > feeds/follow.opml
            echo "Loaded feeds/follow.opml from FOLLOW_OPML_B64 secret"
          else
            echo "No FOLLOW_OPML_B64 secret provided; RSS OPML is optional"
          fi
      
      # 步骤5：运行爬虫
      - name: Update data
        run: |
          if [ -f feeds/follow.opml ]; then
            python scripts/update_news.py --output-dir data --window-hours 24 --rss-opml feeds/follow.opml
          else
            python scripts/update_news.py --output-dir data --window-hours 24
          fi
      
      # 步骤6：提交和推送
      - name: Commit and push changes
        run: |
          if git diff --quiet; then
            echo "No changes to commit"
            exit 0
          fi
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add data/latest-24h.json data/archive.json data/source-status.json data/waytoagi-7d.json data/title-zh-cache.json
          git commit -m "chore: update ai news snapshot"
          git push
```

### Cron表达式说明

```
"*/30 * * * *"
 │   │ │ │ │
 │   │ │ │ └─ 星期几 (0-6, 0=周日)
 │   │ │ └─── 月份 (1-12)
 │   │ └───── 日期 (1-31)
 │   └─────── 小时 (0-23)
 └─────────── 分钟 (0-59)
```

**当前配置**：`*/30` = 每30分钟
- 实际触发时间：00:00, 00:30, 01:00, 01:30, ..., 23:30

**常用配置**：
- 每小时：`0 * * * *`
- 每2小时：`0 */2 * * *`
- 每6小时：`0 */6 * * *`
- 每天凌晨：`0 0 * * *`

### 运行成本

**完全免费**：
- GitHub Actions：公开仓库无限免费分钟
- GitHub Pages：免费静态托管（1GB空间）
- 无需服务器、无需域名

---

## 🌐 访问和验证

### 1. 本地访问

```powershell
cd D:\application\illegal\ai-news-radar
.\.venv\Scripts\Activate.ps1
python -m http.server 8080
```

访问：http://localhost:8080

### 2. GitHub Pages访问

```
https://YOUR_USERNAME.github.io/ai-news-radar/
```

### 3. 数据API访问

可以直接访问JSON数据：

```
https://YOUR_USERNAME.github.io/ai-news-radar/data/latest-24h.json
https://YOUR_USERNAME.github.io/ai-news-radar/data/archive.json
https://YOUR_USERNAME.github.io/ai-news-radar/data/source-status.json
```

### 4. 验证自动更新

1. **查看提交历史**：
   ```
   https://github.com/YOUR_USERNAME/ai-news-radar/commits/master
   ```
   应该看到每30分钟一次的自动提交

2. **查看Actions历史**：
   ```
   https://github.com/YOUR_USERNAME/ai-news-radar/actions
   ```
   应该看到定期运行的workflow

3. **检查数据时间戳**：
   访问网站，查看新闻时间是否为最近30分钟内

---

## ❓ 常见问题

### Q1: Actions运行失败怎么办？

**排查步骤**：

1. **检查权限**：
   - Settings > Actions > General
   - Workflow permissions = Read and write ✅

2. **检查Secret**：
   - Settings > Secrets > Actions
   - FOLLOW_OPML_B64 存在且正确 ✅

3. **查看错误日志**：
   - Actions页面 > 点击失败的run
   - 点击红色的步骤查看详细错误

**常见错误**：

| 错误 | 原因 | 解决 |
|------|------|------|
| `Permission denied` | 无写权限 | 开启 Read and write permissions |
| `No module named 'tzdata'` | 依赖缺失 | 添加 `tzdata==2025.3` 到 requirements.txt |
| `base64: invalid input` | OPML Secret格式错误 | 重新生成Base64 |
| `git push failed` | 无提交权限 | 检查workflow permissions |

### Q2: 如何修改抓取频率？

编辑 `.github/workflows/update-news.yml`:

```yaml
schedule:
  - cron: "0 */2 * * *"  # 改为每2小时
```

提交并推送即可生效。

### Q3: 如何添加自定义RSS源？

1. **编辑本地OPML**：
   ```powershell
   notepad feeds/follow.opml
   ```

2. **添加RSS源**：
   ```xml
   <outline text="新RSS源" 
            xmlUrl="https://example.com/feed.xml" 
            htmlUrl="https://example.com"/>
   ```

3. **重新生成Base64并更新Secret**：
   ```powershell
   $opmlContent = Get-Content feeds/follow.opml -Raw
   $bytes = [System.Text.Encoding]::UTF8.GetBytes($opmlContent)
   $base64 = [Convert]::ToBase64String($bytes)
   $base64 | Set-Clipboard
   ```
   
   然后在GitHub Secrets中更新 `FOLLOW_OPML_B64`

### Q4: GitHub Pages没有更新怎么办？

1. **检查Pages是否启用**：
   - Settings > Pages
   - Source = Deploy from a branch
   - Branch = master ✅

2. **强制刷新浏览器**：
   - Windows: `Ctrl + F5`
   - Mac: `Cmd + Shift + R`

3. **检查Actions是否提交了数据**：
   - 查看最新commit是否包含 `data/*.json` 更新

4. **等待部署完成**：
   - GitHub Pages更新需要1-2分钟
   - 查看 Actions > pages-build-deployment

### Q5: 如何在本地测试不推送到GitHub？

```powershell
# 运行爬虫
python scripts/update_news.py --output-dir data --window-hours 24

# 查看生成的数据
Get-Content data/latest-24h.json | ConvertFrom-Json | Select-Object -First 5

# 启动本地服务器
python -m http.server 8080

# 测试完后不要 git push
```

### Q6: 数据太多占用空间怎么办？

可以修改归档策略：

编辑 `scripts/update_news.py`，找到归档部分：

```python
# 限制归档数量
MAX_ARCHIVE = 50000  # 改小这个数字
```

### Q7: 如何自定义网站样式？

编辑 `index.html`，修改CSS部分：

```html
<style>
  /* 自定义样式 */
  body {
    background-color: #f0f0f0;  /* 背景色 */
  }
  
  .news-item {
    font-size: 16px;  /* 字体大小 */
  }
</style>
```

提交推送后，GitHub Pages会自动更新。

---

## 🎯 总结

### 整体运行机制

```
1. 本地准备
   ├─ Fork项目
   ├─ Clone到本地
   ├─ 安装Python环境
   ├─ 配置OPML订阅
   └─ 测试本地运行

2. GitHub配置
   ├─ 生成OPML Base64
   ├─ 添加GitHub Secret
   ├─ 开启Actions权限
   └─ 推送初始数据

3. 启动自动化
   ├─ 手动触发第一次
   ├─ 验证运行成功
   └─ 后续自动运行（每30分钟）

4. 部署Pages
   ├─ 启用GitHub Pages
   ├─ 等待部署完成
   └─ 访问公开网站

5. 持续运行
   ├─ Actions自动抓取
   ├─ 自动提交数据
   ├─ Pages自动更新
   └─ 完全免费、无需维护
```

### 关键优势

- ✅ **完全免费**：无服务器、无域名成本
- ✅ **全自动化**：每30分钟自动更新
- ✅ **无需维护**：在云端运行，本地无需开机
- ✅ **数据安全**：OPML订阅加密存储
- ✅ **公开访问**：通过GitHub Pages分享
- ✅ **可自定义**：支持自定义RSS源和样式

### 下一步

1. **手动触发第一次Actions运行**
2. **验证数据是否更新**
3. **启用GitHub Pages**
4. **访问并分享你的AI新闻网站**

---

**项目地址**：https://github.com/zc627788/ai-news-radar
**网站地址**：https://zc627788.github.io/ai-news-radar/

🎊 **享受自动更新的AI新闻聚合！**
