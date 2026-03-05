# Twitter CLI

Twitter/X 命令行工具 — 读取 Timeline、书签和用户信息。

**零 API Key** — 使用浏览器 Cookie 认证，免费访问 Twitter。

## 安装

```bash
# 推荐：uv tool（更快，隔离环境）
uv tool install twitter-cli

# 其次：pipx
pipx install twitter-cli
```

从源码安装：

```bash
git clone git@github.com:jackwener/twitter-cli.git
cd twitter-cli
uv sync
```

## Quick Start

```bash
# 运行（自动从 Chrome 提取 Cookie）
twitter feed
```

首次运行确保 Chrome 已登录 x.com。

## 使用方式

### 读取

```bash
# 抓取首页 timeline（For You 算法推荐）
twitter feed

# 抓取关注的人的 timeline（Following 时间线）
twitter feed -t following

# 自定义抓取条数
twitter feed --max 50

# 开启筛选（按 score 排序过滤）
twitter feed --filter

# JSON 输出
twitter feed --json > tweets.json

# 从已有数据加载
twitter feed --input tweets.json


# 抓取收藏
twitter favorite
twitter favorite --max 30 --json
```

### 用户

```bash
# 查看用户资料
twitter user elonmusk

# 列出用户推文
twitter user-posts elonmusk --max 20
```

## Pipeline

```
抓取 (GraphQL API)  →  筛选 (Engagement Score)
      50 条               top 20
```

### 筛选算法

加权评分公式，收藏权重最高（代表"值得回看"）：

```
score = 1.0 × likes + 3.0 × retweets + 2.0 × replies
      + 5.0 × bookmarks + 0.5 × log10(views)
```

## 配置

编辑 `config.yaml`：

```yaml
fetch:
  count: 50

filter:
  mode: "topN"          # "topN" | "score" | "all"
  topN: 20
  weights:
    likes: 1.0
    retweets: 3.0
    replies: 2.0
    bookmarks: 5.0
    views_log: 0.5
```

### Cookie 配置

**方式 1：自动提取**（推荐） — 确保浏览器已登录 x.com，程序自动通过 `browser-cookie3` 按 Chrome → Edge → Firefox → Brave 顺序尝试读取。

**方式 2：环境变量** — 设置：

```bash
export TWITTER_AUTH_TOKEN=your_auth_token
export TWITTER_CT0=your_ct0
```

可通过 [Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm) 浏览器插件导出。

## 项目结构

```
twitter_cli/
├── __init__.py     # 版本信息
├── cli.py          # CLI 入口 (click)
├── client.py       # Twitter GraphQL API Client (GET)
├── auth.py         # Cookie 提取 (env / browser-cookie3)
├── filter.py       # Engagement scoring + 筛选
├── formatter.py    # Rich 终端输出 + JSON
├── config.py       # YAML 配置加载
├── serialization.py # Tweet JSON <-> dataclass
└── models.py       # 数据模型 (dataclass)
```

## Development

```bash
# Install development tools
uv sync --extra dev

# Run tests
uv run pytest

# Lint
uv run ruff check .
```

## 注意事项

- 使用 Cookie 登录存在被平台检测的风险，建议使用**专用小号**
- Cookie 只存在本地，不上传不外传
- GraphQL `queryId` 会从 Twitter 前端 JS 自动检测，无需手动维护
