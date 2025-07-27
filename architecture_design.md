# Agent 架构设计

## 1. 总体架构

本 Agent 将采用模块化设计，主要分为以下几个核心组件：

1.  **数据抓取模块 (Scraping Module)**：负责从领英、Facebook、X等社交媒体平台抓取目标候选人信息。
2.  **数据处理模块 (Processing Module)**：对抓取到的原始数据进行清洗、标准化和结构化处理。
3.  **Notion 集成模块 (Notion Integration Module)**：通过 Notion API 将处理后的数据汇编并更新到 Notion 数据库。
4.  **调度模块 (Scheduler Module)**：负责Agent的定时运行，触发数据抓取和Notion更新。
5.  **配置管理模块 (Configuration Module)**：管理Agent的各项配置，如API密钥、Notion数据库ID、搜索关键词等。
6.  **日志模块 (Logging Module)**：记录Agent的运行状态、错误信息等，便于问题排查和监控。

## 2. 数据流

数据流向如下：

社交媒体平台 (LinkedIn, Facebook, X) --> 数据抓取模块 --> 数据处理模块 --> Notion 集成模块 --> Notion 数据库

调度模块将定时触发整个流程。

## 3. 模块详细设计

### 3.1 数据抓取模块

考虑到各平台反爬机制和API限制，本模块将采取以下策略：

*   **领英**: 优先考虑使用第三方服务或库，如 `linkedin-api` (非官方) 或通过模拟浏览器行为（如 `Selenium` + `undetected_chromedriver`）进行有限度的抓取。对于大规模或高频率需求，建议用户考虑购买合法数据服务或API。初始阶段将探索基于 `requests` 和 `BeautifulSoup` 的轻量级爬取，若受限则转向模拟浏览器。
*   **Facebook**: 主要依赖 Facebook Graph API。需要用户提供有效的 Access Token，并确保遵守 Facebook 的平台政策。对于公共页面或群组信息，可能需要结合 `requests` 进行爬取，但需注意合规性。
*   **X (Twitter)**: 优先使用 `tweepy` (Twitter API v2)。需要用户提供 Bearer Token 或 OAuth 1.0a 凭证。对于非API可获取的数据，可能需要探索其他开源库或模拟浏览器，但同样需注意平台政策。

**关键挑战**: 各平台的反爬机制和API限制，需要灵活应对和持续维护。

### 3.2 数据处理模块

该模块负责将从不同平台抓取到的非结构化或半结构化数据转换为统一的结构化格式，以便存储到 Notion 数据库。

*   **清洗**: 移除HTML标签、特殊字符、重复数据等。
*   **标准化**: 统一数据格式，例如日期格式、姓名格式等。
*   **结构化**: 将数据映射到预定义的数据模型中，例如：
    *   `name`: 姓名
    *   `title`: 职位
    *   `company`: 公司
    *   `location`: 地点
    *   `profile_url`: 个人主页URL
    *   `contact_info`: 联系方式 (邮箱、电话等，如果可获取且合规)
    *   `skills`: 技能
    *   `experience`: 工作经验
    *   `education`: 教育背景
    *   `source_platform`: 来源平台 (LinkedIn, Facebook, X)
    *   `last_updated`: 最后更新时间

### 3.3 Notion 集成模块

该模块将使用 Notion Python SDK (`notion-sdk-py`) 与 Notion API 进行交互，实现数据的写入和更新。

*   **认证**: 使用 Notion API Key 进行认证。
*   **数据库操作**: 
    *   **创建页面**: 将新的候选人信息作为 Notion 数据库中的新页面创建。
    *   **更新页面**: 根据候选人ID或URL判断是否已存在，若存在则更新其信息。
    *   **查询数据库**: 用于检查候选人是否存在或获取现有数据。
*   **错误处理**: 处理API请求失败、数据格式不匹配等异常情况。

### 3.4 调度模块

调度模块将负责 Agent 的定时运行。可以考虑以下几种实现方式：

