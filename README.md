# iKuuu 自动签到项目

![GitHub](https://img.shields.io/badge/Node.js-18+-green?logo=nodedotjs)
![GitHub](https://img.shields.io/badge/License-MIT-blue)
![GitHub](https://img.shields.io/badge/Platform-QingLong%20%7C%20GitHub%20Actions%20%7C%20Docker-orange)
![GitHub](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)

## 🎯 项目简介

iKuuu 自动签到项目是一个专为 [iKuuu VPN](https://ikuuu.eu) 服务设计的自动化签到工具。通过自动化每日签到，帮助用户稳定获取免费流量，支持多账户管理、智能重试、多种通知方式等功能。

> 🔗 **相关项目**: 本项目可与 [实时通知中心](https://github.com/your-username/notification-center) 无缝对接，实现统一的消息管理。

## ✨ 核心特性

### 🚀 自动化签到
- **多账户支持**: 同时管理多个 iKuuu 账户
- **智能重试机制**: 内置 3 级重试，提高签到成功率
- **重复签到处理**: 自动识别重复签到并标记为成功
- **域名自动切换**: 当主域名不可用时自动切换到备用域名

### 📊 数据统计与分析
- **完整历史记录**: 记录每次签到的详细结果
- **月度统计报表**: 自动生成成功率、签到次数等统计
- **实时状态监控**: 监控各节点健康状态
- **可视化报告**: 生成易于阅读的统计报告

### 🔔 多渠道通知
- **Telegram Bot**
- **企业微信**
- **Bark (iOS)**
- **钉钉机器人**
- **我的网站集成**

### 🛡️ 安全与稳定
- **信息脱敏处理**: 自动隐藏敏感信息，保护隐私安全
- **完善错误处理**: 多层异常捕获和恢复机制
- **连接稳定性**: 自动重连和故障转移
- **代理支持**: 支持通过代理服务器访问

### 🌍 高级功能
- **多语言支持**: 支持中文和英文界面
- **健康检查**: 定期检查系统健康状态
- **节点状态监控**: 实时监控各服务节点可用性
- **历史记录清理**: 自动清理过期历史数据
- **命令行工具**: 支持多种运行模式和参数

## 🏗️ 技术架构

### 技术栈
- **运行时**: Node.js 18+
- **HTTP 客户端**: 原生 fetch API
- **配置管理**: 环境变量 + JSON 配置文件
- **数据存储**: 本地文件系统
- **部署平台**: 青龙面板 / GitHub Actions

### 系统要求
- Node.js 18.0 或更高版本
- 网络访问权限（访问 iKuuu 服务和外网）
- 文件系统写入权限（存储历史记录）

## 🚀 快速开始

### 方式一：青龙面板部署（推荐）

#### 1. 添加环境变量
在青龙面板的「环境变量」页面添加：

```bash
# 必需配置
IKUUU_ACCOUNTS=[{"name":"我的账号","email":"your_email@example.com","passwd":"your_password"}]

# 可选配置
IKUUU_DOMAIN=ikuuu.boo
IKUUU_MAX_RETRY=3
IKUUU_ENABLE_HISTORY=true
```

#### 2. 添加脚本文件
1. 进入「脚本管理」
2. 创建新文件：`ikuuu.js`
3. 复制项目中的脚本内容
4. 保存文件

#### 3. 设置定时任务
在「定时任务」中添加：
```bash
# 每天上午8点执行
0 8 * * * task ikuuu.js

# 或每天多个时间点执行
0 8,12,18 * * * task ikuuu.js
```

### 方式二：GitHub Actions 部署

#### 1. Fork 项目仓库
访问 [项目仓库](https://github.com/zzk1314/scripts) 并 Fork 到自己的账户。

#### 2. 配置 Secrets
在仓库的 `Settings > Secrets and variables > Actions` 中添加：

**必需 Secrets:**
- `IKUUU_ACCOUNTS` - 账户列表（JSON 格式）

**可选 Secrets:**
- `IKUUU_TG_BOT_TOKEN` - Telegram Bot Token
- `IKUUU_TG_CHAT_ID` - Telegram Chat ID
- `IKUUU_QYWX_WEBHOOK` - 企业微信 Webhook
- `IKUUU_GH_TOKEN` - GitHub Token
- `IKUUU_GH_REPO` - GitHub 仓库

#### 3. 启用工作流
在 Actions 页面启用工作流，脚本将按计划自动运行。


## ⚙️ 配置详解

### 账户配置
支持多账户配置，格式如下：

```json
[
  {
    "name": "账号1",
    "email": "user1@example.com",
    "passwd": "password1"
  },
  {
    "name": "账号2", 
    "email": "user2@example.com",
    "passwd": "password2"
  }
]
```

### 通知配置示例

#### Telegram 配置
1. 创建 Bot: 联系 [@BotFather](https://t.me/BotFather)
2. 获取 Chat ID: 联系 [@userinfobot](https://t.me/userinfobot)
3. 配置环境变量:
```bash
IKUUU_TG_BOT_TOKEN=123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11
IKUUU_TG_CHAT_ID=123456789
```

#### 企业微信配置
1. 群聊中添加机器人，获取 Webhook URL
2. 配置环境变量:
```bash
IKUUU_QYWX_WEBHOOK=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=your_key
```

#### GitHub 通知配置
1. 创建 [GitHub Personal Access Token](https://github.com/settings/tokens)
2. 配置环境变量:
```bash
IKUUU_GH_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
IKUUU_GH_REPO=username/repo
```

### 高级配置选项

```bash
# 域名自动切换
IKUUU_AUTO_SWITCH_DOMAIN=true
IKUUU_BACKUP_DOMAINS=["ikuuu.de","ikuuu.boo"]

# 节点状态检查
IKUUU_ENABLE_NODE_STATUS=true

# 代理设置
IKUUU_PROXY_URL=http://proxy.example.com:8080

# 多语言支持
IKUUU_LANGUAGE=zh-CN  # 或 en

# 超时设置
IKUUU_TIMEOUT=30000
```

## 📖 使用指南

### 命令行参数

```bash
# 正常执行签到
node ikuuu.js

# 健康检查模式
node ikuuu.js --health

# 清理历史记录
node ikuuu.js --cleanup

# 显示当前配置
node ikuuu.js --config

# 只检查不签到
node ikuuu.js --check

# 显示帮助信息
node ikuuu.js --help
```

### 日志查看

**青龙面板**:
- 进入「定时任务」页面
- 点击任务右侧的「日志」按钮
- 查看详细的执行日志

**GitHub Actions**:
- 进入仓库的 Actions 页面
- 点击对应的 workflow run
- 查看执行日志


## 🐛 故障排除

### 常见问题

#### 1. 签到失败
**可能原因**:
- 账户密码错误
- 网络连接问题
- iKuuu 服务暂时不可用

**解决方案**:
- 检查账户密码是否正确
- 确认网络连接正常
- 查看详细错误日志

#### 2. 通知未发送
**可能原因**:
- 通知配置错误
- API 令牌失效
- 网络访问限制

**解决方案**:
- 验证各平台配置信息
- 检查 API 令牌是否有效
- 确认网络可以访问外网

#### 3. 脚本执行错误
**可能原因**:
- Node.js 版本过低
- 环境变量格式错误
- 文件权限问题

**解决方案**:
- 升级 Node.js 到 18+ 版本
- 检查环境变量 JSON 格式
- 确认脚本有执行权限

### 调试模式

启用调试模式获取更详细的日志：

```bash
export IKUUU_DEBUG=true
node ikuuu.js
```

## 🔄 更新维护

### 版本更新
定期检查项目更新，获取新功能和修复：

```bash
# 青龙面板更新
# 1. 重新下载脚本文件
# 2. 替换现有脚本本

### 数据备份
建议定期备份历史记录目录：

```

# 备份历史数据
```
tar -czf ikuuu_history_backup_$(date +%Y%m%d).tar.gz history/
```

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request 来改进项目！

### 开发环境设置
1. Fork 项目仓库
2. 克隆到本地: `git clone https://github.com/your-username/scripts.git`
3. 安装依赖: `npm install`
4. 创建测试配置
5. 开始开发


## 📄 许可证

本项目采用 MIT 许可证 

## 🙏 致谢

感谢所有贡献者和用户的支持！特别感谢：

- iKuuu 服务提供免费的 VPN 服务
- 青龙面板团队提供优秀的定时任务平台
- 各消息平台提供的 API 服务

---

**如果这个项目对你有帮助，请给个 ⭐️ 支持一下！**

> 📖 **详细文档**: [查看开发文档](DEVELOPMENT.md)

