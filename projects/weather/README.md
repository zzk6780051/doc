# 天气通知脚本项目

## 项目概述

天气通知脚本是一个多平台自动化天气信息推送工具，能够定时获取多个城市的天气信息，并通过多种渠道（Telegram、企业微信、GitHub）发送通知。项目设计为可在青龙面板和GitHub Actions中运行，满足不同环境下的自动化需求。
可与我的另一个项目**实时通知中心**对接

## 主要功能

- 🌤️ **多城市天气查询**：支持同时查询多个城市的实时天气、空气质量、3天预报
- 📱 **多渠道通知**：支持Telegram、企业微信、GitHub通知
- ⚡ **多平台部署**：支持青龙面板和GitHub Actions两种运行环境
- 🔒 **安全配置**：所有敏感信息通过环境变量管理
- 📊 **数据持久化**：可将天气信息更新到GitHub仓库

## 技术栈

- **编程语言**: Python 3
- **核心库**: requests, json, base64, os, datetime
- **API集成**: 和风天气API、Telegram Bot API、企业微信机器人、GitHub API
- **部署平台**: 青龙面板、GitHub Actions

## 项目结构

```
scripts/
├── weather.py                 # 主脚本文件
├── .github/
│   └── workflows/
│       └── weather.yml        # GitHub Actions工作流
└── 其他/
```

## 适用场景

- 个人日常天气提醒
- 团队天气信息共享
- 多平台消息同步
- 自动化运维监控

#