*   **本地定时任务**: 使用 `APScheduler` 或 `schedule` 库在本地运行 Python 脚本，适用于 Agent 部署在用户本地服务器的情况。
*   **云函数/Serverless**: 将 Agent 部署为云函数（如 AWS Lambda, Google Cloud Functions），通过云服务商的定时触发器来运行，适用于无服务器部署。
*   **GitHub Actions**: 对于轻量级任务，可以配置 GitHub Actions 定时触发工作流，运行 Agent 脚本。

### 3.5 配置管理模块

使用 `.env` 文件或配置文件 (如 `config.ini`, `config.json`) 来管理敏感信息 (API Key) 和可配置参数 (搜索关键词、Notion 数据库 ID)。

### 3.6 日志模块

使用 Python 内置的 `logging` 模块记录 Agent 的运行日志，包括信息、警告、错误等不同级别的日志，便于调试和监控。

## 4. 数据模型 (Notion 数据库结构)

Notion 数据库将包含以下属性 (Properties)：

| 属性名称 (Property Name) | 类型 (Type) | 描述 (Description) | 示例 (Example) |
| :----------------------- | :---------- | :----------------- | :------------- |
| `姓名`                   | Text        | 候选人全名         | 张三           |
| `职位`                   | Text        | 候选人当前职位     | 软件工程师     |
| `公司`                   | Text        | 候选人当前公司     | ABC科技        |
| `地点`                   | Text        | 候选人所在城市/地区 | 北京           |
| `领英主页`               | URL         | 领英个人主页链接   | `https://www.linkedin.com/in/xxx` |
| `Facebook主页`           | URL         | Facebook个人主页链接 | `https://www.facebook.com/xxx` |
| `X主页`                  | URL         | X (Twitter) 个人主页链接 | `https://twitter.com/xxx` |
| `联系邮箱`               | Email       | 候选人联系邮箱     | `xxx@example.com` |
| `联系电话`               | Phone       | 候选人联系电话     | `+86 138xxxx` |
| `技能`                   | Multi-select | 候选人技能标签     | Python, AI, ML |
| `工作经验`               | Text        | 候选人工作经验摘要 | 5年软件开发经验 |
| `教育背景`               | Text        | 候选人教育背景摘要 | 清华大学计算机科学硕士 |
| `来源平台`               | Select      | 数据来源平台       | LinkedIn       |
| `最后更新时间`           | Date        | 记录更新时间       | 2025-07-27     |
| `备注`                   | Text        | 其他备注信息       |                |

## 5. 技术栈

*   **编程语言**: Python 3.x
*   **数据抓取**: `requests`, `BeautifulSoup`, `Selenium` (按需), `tweepy` (Twitter API), Facebook Graph API (通过 `requests` 或官方 SDK)
*   **Notion 集成**: `notion-sdk-py`
*   **定时任务**: `APScheduler` 或 `schedule` (本地部署), GitHub Actions (云端)
*   **配置管理**: `python-dotenv`
*   **版本控制**: Git
*   **项目托管**: GitHub

## 6. 部署方案

初期考虑本地部署，方便开发和调试。成熟后可考虑部署到云服务器或作为云函数运行，以实现更稳定的定期更新。

## 7. 安全与合规性

*   **API Key 管理**: 敏感信息 (API Key) 绝不硬编码，通过环境变量或配置文件加载。
*   **平台政策**: 严格遵守领英、Facebook、X和Notion的平台使用条款和API政策，避免非法抓取和滥用数据。
*   **数据隐私**: 确保只抓取公开可用的信息，并尊重个人隐私。对于任何可能涉及个人敏感信息的数据，需获得明确授权。
*   **错误处理与重试**: 针对网络请求、API限制等可能出现的错误，实现健壮的错误处理和重试机制，避免因瞬时问题导致程序中断。

## 8. 后续迭代

*   **高级搜索**: 增加更复杂的搜索条件和过滤功能。
*   **数据去重与合并**: 优化不同平台数据的去重和合并逻辑。
*   **通知机制**: 增加新候选人或更新的通知功能 (如邮件、Slack等)。
*   **Web界面**: 开发一个简单的Web界面，方便用户配置和查看运行状态。


