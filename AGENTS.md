# AI Agent Guide for TrendRadar

> **Last Updated:** 2025-11-15
> **Version:** 3.0.5 (main crawler) / 1.0.1 (MCP server)

This document provides comprehensive guidance for AI agents (like Jules, Claude, etc.) working with the TrendRadar codebase. It covers project structure, development workflows, conventions, and best practices.

---

## 📋 Table of Contents

1. [Project Overview](#-project-overview)
2. [Repository Structure](#-repository-structure)
3. [Technology Stack](#-technology-stack)
4. [Architecture & Design Patterns](#-architecture--design-patterns)
5. [Development Workflows](#-development-workflows)
6. [Configuration Management](#-configuration-management)
7. [Code Conventions](#-code-conventions)
8. [Testing & CI/CD](#-testing--cicd)
9. [MCP Server Integration](#-mcp-server-integration)
10. [Common Tasks](#-common-tasks)
11. [Troubleshooting](#-troubleshooting)

---

## 🎯 Project Overview

### What is TrendRadar?

TrendRadar is a **lightweight, production-ready Chinese news aggregation system** that:

- **Crawls** hot topics from 11+ Chinese news platforms (Weibo, Zhihu, Douyin, Baidu, etc.)
- **Filters** news based on user-defined frequency keywords (`config/frequency_words.txt`)
- **Aggregates** and ranks using a weighted scoring algorithm
- **Notifies** via multiple channels (Telegram, Email, DingTalk, Feishu, WeWork, ntfy)
- **Provides AI analysis** through Model Context Protocol (MCP) server integration

### Project Goals

- **Lightweight**: Minimal dependencies, easy deployment
- **Configurable**: Extensive YAML configuration without code changes
- **Multi-deployment**: GitHub Actions (zero-config), Docker, local Python, or MCP server
- **Production-ready**: Mature codebase with CI/CD automation

### Key Features

✅ **11+ News Platforms** - Comprehensive Chinese news coverage
✅ **Keyword Monitoring** - Custom frequency word filtering
✅ **Smart Ranking** - Weighted algorithm (rank, frequency, hotness)
✅ **6 Notification Channels** - Flexible push options
✅ **3 Report Modes** - Daily, current, incremental
✅ **13 MCP Tools** - Full AI analysis integration
✅ **Multi-arch Docker** - AMD64 + ARM64 support
✅ **GitHub Actions** - Automated hourly crawling

---

## 📁 Repository Structure

```
TrendRadar/
├── .github/
│   ├── ISSUE_TEMPLATE/          # Issue templates (bug, feature, config help)
│   │   ├── 01-bug-report.yml
│   │   ├── 02-feature-request.yml
│   │   └── 03-config-help.yml
│   └── workflows/
│       ├── crawler.yml          # Hourly news crawler automation
│       └── docker.yml           # Multi-arch Docker builds (on tags)
│
├── _image/                      # Documentation images and assets
│
├── config/
│   ├── config.yaml             # Main configuration file (4.6KB)
│   │                           # - App settings, crawler config
│   │                           # - Report modes, notification webhooks
│   │                           # - Platform definitions, weight algorithm
│   └── frequency_words.txt     # User-defined keywords (114 lines)
│
├── docker/
│   ├── Dockerfile              # Multi-arch support (amd64, arm64)
│   ├── docker-compose.yml      # Production deployment
│   ├── docker-compose-build.yml
│   ├── entrypoint.sh           # Container entry point
│   └── manage.py               # Docker management script
│
├── mcp_server/                 # MCP Server implementation
│   ├── __init__.py
│   ├── server.py               # FastMCP 2.0 main server (697 lines)
│   │                           # Entry point: python -m mcp_server.server
│   │
│   ├── services/               # Business logic layer
│   │   ├── __init__.py
│   │   ├── cache_service.py    # Caching layer for performance
│   │   ├── data_service.py     # Data access layer (20KB)
│   │   └── parser_service.py   # Data parsing logic (12KB)
│   │
│   ├── tools/                  # MCP tool implementations (13 tools)
│   │   ├── __init__.py
│   │   ├── analytics.py        # Analytics tools (74KB - largest file)
│   │   ├── config_mgmt.py      # Configuration management
│   │   ├── data_query.py       # Data query tools
│   │   ├── search_tools.py     # Search functionality (26KB)
│   │   └── system.py           # System management tools (18KB)
│   │
│   └── utils/                  # Utility modules
│       ├── __init__.py
│       ├── date_parser.py      # Date parsing utilities
│       ├── errors.py           # Custom error definitions
│       └── validators.py       # Input validation
│
├── main.py                     # Main crawler script (4,554 lines)
│                               # - News aggregation logic
│                               # - Email/notification sending
│                               # - Data persistence to JSON
│
├── index.html                  # GitHub Pages visualization (26KB)
│                               # Hosted at: https://sansan0.github.io/TrendRadar
│
├── pyproject.toml              # Python project metadata
├── requirements.txt            # Python dependencies (5 packages)
├── version                     # Version file for update checks
│
├── setup-mac.sh                # macOS automated setup (uses UV)
├── setup-windows.bat           # Windows setup (Chinese)
├── setup-windows-en.bat        # Windows setup (English)
├── start-http.sh               # HTTP mode server launcher
├── start-http.bat              # HTTP mode server launcher (Windows)
│
├── readme.md                   # Primary documentation (66KB)
├── README-Cherry-Studio.md     # Cherry Studio MCP client guide
├── README-MCP-FAQ.md           # MCP-specific FAQ
└── LICENSE                     # GPL-3.0 License

```

### Critical Files

| File | Purpose | Size | Notes |
|------|---------|------|-------|
| `main.py` | Main crawler script | 171KB (4,554 lines) | Core news aggregation logic |
| `mcp_server/server.py` | MCP server entry point | 697 lines | 13 MCP tools registered |
| `config/config.yaml` | Main configuration | 4.6KB | **DO NOT hardcode secrets here** |
| `config/frequency_words.txt` | Keyword list | 114 lines | One keyword per line |
| `.github/workflows/crawler.yml` | Hourly automation | 2.6KB | Uses GitHub Secrets |
| `mcp_server/tools/analytics.py` | Analytics tools | 74KB | Largest file, complex logic |

---

## 🛠️ Technology Stack

### Core Dependencies

```python
# requirements.txt & pyproject.toml
requests>=2.32.5,<3.0.0      # HTTP client for web scraping
pytz>=2025.2,<2026.0          # Timezone handling (Beijing time)
PyYAML>=6.0.3,<7.0.0          # YAML configuration parser
fastmcp>=2.12.0,<2.14.0       # MCP server framework (FastMCP 2.0)
websockets>=13.0,<14.0        # WebSocket support for MCP
```

### Technology Breakdown

#### Backend Framework
- **Python 3.10+** - Minimum version requirement
- **FastMCP 2.0** - Model Context Protocol server framework
- **Requests** - HTTP client for API calls to news platforms

#### Data Sources
- **NewsNow API** - Primary data source ([newsnow](https://github.com/ourongxing/newsnow))
- **11+ Chinese Platforms** - Weibo, Zhihu, Douyin, Baidu, Bilibili, etc.

#### Notification Services
- **SMTP** - Email (Gmail, QQ, Outlook, 163, 126, Sina, Sohu)
- **Telegram Bot API** - Telegram notifications
- **Webhook APIs** - DingTalk, Feishu, WeWork
- **ntfy** - Self-hosted push notifications

#### Deployment
- **Docker** - Multi-architecture (amd64, arm64)
- **GitHub Actions** - Automated hourly crawling
- **Supercronic** - Cron scheduler in Docker containers

#### Build Tools
- **UV** - Fast Python package manager (recommended for Mac)
- **Hatchling** - Build backend for pyproject.toml

---

## 🏗️ Architecture & Design Patterns

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    TrendRadar System                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐        ┌──────────────┐                   │
│  │  main.py     │        │ MCP Server   │                   │
│  │  (Crawler)   │        │  (server.py) │                   │
│  └──────┬───────┘        └──────┬───────┘                   │
│         │                       │                            │
│         ├─ Crawl news           ├─ FastMCP 2.0 (13 tools)   │
│         ├─ Filter keywords      ├─ STDIO/HTTP transport     │
│         ├─ Weight algorithm     └─ AI analysis layer        │
│         ├─ Push notifications                                │
│         └─ Save to JSON files                                │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Configuration Layer                      │   │
│  │  • config/config.yaml (main settings)                 │   │
│  │  • config/frequency_words.txt (keywords)              │   │
│  │  • Environment variables (secrets)                    │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Data Persistence Layer                   │   │
│  │  • JSON files (crawler output)                        │   │
│  │  • In-memory cache (MCP server)                       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### MCP Server Layered Architecture

```
┌────────────────────────────────────────────────────┐
│          Presentation Layer (FastMCP Tools)         │
│  13 tools: data_query, analytics, search, etc.     │
└─────────────────┬──────────────────────────────────┘
                  │
┌─────────────────▼──────────────────────────────────┐
│            Service Layer (Business Logic)           │
│  • data_service.py - Data access & filtering        │
│  • cache_service.py - Performance caching           │
│  • parser_service.py - Data parsing & transformation│
└─────────────────┬──────────────────────────────────┘
                  │
┌─────────────────▼──────────────────────────────────┐
│            Utility Layer (Cross-cutting)            │
│  • date_parser.py - Date parsing utilities          │
│  • validators.py - Input validation                 │
│  • errors.py - Custom exceptions                    │
└────────────────────────────────────────────────────┘
```

### Design Patterns Used

#### 1. **Singleton Pattern**
- **Location:** `mcp_server/server.py`
- **Usage:** Tool instances (`_tools_instances`)
- **Purpose:** Reuse tool objects across requests

```python
_tools_instances = {}

def _get_tools(project_root: Optional[str] = None):
    """获取或创建工具实例（单例模式）"""
    if not _tools_instances:
        _tools_instances['data'] = DataQueryTools(project_root)
        _tools_instances['analytics'] = AnalyticsTools(project_root)
        # ...
    return _tools_instances
```

#### 2. **Configuration-Driven Design**
- **Location:** `config/config.yaml`
- **Pattern:** External configuration with environment variable overrides
- **Benefits:** No code changes needed for deployment variations

#### 3. **Layered Architecture**
- **Layers:** Presentation → Service → Utility
- **Separation:** Clear boundaries between UI, business logic, and utilities
- **Testability:** Each layer can be tested independently

#### 4. **Weighted Ranking Algorithm**
- **Location:** `config/config.yaml` (weight section)
- **Formula:** `score = rank_weight × rank + frequency_weight × frequency + hotness_weight × hotness`
- **Default Weights:**
  ```yaml
  weight:
    rank_weight: 0.6       # Platform ranking (60%)
    frequency_weight: 0.3   # Keyword frequency (30%)
    hotness_weight: 0.1     # Hotness metric (10%)
  ```

---

## 🔄 Development Workflows

### Local Development Setup

#### Option 1: Using UV (Recommended for macOS)

```bash
# 1. Clone repository
git clone https://github.com/sansan0/TrendRadar.git
cd TrendRadar

# 2. Run automated setup
chmod +x setup-mac.sh
./setup-mac.sh

# 3. Edit configuration
nano config/config.yaml
nano config/frequency_words.txt

# 4. Run crawler
python main.py
```

#### Option 2: Manual Setup

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Configure
cp config/config.yaml config/config.yaml.local
# Edit config.yaml.local

# 3. Run crawler
python main.py

# 4. Run MCP server (STDIO mode)
python -m mcp_server.server

# 5. Run MCP server (HTTP mode)
./start-http.sh  # or start-http.bat on Windows
```

### Docker Development

```bash
# Build image
cd docker
docker compose -f docker-compose-build.yml build

# Run container
docker compose up -d

# View logs
docker compose logs -f

# Access container
docker exec -it trendradar sh

# Stop container
docker compose down
```

### GitHub Actions Development

1. **Fork repository** to your GitHub account
2. **Add secrets** at Settings → Secrets and variables → Actions:
   - `FEISHU_WEBHOOK_URL`
   - `TELEGRAM_BOT_TOKEN`
   - `TELEGRAM_CHAT_ID`
   - `DINGTALK_WEBHOOK_URL`
   - `EMAIL_FROM`
   - `EMAIL_PASSWORD`
   - etc.
3. **Enable Actions** at Actions tab
4. **Manual trigger:** Actions → News Crawler → Run workflow

### MCP Server Development

```bash
# STDIO mode (for Cherry Studio, Claude Desktop)
python -m mcp_server.server

# HTTP mode (for web clients)
python -m mcp_server.server --transport http --port 8000

# Test with MCP client
# Configure client to use: python -m mcp_server.server
```

---

## ⚙️ Configuration Management

### Configuration Files

#### 1. `config/config.yaml` - Main Configuration

**Structure:**

```yaml
app:
  version_check_url: "..."         # Version update check URL
  show_version_update: true        # Show update notifications

crawler:
  request_interval: 1000           # Request interval (ms)
  enable_crawler: true             # Enable/disable crawling
  use_proxy: false                 # Proxy toggle
  default_proxy: "http://..."      # Proxy URL

report:
  mode: "daily"                    # Report mode: daily|current|incremental
  rank_threshold: 5                # Ranking highlight threshold

notification:
  enable_notification: true        # Enable notifications
  message_batch_size: 4000         # Message batch size (bytes)
  push_window:                     # Push time window control
    enabled: false
    time_range:
      start: "20:00"
      end: "22:00"
  webhooks:                        # ⚠️ DO NOT commit secrets here
    feishu_url: ""
    telegram_bot_token: ""
    # ... (use GitHub Secrets instead)

weight:
  rank_weight: 0.6                 # Platform ranking weight
  frequency_weight: 0.3            # Keyword frequency weight
  hotness_weight: 0.1              # Hotness metric weight

platforms:                         # Platform definitions
  - id: "toutiao"
    name: "今日头条"
  # ... (11+ platforms)
```

**Important Notes:**

⚠️ **NEVER commit webhooks/tokens to `config.yaml`**
✅ **Use environment variables or GitHub Secrets for sensitive data**
✅ **Use `config.yaml` for structure, GitHub Secrets for values**

#### 2. `config/frequency_words.txt` - Keyword Monitoring

**Format:**
```
华为
特斯拉
AI
芯片
# Comments are supported
```

**Guidelines:**
- One keyword per line
- Case-sensitive matching
- Supports Chinese and English
- Lines starting with `#` are ignored

### Environment Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `FEISHU_WEBHOOK_URL` | Feishu bot webhook | `https://open.feishu.cn/...` |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token | `123456:ABC-DEF...` |
| `TELEGRAM_CHAT_ID` | Telegram chat ID | `@channel_name` |
| `DINGTALK_WEBHOOK_URL` | DingTalk webhook | `https://oapi.dingtalk.com/...` |
| `EMAIL_FROM` | Sender email | `user@gmail.com` |
| `EMAIL_PASSWORD` | Email password/app password | `xxxx xxxx xxxx xxxx` |
| `EMAIL_TO` | Recipient emails (comma-separated) | `user1@gmail.com,user2@qq.com` |
| `NTFY_TOPIC` | ntfy topic name | `my-topic-name` |
| `NTFY_TOKEN` | ntfy access token (optional) | `tk_...` |

### Report Modes

#### 1. **Daily Mode** (`mode: "daily"`)
- **Push Timing:** Scheduled (hourly by default)
- **Content:** All matching news for the day + new additions
- **Use Case:** Daily summaries, comprehensive overview

#### 2. **Current Mode** (`mode: "current"`)
- **Push Timing:** Scheduled (hourly by default)
- **Content:** Current trending list + new additions
- **Use Case:** Real-time trending topics

#### 3. **Incremental Mode** (`mode: "incremental"`)
- **Push Timing:** Only when new items appear
- **Content:** Only newly matched news
- **Use Case:** Avoid duplicate notifications

---

## 📝 Code Conventions

### Python Code Style

#### General Guidelines

1. **Follow PEP 8** for basic style
2. **Use type hints** where applicable
3. **Write docstrings** for public functions/classes
4. **Use descriptive variable names** (Chinese comments OK for Chinese domain logic)

#### Example from `mcp_server/server.py`:

```python
@mcp.tool
async def get_latest_news(
    platforms: Optional[List[str]] = None,
    limit: int = 50,
    include_url: bool = False
) -> str:
    """
    获取最新一批爬取的新闻数据，快速了解当前热点

    Args:
        platforms: 平台ID列表，如 ['zhihu', 'weibo', 'douyin']
        limit: 返回新闻数量限制
        include_url: 是否包含新闻链接

    Returns:
        str: JSON格式的新闻数据
    """
    tools = _get_tools()
    return tools['data'].get_latest_news(platforms, limit, include_url)
```

#### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Functions | `snake_case` | `get_latest_news()` |
| Classes | `PascalCase` | `DataQueryTools` |
| Constants | `UPPER_SNAKE_CASE` | `DEFAULT_LIMIT = 50` |
| Private methods | `_leading_underscore` | `_get_tools()` |
| Module names | `snake_case` | `data_service.py` |

### File Organization

#### MCP Server Structure

```
mcp_server/
├── __init__.py              # Package initialization
├── server.py                # Main entry point, tool registration
├── services/                # Business logic layer
│   ├── __init__.py
│   ├── *_service.py         # Service implementations
├── tools/                   # MCP tool implementations
│   ├── __init__.py
│   ├── *.py                 # Tool categories (data_query, analytics, etc.)
└── utils/                   # Utility functions
    ├── __init__.py
    └── *.py                 # Helpers (date_parser, validators, errors)
```

**Principle:** Separation of concerns - tools → services → utils

### Configuration Conventions

#### YAML Formatting

```yaml
# ✅ Good: Use comments to explain complex settings
weight:
  rank_weight: 0.6       # 排名权重 (60%)
  frequency_weight: 0.3   # 频次权重 (30%)
  hotness_weight: 0.1     # 热度权重 (10%)

# ✅ Good: Group related settings
notification:
  enable_notification: true
  message_batch_size: 4000
  webhooks:
    feishu_url: ""

# ❌ Bad: No secrets in config files
webhooks:
  telegram_bot_token: "123456:ABC-DEF"  # NEVER DO THIS
```

### Git Commit Conventions

**Format:** `<type>: <description>`

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `chore`: Maintenance tasks
- `refactor`: Code refactoring
- `test`: Test additions/changes

**Examples:**
```bash
git commit -m "feat: 添加微博平台支持"
git commit -m "fix: 修复邮件发送超时问题"
git commit -m "docs: 更新 MCP 配置文档"
git commit -m "chore: 更新依赖版本"
```

---

## 🧪 Testing & CI/CD

### Testing Strategy

**Current State:** No formal test suite (manual/integration testing)

**Recommended Testing Approach:**

```python
# Future: Add pytest tests
# tests/test_data_service.py

import pytest
from mcp_server.services.data_service import DataService

def test_get_latest_news():
    service = DataService()
    news = service.get_latest_news(limit=10)
    assert len(news) <= 10
    assert all('title' in item for item in news)
```

### GitHub Actions Workflows

#### 1. **crawler.yml** - News Crawler Automation

**Trigger:** Hourly (`0 * * * *`) + manual dispatch

**Steps:**
```yaml
1. Checkout repository
2. Setup Python 3.10
3. Install dependencies (pip install -r requirements.txt)
4. Verify config files exist
5. Run main.py with environment variables from GitHub Secrets
6. Auto-commit changes (git add, commit, push)
```

**Key Features:**
- Uses GitHub Secrets for webhooks/tokens
- Auto-commits generated JSON files
- Runs on `ubuntu-latest`

**Monitoring:**
- Check Actions tab for run status
- Review logs for errors
- Check commit history for updates

#### 2. **docker.yml** - Multi-Arch Docker Build

**Trigger:** Git tags (`v*`) + manual dispatch

**Steps:**
```yaml
1. Checkout repository
2. Set up QEMU for multi-arch
3. Set up Docker Buildx
4. Login to Docker Hub
5. Extract metadata (tags, labels)
6. Build and push multi-arch image (amd64, arm64)
```

**Registry:** Docker Hub (`wantcat/trendradar`)

**Tags:**
- `latest` - Latest stable release
- `v3.0.5` - Semantic version tags

### CI/CD Best Practices

1. **Always test locally before pushing**
   ```bash
   python main.py  # Test crawler
   python -m mcp_server.server  # Test MCP server
   ```

2. **Use GitHub Secrets for sensitive data**
   - Never commit tokens/webhooks to code
   - Add secrets at: Settings → Secrets and variables → Actions

3. **Monitor GitHub Actions**
   - Check Actions tab after pushing
   - Review logs for errors
   - Fix issues promptly

4. **Version tagging for Docker**
   ```bash
   git tag v3.0.6
   git push origin v3.0.6
   # Triggers docker.yml workflow
   ```

---

## 🤖 MCP Server Integration

### What is MCP?

**Model Context Protocol (MCP)** is a standard protocol for connecting AI assistants to external data sources and tools.

**TrendRadar MCP Server** provides:
- **13 specialized tools** for news analysis
- **STDIO transport** (for Cherry Studio, Claude Desktop)
- **HTTP transport** (for web clients)
- **Caching layer** for performance
- **Structured data access** to crawler output

### MCP Tools Overview

#### Data Query Tools (3 tools)

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `get_latest_news` | Get latest crawled news | `platforms`, `limit`, `include_url` |
| `get_news_by_date` | Get historical news by date | `date`, `platforms`, `limit` |
| `get_trending_topics` | Get frequency word statistics | `date_range`, `min_count` |

#### Search Tools (2 tools)

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `search_news` | Unified search (keyword/fuzzy/entity) | `query`, `search_type`, `date_range` |
| `search_related_news_history` | Find related news in history | `keyword`, `days_back`, `platforms` |

#### Analytics Tools (5 tools)

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `analyze_topic_trend` | Topic trend analysis (4 modes) | `keyword`, `analysis_type`, `date_range` |
| `analyze_data_insights` | Data insights (platform compare, etc.) | `analysis_type`, `date_range` |
| `analyze_sentiment` | Sentiment analysis | `keyword`, `date_range` |
| `find_similar_news` | Similar news finder | `reference_title`, `date_range` |
| `generate_summary_report` | Daily/weekly summaries | `report_type`, `date_range` |

#### System Tools (3 tools)

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `get_current_config` | View current configuration | None |
| `get_system_status` | System health check | None |
| `trigger_crawl` | Manual crawl trigger | `platforms` |

### Running MCP Server

#### STDIO Mode (Default)

**For:** Cherry Studio, Claude Desktop

```bash
# Start server
python -m mcp_server.server

# MCP client configuration
{
  "command": "python",
  "args": ["-m", "mcp_server.server"],
  "cwd": "/path/to/TrendRadar"
}
```

#### HTTP Mode

**For:** Web clients, testing

```bash
# Start HTTP server on port 8000
python -m mcp_server.server --transport http --port 8000

# Or use convenience script
./start-http.sh  # Linux/Mac
start-http.bat   # Windows
```

### MCP Client Setup

#### Cherry Studio (Zero-Coding Guide)

See `README-Cherry-Studio.md` for detailed instructions.

**Quick Steps:**
1. Download Cherry Studio from [https://cherry-ai.com/](https://cherry-ai.com/)
2. Install TrendRadar MCP server: `pip install trendradar-mcp`
3. Configure in Cherry Studio: Settings → MCP
4. Add server: `python -m mcp_server.server`
5. Test tools in Cherry Studio chat

#### Claude Desktop

**Config file:** `~/Library/Application Support/Claude/claude_desktop_config.json` (Mac)

```json
{
  "mcpServers": {
    "trendradar": {
      "command": "python",
      "args": ["-m", "mcp_server.server"],
      "cwd": "/path/to/TrendRadar"
    }
  }
}
```

### MCP Development Guidelines

#### Adding New Tools

1. **Choose appropriate category** (`tools/data_query.py`, `tools/analytics.py`, etc.)
2. **Implement tool class method**
3. **Register in `server.py`**

**Example:**

```python
# 1. In tools/data_query.py
class DataQueryTools:
    def get_news_count(self, date: str) -> str:
        """Get total news count for a date"""
        # Implementation...
        return json.dumps({"count": count})

# 2. In server.py
@mcp.tool
async def get_news_count(date: str) -> str:
    """获取指定日期的新闻总数"""
    tools = _get_tools()
    return tools['data'].get_news_count(date)
```

#### Tool Design Principles

1. **Return JSON strings** (MCP tools must return strings)
2. **Use type hints** for parameters
3. **Write comprehensive docstrings** (Chinese OK)
4. **Validate inputs** using `utils/validators.py`
5. **Handle errors gracefully** with custom exceptions from `utils/errors.py`
6. **Use caching** for expensive operations (`services/cache_service.py`)

---

## 🔧 Common Tasks

### Task 1: Add a New News Platform

**Steps:**

1. **Update `config/config.yaml`:**
   ```yaml
   platforms:
     - id: "new-platform-id"
       name: "新平台名称"
   ```

2. **Verify platform ID** matches NewsNow API
   - Check: https://github.com/ourongxing/newsnow

3. **Test crawler:**
   ```bash
   python main.py
   # Check output for new platform
   ```

4. **Commit changes:**
   ```bash
   git add config/config.yaml
   git commit -m "feat: 添加新平台支持"
   git push
   ```

### Task 2: Modify Notification Settings

**Scenario:** Change push time window

1. **Edit `config/config.yaml`:**
   ```yaml
   notification:
     push_window:
       enabled: true
       time_range:
         start: "09:00"
         end: "18:00"
       once_per_day: true
   ```

2. **Test locally:**
   ```bash
   python main.py
   # Verify push window logic
   ```

3. **Deploy:**
   - GitHub Actions: Push changes (auto-deploys)
   - Docker: Rebuild container
   - Local: Restart script

### Task 3: Add New MCP Tool

**Example:** Add tool to get news by platform

1. **Implement in `tools/data_query.py`:**
   ```python
   def get_news_by_platform(self, platform: str, limit: int = 50) -> str:
       """Get news from specific platform"""
       # Implementation
       return json.dumps(results)
   ```

2. **Register in `server.py`:**
   ```python
   @mcp.tool
   async def get_news_by_platform(platform: str, limit: int = 50) -> str:
       """获取指定平台的新闻"""
       tools = _get_tools()
       return tools['data'].get_news_by_platform(platform, limit)
   ```

3. **Test:**
   ```bash
   python -m mcp_server.server
   # Test in MCP client
   ```

4. **Commit:**
   ```bash
   git add mcp_server/
   git commit -m "feat: 添加按平台查询工具"
   git push
   ```

### Task 4: Update Dependencies

**Steps:**

1. **Check outdated packages:**
   ```bash
   pip list --outdated
   ```

2. **Update `requirements.txt` and `pyproject.toml`:**
   ```txt
   requests>=2.33.0,<3.0.0  # Updated version
   ```

3. **Test compatibility:**
   ```bash
   pip install -r requirements.txt
   python main.py
   python -m mcp_server.server
   ```

4. **Commit:**
   ```bash
   git add requirements.txt pyproject.toml
   git commit -m "chore: 更新依赖版本"
   git push
   ```

### Task 5: Customize Frequency Words

**Steps:**

1. **Edit `config/frequency_words.txt`:**
   ```
   华为
   特斯拉
   AI
   芯片
   自动驾驶
   # Add more keywords...
   ```

2. **Test crawler:**
   ```bash
   python main.py
   # Check matching logic
   ```

3. **Deploy:**
   - Commit to GitHub (auto-deploys to Actions)
   - Or update local file

### Task 6: Debug GitHub Actions

**Scenario:** Crawler workflow failing

1. **Check Actions tab:**
   - Navigate to: Actions → News Crawler
   - Click on failed run
   - Review logs

2. **Common issues:**
   - **Missing secrets:** Add at Settings → Secrets
   - **Config syntax error:** Validate YAML with `yamllint`
   - **API rate limit:** Check request_interval in config

3. **Manual test locally:**
   ```bash
   # Export secrets as env vars
   export TELEGRAM_BOT_TOKEN="your-token"
   export EMAIL_FROM="your-email"
   # ... other secrets

   # Run crawler
   python main.py
   ```

4. **Fix and redeploy:**
   ```bash
   git add .
   git commit -m "fix: 修复 GitHub Actions 配置"
   git push
   ```

---

## 🐛 Troubleshooting

### Common Issues

#### 1. Email Sending Fails

**Symptoms:**
- No email notifications
- SMTP timeout errors

**Solutions:**

✅ **Check SMTP credentials:**
```yaml
# config/config.yaml
webhooks:
  email_from: "your-email@gmail.com"
  email_password: "app-password"  # Use app-specific password
  email_to: "recipient@example.com"
```

✅ **Enable 2FA and create app password:**
- Gmail: https://support.google.com/accounts/answer/185833
- QQ Mail: https://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256

✅ **Verify SMTP server:**
```yaml
email_smtp_server: "smtp.gmail.com"  # Auto-detected if empty
email_smtp_port: "587"               # Auto-detected if empty
```

#### 2. Telegram Notifications Not Working

**Symptoms:**
- No Telegram messages
- Bot token errors

**Solutions:**

✅ **Verify bot token and chat ID:**
```bash
# Test bot token
curl https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getMe

# Get chat ID
curl https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
```

✅ **Check configuration:**
```yaml
webhooks:
  telegram_bot_token: "123456:ABC-DEF..."
  telegram_chat_id: "@channel_name"  # or numeric ID
```

#### 3. MCP Server Won't Start

**Symptoms:**
- `ModuleNotFoundError: No module named 'fastmcp'`
- Server crashes on startup

**Solutions:**

✅ **Install dependencies:**
```bash
pip install -r requirements.txt
# or
pip install fastmcp>=2.12.0
```

✅ **Check Python version:**
```bash
python --version  # Should be 3.10+
```

✅ **Verify project structure:**
```bash
ls mcp_server/
# Should contain: server.py, tools/, services/, utils/
```

#### 4. Docker Container Exits Immediately

**Symptoms:**
- Container starts then stops
- No logs in `docker logs`

**Solutions:**

✅ **Check Docker logs:**
```bash
docker logs trendradar
```

✅ **Verify config files exist:**
```bash
docker exec -it trendradar ls -la /app/config/
# Should show: config.yaml, frequency_words.txt
```

✅ **Check cron configuration:**
```bash
docker exec -it trendradar cat /etc/crontabs/root
```

#### 5. GitHub Actions Not Running

**Symptoms:**
- No hourly runs
- Workflow disabled

**Solutions:**

✅ **Enable Actions:**
- Navigate to: Actions tab
- Click "I understand my workflows, go ahead and enable them"

✅ **Check workflow schedule:**
```yaml
# .github/workflows/crawler.yml
on:
  schedule:
    - cron: '0 * * * *'  # Runs hourly
```

✅ **Manual trigger:**
- Actions → News Crawler → Run workflow

#### 6. Frequency Words Not Matching

**Symptoms:**
- Keywords in `frequency_words.txt` not triggering notifications
- No matching news despite relevant content

**Solutions:**

✅ **Check file encoding:**
```bash
file config/frequency_words.txt
# Should be: UTF-8 Unicode text
```

✅ **Verify format:**
```
华为
特斯拉
AI
# One keyword per line, no extra spaces
```

✅ **Test matching logic:**
```bash
python main.py
# Check console output for matched keywords
```

### Debug Mode

**Enable verbose logging:**

```python
# Add to main.py (temporary)
import logging
logging.basicConfig(level=logging.DEBUG)
```

**Run with debug output:**

```bash
python main.py > debug.log 2>&1
# Review debug.log for detailed traces
```

### Getting Help

1. **Check existing documentation:**
   - `readme.md` - Main documentation
   - `README-MCP-FAQ.md` - MCP-specific FAQ
   - `README-Cherry-Studio.md` - Cherry Studio guide

2. **Search GitHub Issues:**
   - https://github.com/sansan0/TrendRadar/issues

3. **Create new issue:**
   - Use templates: Bug Report, Feature Request, Config Help
   - Provide: OS, Python version, error logs, config (redacted)

4. **Community support:**
   - LinuxDo Community: https://linux.do/
   - GitHub Discussions (if enabled)

---

## 📚 Additional Resources

### Official Documentation

- **Main README:** [readme.md](readme.md)
- **MCP FAQ:** [README-MCP-FAQ.md](README-MCP-FAQ.md)
- **Cherry Studio Guide:** [README-Cherry-Studio.md](README-Cherry-Studio.md)
- **GitHub Pages:** https://sansan0.github.io/TrendRadar

### External Resources

- **Model Context Protocol:** https://modelcontextprotocol.io/
- **FastMCP Framework:** https://github.com/jlowin/fastmcp
- **NewsNow API:** https://github.com/ourongxing/newsnow
- **Docker Hub:** https://hub.docker.com/r/wantcat/trendradar

### Related Projects

- **Cherry Studio:** https://cherry-ai.com/ (MCP client)
- **Claude Desktop:** https://claude.ai/download (Anthropic's desktop app)

---

## 📄 License

This project is licensed under the **GPL-3.0 License**. See [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgments

- **NewsNow API:** https://github.com/ourongxing/newsnow (data source)
- **Contributors:** See [致谢名单](readme.md) in main README
- **Community:** LinuxDo, 小众软件, 阮一峰周刊

---

**Last Updated:** 2025-11-15
**Maintained By:** AI Assistants + TrendRadar Community
**Feedback:** https://github.com/sansan0/TrendRadar/issues
