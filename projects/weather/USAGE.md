# 天气通知脚本使用说明

## 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/zzk6780051/scripts.git
cd scripts
```

### 2. 配置环境变量

根据您的需求设置以下环境变量：

#### 必需配置
```bash
# 天气API配置（需要申请和风天气API）
WEATHER_API_KEY="你的和风天气API_KEY"
WEATHER_API_HOST="你的和风天气API主机"
WEATHER_CITIES='{"城市1": "城市ID1", "城市2": "城市ID2"}'
```

#### 可选配置
```bash
# Telegram机器人配置
TELEGRAM_BOT_TOKEN="你的Telegram机器人Token"
TELEGRAM_CHAT_ID="你的Telegram聊天ID"

# 企业微信机器人配置
WECHAT_WORK_WEBHOOK="你的企业微信Webhook地址"

# 通知网站配置（用于更新通知到仓库）
NOTIFY_GITHUB_TOKEN="你的GitHub Token"
NOTIFY_REPO_OWNER="仓库所有者"
NOTIFY_REPO_NAME="仓库名称"
NOTIFY_FILE_PATH="data.json"
```

## 运行方式

### 方式一：本地运行

#### 1.环境要求

- Python 3.6+
- 必要的Python包：`requests`



#### 2.安装依赖

```bash
pip install requests
```
#### 3.运行
```bash
# 普通运行（控制台输出）
python3 weather.py

# 通知模式运行（发送所有通知）
python3 weather.py ql
```

### 方式二：青龙面板运行

1. 在青龙面板中添加环境变量
2. 创建脚本文件，内容为本项目的`weather.py`
3. 添加定时任务，命令为：`python3 /ql/scripts/weather.py ql`
4. 设置执行时间（如：`0 8 * * *` 每天8点执行）

### 方式三：GitHub Actions运行

1. Fork本仓库到您的账户
2. 在仓库Settings → Secrets中添加上述环境变量
3. 工作流会自动按计划运行（默认每天8点）
4. 也可以手动触发：Actions → Weather Notification → Run workflow

## 获取必要的API密钥

### 和风天气API
1. 访问 [和风天气开发平台](https://dev.qweather.com/)
2. 注册账号并创建项目
3. 获取API Key和API Host

### Telegram机器人
1. 在Telegram中搜索 `@BotFather`
2. 发送 `/newbot` 创建新机器人
3. 获取机器人Token
4. 将机器人添加到群组，获取群组Chat ID

### 企业微信机器人
1. 进入企业微信群聊
2. 点击右上角群设置 → 添加群机器人
3. 获取Webhook地址

### GitHub Token
1. 访问 GitHub Settings → Developer settings → Personal access tokens
2. 生成新的token，勾选`repo`权限

## 城市ID获取

城市ID可以在和风天气API文档中查询，或使用以下格式：
```json
{"北京": "101010100", "上海": "101020100", "广州": "101280101"}
```

## 测试运行

配置完成后，可以运行测试命令验证配置：

```bash
python3 weather.py
```

#