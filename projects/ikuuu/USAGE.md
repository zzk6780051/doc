# iKuuu 自动签到使用说明

## 快速开始

### 方式一：青龙面板部署（推荐）

#### 1. 添加环境变量

在青龙面板的「环境变量」中添加以下变量：

**必需变量：**
```bash
IKUUU_ACCOUNTS=[{"name":"账号1","email":"user1@example.com","passwd":"password1"},{"name":"账号2","email":"user2@example.com","passwd":"password2"}]
```

**可选变量：**
```bash
# 基础配置
IKUUU_DOMAIN=ikuuu.eu
IKUUU_MAX_RETRY=3
IKUUU_ENABLE_HISTORY=true

# Telegram 通知
IKUUU_TG_BOT_TOKEN=your_bot_token
IKUUU_TG_CHAT_ID=your_chat_id

# 企业微信机器人
IKUUU_QYWX_WEBHOOK=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=your_key

# 企业微信应用
IKUUU_WECOM_CORPID=your_corpid
IKUUU_WECOM_AGENT_ID=your_agentid
IKUUU_WECOM_SECRET=your_secret
IKUUU_WECOM_TO_USER=@all

# GitHub 通知
IKUUU_GITHUB_TOKEN=your_github_token
IKUUU_GITHUB_REPO=username/repo
IKUUU_GITHUB_BRANCH=main
```

#### 2. 添加脚本

1. 在青龙面板「脚本管理」中创建新文件
2. 文件名：`ikuuu.js`
3. 将项目中的 `ikuuu.js` 内容复制粘贴
4. 保存文件

#### 3. 设置定时任务

在「定时任务」中添加：
```bash
# 每天上午8点执行
0 8 * * * task ikuuu.js
```

### 方式二：GitHub Actions 部署

#### 1. Fork 仓库

访问 [项目仓库](https://github.com/zzk6780051/scripts) 并 Fork 到自己的账户。

#### 2. 配置 Secrets

在仓库的 `Settings > Secrets and variables > Actions` 中添加：

**必需 Secrets：**
- `IKUUU_ACCOUNTS` - 账户列表（JSON格式）

**可选 Secrets：**
- `IKUUU_TG_BOT_TOKEN` - Telegram Bot Token
- `IKUUU_TG_CHAT_ID` - Telegram Chat ID
- `IKUUU_QYWX_WEBHOOK` - 企业微信 Webhook
- `IKUUU_GITHUB_TOKEN` - GitHub Token
- `IKUUU_GITHUB_REPO` - GitHub 仓库(owner/repo)

#### 3. 启用 Actions

在仓库的 Actions 页面启用工作流程，脚本将按计划自动运行。

## 环境变量详解

### 账户配置
```json
IKUUU_ACCOUNTS=[
  {
    "name": "账号名称",
    "email": "邮箱地址",
    "passwd": "密码"
  }
]
```

### 通知配置示例

#### Telegram 配置
1. 创建 Bot：联系 @BotFather
2. 获取 Chat ID：联系 @userinfobot
3. 设置环境变量：
```bash
IKUUU_TG_BOT_TOKEN=123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11
IKUUU_TG_CHAT_ID=123456789
```

#### 企业微信配置
1. 群聊中添加机器人，获取 Webhook URL
2. 设置环境变量：
```bash
IKUUU_QYWX_WEBHOOK=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

#### GitHub 通知配置
1. 创建 GitHub Personal Access Token
2. 设置环境变量：
```bash
IKUUU_GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
IKUUU_GITHUB_REPO=username/repo
```

## 故障排除

### 常见问题

1. **签到失败**
   - 检查账户密码是否正确
   - 确认 iKuuu 域名可访问
   - 查看日志获取详细错误信息

2. **通知未发送**
   - 检查通知配置是否正确
   - 确认网络连接正常
   - 查看各平台 API 限制

3. **脚本执行错误**
   - 确认 Node.js 版本 ≥ 18
   - 检查环境变量格式
   - 查看系统权限设置

### 日志查看

在青龙面板中查看任务日志，或在 GitHub Actions 中查看运行日志。

## 更新维护

定期检查项目更新，及时获取新功能和修复。

#
