# 招聘Agent - 自动化候选人搜索和管理系统

[![Python Version](https://img.shields.io/badge/python-3.11+-blue.svg)](https://python.org)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Tests](https://img.shields.io/badge/tests-passing-brightgreen.svg)](tests/)

一个强大的自动化招聘Agent，能够在LinkedIn、Facebook、X (Twitter)等社交媒体平台上智能搜索合适的候选人，并将相关人员信息自动整理汇编到Notion数据库中，支持定期更新和多种部署方式。

## ✨ 主要特性

- **多平台数据抓取**: 支持LinkedIn、Facebook、X (Twitter)三大主流社交媒体平台
- **智能数据处理**: 自动清洗、标准化和结构化候选人信息
- **Notion集成**: 无缝集成Notion API，自动创建和更新候选人数据库
- **定时任务**: 支持多种调度方式，实现数据的定期自动更新
- **模块化设计**: 清晰的架构设计，易于扩展和维护
- **命令行界面**: 友好的CLI工具，支持一键运行和配置
- **完整测试**: 包含单元测试和集成测试，确保代码质量
- **合规性考虑**: 严格遵守各平台使用条款和数据隐私规定

## 🚀 快速开始

### 环境要求

- Python 3.11+
- Chrome浏览器 (用于LinkedIn爬取)
- 有效的API密钥 (Notion必需，其他平台可选)

### 安装

1. 克隆项目
```bash
git clone https://github.com/your-username/recruitment-agent.git
cd recruitment-agent
```

2. 安装依赖
```bash
pip install -r requirements.txt
```

3. 配置API密钥
```bash
python main.py config
```

或者手动创建 `.env` 文件：
```env
# Notion API配置 (必需)
NOTION_API_KEY=your_notion_api_key_here
NOTION_DATABASE_ID=your_notion_database_id_here

# Twitter API配置 (可选)
TWITTER_BEARER_TOKEN=your_twitter_bearer_token_here

# Facebook API配置 (可选)
FACEBOOK_ACCESS_TOKEN=your_facebook_access_token_here

# LinkedIn配置 (可选，需谨慎使用)
LINKEDIN_USERNAME=your_linkedin_username_here
LINKEDIN_PASSWORD=your_linkedin_password_here

# 搜索配置
SEARCH_KEYWORDS=software engineer,data scientist,product manager
SEARCH_LOCATIONS=Beijing,Shanghai,Shenzhen
```

### 基本使用

1. **运行一次Agent**
```bash
python main.py run
```

2. **指定搜索参数**
```bash
python main.py run --keywords "python,ai" --locations "Beijing,Shanghai" --max-results 30
```

3. **只搜索特定平台**
```bash
python main.py run --platform linkedin
```

4. **设置定时任务**
```bash
# 每天9点运行
python main.py schedule --cron "0 9 * * *"

# 每2小时运行一次
python main.py schedule --interval 120
```

5. **查看状态**
```bash
python main.py status
```

6. **搜索现有候选人**
```bash
python main.py search "软件工程师"
```

## 📋 详细功能

### 数据抓取模块

Agent支持从以下平台抓取候选人信息：

**LinkedIn**
- 基于关键词和地点的候选人搜索
- 个人资料详细信息获取
- 技能、经验、教育背景提取
- 反爬机制应对和合规性考虑

**Facebook**
- 通过Graph API搜索公共页面和群组
- 用户基本信息和工作经历获取
- 严格遵守Facebook平台政策

**X (Twitter)**
- 基于Twitter API v2的用户搜索
- 用户资料和最近推文分析
- 支持Bearer Token和OAuth认证

### 数据处理

- **清洗**: 自动移除HTML标签、特殊字符和重复数据
- **标准化**: 统一数据格式，包括日期、姓名等
- **结构化**: 映射到预定义的候选人数据模型
- **去重**: 智能识别和合并重复候选人信息

### Notion集成

- **自动数据库创建**: 如果不存在则自动创建标准化的候选人数据库
- **智能更新**: 检测现有候选人并更新信息而非重复创建
- **丰富的属性支持**: 包括文本、URL、邮箱、电话、多选等多种属性类型
- **批量操作**: 支持批量创建和更新，提高效率

### 调度系统

支持多种调度方式：

- **简单调度**: 基于`schedule`库的轻量级定时任务
- **高级调度**: 基于`APScheduler`的企业级调度功能
- **Cron表达式**: 支持标准Cron语法的复杂时间规则
- **间隔调度**: 支持固定间隔的重复执行

## 🏗️ 架构设计

```
recruitment_agent/
├── main.py                 # 主入口文件
├── recruitment_agent.py    # 核心Agent类
├── config.py              # 配置管理
├── logger.py              # 日志模块
├── models.py              # 数据模型
├── scheduler.py           # 调度模块
├── notion_integration.py  # Notion集成
├── scrapers/              # 爬虫模块
│   ├── __init__.py
│   ├── base_scraper.py    # 基础爬虫类
│   ├── linkedin_scraper.py
│   ├── facebook_scraper.py
│   └── twitter_scraper.py
├── tests/                 # 测试模块
│   ├── test_models.py
│   ├── test_config.py
│   └── test_scrapers.py
├── requirements.txt       # 依赖列表
├── .env.example          # 配置示例
├── demo.py               # 演示脚本
└── run_tests.py          # 测试运行器
```

## 🔧 配置说明

### 必需配置

- `NOTION_API_KEY`: Notion集成令牌
- `NOTION_DATABASE_ID`: 目标Notion数据库ID

### 可选配置

- `TWITTER_BEARER_TOKEN`: Twitter API Bearer Token
- `FACEBOOK_ACCESS_TOKEN`: Facebook Graph API访问令牌
- `LINKEDIN_USERNAME/PASSWORD`: LinkedIn登录凭据 (谨慎使用)
- `SEARCH_KEYWORDS`: 搜索关键词 (逗号分隔)
- `SEARCH_LOCATIONS`: 搜索地点 (逗号分隔)
- `MAX_RESULTS_PER_PLATFORM`: 每个平台的最大结果数
- `REQUEST_DELAY`: 请求间隔时间 (秒)
- `LOG_LEVEL`: 日志级别 (DEBUG/INFO/WARNING/ERROR)

## 🧪 测试

运行所有测试：
```bash
python run_tests.py
```

运行特定测试：
```bash
python run_tests.py test_models.TestCandidate.test_candidate_creation
```

查看演示：
```bash
python demo.py
```

## 📊 数据模型

候选人数据包含以下字段：

| 字段名 | 类型 | 描述 |
|--------|------|------|
| 姓名 | Text | 候选人全名 |
| 职位 | Text | 当前职位 |
| 公司 | Text | 当前公司 |
| 地点 | Text | 所在城市/地区 |
| 领英主页 | URL | LinkedIn个人主页链接 |
| Facebook主页 | URL | Facebook个人主页链接 |
| X主页 | URL | X (Twitter)个人主页链接 |
| 联系邮箱 | Email | 联系邮箱地址 |
| 联系电话 | Phone | 联系电话号码 |
| 技能 | Multi-select | 技能标签列表 |
| 工作经验 | Text | 工作经验描述 |
| 教育背景 | Text | 教育背景信息 |
| 来源平台 | Select | 数据来源平台 |
| 最后更新时间 | Date | 记录更新时间 |
| 备注 | Text | 其他备注信息 |

## ⚠️ 重要说明

### 合规性和法律考虑

1. **平台政策遵守**: 严格遵守LinkedIn、Facebook、X等平台的使用条款和API政策
2. **数据隐私**: 只抓取公开可用的信息，尊重个人隐私权
3. **使用限制**: 避免过度频繁的请求，遵守各平台的速率限制
4. **商业用途**: 如用于商业目的，请确保获得适当的授权和许可

### 技术限制

1. **LinkedIn限制**: LinkedIn有严格的反爬机制，建议使用官方API或第三方服务
2. **API配额**: 各平台API都有使用配额限制，需要合理规划使用频率
3. **网络环境**: 某些功能可能需要稳定的网络环境和特定的地理位置

## 🤝 贡献指南

欢迎贡献代码！请遵循以下步骤：

1. Fork本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启Pull Request

## 📄 许可证

本项目采用MIT许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 🆘 支持

如果您遇到问题或有疑问，请：

1. 查看 [Issues](https://github.com/your-username/recruitment-agent/issues) 页面
2. 创建新的Issue描述您的问题
3. 参考项目文档和示例代码

## 🔮 未来计划

- [ ] 支持更多社交媒体平台 (Instagram, TikTok等)
- [ ] 增加AI驱动的候选人匹配和评分
- [ ] 开发Web界面和仪表板
- [ ] 集成更多HR系统和ATS平台
- [ ] 增加候选人沟通和跟进功能
- [ ] 支持多语言和国际化

---

**作者**: Manus AI  
**版本**: 1.0.0  
**最后更新**: 2025-07-27

