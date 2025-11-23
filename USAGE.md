# iKuuu 自动签到使用说明

## 📋 目录
- [快速开始](#快速开始)
- [环境变量配置](#环境变量配置)
- [部署方式](#部署方式)
- [通知配置](#通知配置)
- [高级功能](#高级功能)
- [故障排除](#故障排除)
- [常见问题](#常见问题)

## 🚀 快速开始

### 选择部署方式

根据你的需求选择合适的部署方式：

| 部署方式 | 适用场景 | 难度 | 推荐度 |<
|---------|---------|------|--------|
| 青龙面板 | 已有青龙环境，需要可视化管理 | 简单 | ⭐⭐⭐⭐⭐ |
| GitHub Actions | 无服务器，想用免费资源 | 简单 | ⭐⭐⭐⭐ |
| Docker | 有自己的服务器，需要容器化部署 | 中等 | ⭐⭐⭐⭐ |
| 直接运行 | 测试或开发环境 | 简单 | ⭐⭐⭐ |

### 环境准备

1. **Node.js 环境**: 确保 Node.js 版本 ≥ 18.0
   ```bash
   node --version  # 检查版本
   ```

2. **网络要求**: 能够访问 iKuuu 服务和外网通知接口

3. **账户信息**: 准备好 iKuuu 账户的邮箱和密码

## ⚙️ 环境变量配置

### 必需配置

#### 账户配置 (IKUUU_ACCOUNTS)
**格式**: JSON 数组
```json
[
  {
    "name": "账号显示名称",
    "email": "your_email@example.com",
    "passwd": "your_password"
  }
]
```

**示例**:
```bash
# 单账户
IKUUU_ACCOUNTS=[{"name":"主账号","email":"user1@example.com","passwd":"password1"}]

# 多账户
IKUUU_ACCOUNTS=[{"name":"账号1","email":"user1@example.com","passwd":"pass1"},{"name":"账号2","email":"user2@example.com","passwd":"pass2"}]
```

### 基础配置（可选）

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `IKUUU_DOMAIN` | `ikuuu.eu` | iKuuu 服务域名 |
| `IKUUU_MAX_RETRY` | `3` | 失败重试次数 |
| `IKUUU_ENABLE_HISTORY` | `true` | 是否启用历史记录 |
| `IKUUU_LANGUAGE` | `zh-CN` | 界面语言 (`zh-CN`/`en`) |

## 🛠️ 部署方式

### 方式一：青龙面板部署（推荐）

#### 步骤详解

1. **安装青龙面板**（如果尚未安装）
   ```bash
   # 使用 Docker 安装
   docker run -d \
     --name qinglong \
     -p 5700:5700 \
     -v /opt/ql/data:/ql/data \
     whyour/qinglong:latest
   ```

2. **添加环境变量**
   - 登录青龙面板 (http://localhost:5700)
   - 进入「环境变量」页面
   - 点击「添加变量」
   - 填写变量名称和值

3. **添加脚本文件**
   - 进入「脚本管理」
   - 点击「添加脚本」
   - 名称: `ikuuu.js`
   - 将项目中的 `ikuuu.js` 内容完整复制粘贴
   - 点击「保存」

4. **设置定时任务**
   - 进入「定时任务」
   - 点击「添加任务」
   - 名称: `iKuuu 自动签到`
   - 命令: `task ikuuu.js`
   - 定时规则: `0 8 * * *` (每天上午8点)

#### 推荐定时规则

```bash
# 每天上午8点（推荐）
0 8 * * *

# 每天多个时间点（提高成功率）
0 8,12,18 * * *

# 随机时间（避免高峰）
0 8-10 * * *  # 8-10点之间的随机时间
```

### 方式二：GitHub Actions 部署

#### 详细步骤

1. **Fork 项目仓库**
   - 访问 [项目仓库](https://github.com/zzk1314/scripts)
   - 点击右上角 "Fork" 按钮
   - 选择你的账户作为目标

2. **配置 Secrets**
   - 进入你 Fork 的仓库
   - 点击 "Settings" → "Secrets and variables" → "Actions"
   - 点击 "New repository secret"

   **必需 Secrets**:
   - `IKUUU_ACCOUNTS`: 账户配置 JSON

   **可选 Secrets**:
   - `IKUUU_TG_BOT_TOKEN`: Telegram Bot Token
   - `IKUUU_TG_CHAT_ID`: Telegram Chat ID
   - 其他通知相关的配置

3. **启用 Actions**
   - 进入 "Actions" 页面
   - 点击 "I understand my workflows, go ahead and enable them"
   - 工作流将按计划自动运行

#### 自定义执行时间

编辑 `.github/workflows/ikuuu.yml` 文件中的 `schedule` 部分：

```yaml
schedule:
  - cron: '0 8 * * *'    # 每天 UTC 时间 8:00 (北京时间 16:00)
  - cron: '0 20 * * *'   # 每天 UTC 时间 20:00 (北京时间 次日 4:00)
```

### 方式三：Docker 部署

#### 完整部署流程

1. **创建配置目录**
   ```bash
   mkdir -p /opt/ikuuu/{config,history}
   ```

2. **创建配置文件**
   ```bash
   cat > /opt/ikuuu/config/config.json << 'EOF'
   {
     "ACCOUNTS": [
       {
         "name": "我的账号",
         "email": "your_email@example.com",
         "passwd": "your_password"
       }
     ],
     "DOMAIN": "ikuuu.eu",
     "ENABLE_HISTORY": true,
     "MAX_RETRY": 3
   }
   EOF
   ```

3. **运行容器**
   ```bash
   docker run -d \
     --name ikuuu \
     --restart unless-stopped \
     -v /opt/ikuuu/config:/app/config \
     -v /opt/ikuuu/history:/app/history \
     zzk1314/ikuuu:latest
   ```

4. **设置定时执行**（使用 crontab）
   ```bash
   # 编辑 crontab
   crontab -e

   # 添加定时任务（每天8点执行）
   0 8 * * * docker restart ikuuu
   ```

### 方式四：直接运行

1. **下载脚本**
   ```bash
   wget https://raw.githubusercontent.com/zzk1314/scripts/main/ikuuu.js
   ```

2. **设置环境变量**
   ```bash
   export IKUUU_ACCOUNTS='[{"name":"我的账号","email":"your_email@example.com","passwd":"your_password"}]'
   ```

3. **执行脚本**
   ```bash
   node ikuuu.js
   ```

4. **设置定时任务**（Linux/Mac）
   ```bash
   # 编辑 crontab
   crontab -e

   # 添加定时任务
   0 8 * * * cd /path/to/script && node ikuuu.js
   ```

## 📢 通知配置

### Telegram 配置指南

#### 1. 创建 Bot
1. 在 Telegram 中搜索 `@BotFather`
2. 发送 `/newbot` 命令
3. 按提示设置机器人名称和用户名
4. 保存生成的 Bot Token

#### 2. 获取 Chat ID
1. 在 Telegram 中搜索 `@userinfobot`
2. 发送任意消息
3. 复制你的 Chat ID

#### 3. 配置环境变量
```bash
IKUUU_TG_BOT_TOKEN=123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11
IKUUU_TG_CHAT_ID=123456789
```

### 企业微信配置指南

#### 方式一：群机器人（推荐）
1. 进入企业微信，选择要通知的群聊
2. 点击右上角群设置 → 添加群机器人
3. 设置机器人名称，获取 Webhook URL
4. 配置环境变量：
   ```bash
   IKUUU_QYWX_WEBHOOK=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   ```

#### 方式二：企业微信应用
1. 登录企业微信管理后台
2. 进入「应用管理」→「自建应用」→「创建应用」
3. 配置应用信息，获取以下参数：
   - AgentId
   - Secret
   - CorpId（企业ID）
4. 配置环境变量：
   ```bash
   IKUUU_WECOM_CORPID=wwxxxxxxxxxxxxxxxx
   IKUUU_WECOM_AGENT_ID=1000002
   IKUUU_WECOM_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   IKUUU_WECOM_TO_USER=@all
   ```

### Bark 配置（iOS）
1. 在 App Store 安装 Bark 应用
2. 打开应用获取设备 Key
3. 配置环境变量：
   ```bash
   IKUUU_BARK_URL=https://api.day.app/your_device_key
   ```

### 钉钉配置
1. 在钉钉群聊中添加自定义机器人
2. 选择「自定义」→「加签」
3. 获取 Webhook URL
4. 配置环境变量：
   ```bash
   IKUUU_DINGTALK_WEBHOOK=https://oapi.dingtalk.com/robot/send?access_token=xxxx
   ```

### GitHub 通知配置
1. 创建 [Personal Access Token](https://github.com/settings/tokens)
   - 权限：`repo`（完全控制仓库）
2. 准备一个用于存储通知的仓库
3. 配置环境变量：
   ```bash
   IKUUU_GH_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
   IKUUU_GH_REPO=username/repo-name
   IKUUU_GH_BRANCH=main
   ```

## 🎯 高级功能

### 域名自动切换

当主域名不可用时自动切换到备用域名：

```bash
IKUUU_AUTO_SWITCH_DOMAIN=true
IKUUU_BACKUP_DOMAINS=["ikuuu.eu","ikuuu.art","ikuuu.club","ikuuu.xyz"]
```

### 节点状态检查

启用节点状态监控功能：

```bash
IKUUU_ENABLE_NODE_STATUS=true
```

### 代理设置

通过代理服务器访问：

```bash
IKUUU_PROXY_URL=http://proxy.example.com:8080
# 或
IKUUU_PROXY_URL=http://username:password@proxy.example.com:8080
```

### 健康检查模式

```bash
# 执行健康检查（不签到）
node ikuuu.js --health

# 输出示例：
🔍 检查节点状态...
📍 ikuuu.eu: 200 (156ms)
📍 ikuuu.art: 200 (234ms)  
❌ ikuuu.club: 不可用
✅ 健康检查完成！
```

### 历史记录管理

```bash
# 清理30天前的历史记录
node ikuuu.js --cleanup

# 查看当前配置
node ikuuu.js --config
```

## 🔧 故障排除

### 常见错误及解决方案

#### 1. "登录失败: 账户或密码错误"
**可能原因**:
- 账户密码错误
- 账户被锁定或禁用
- 域名配置错误

**解决方案**:
1. 手动登录 iKuuu 网站验证账户
2. 检查密码是否正确（注意大小写）
3. 尝试更换域名

#### 2. "网络错误: ETIMEDOUT"
**可能原因**:
- 网络连接问题
- 服务器响应慢
- 防火墙阻挡

**解决方案**:
1. 检查网络连接
2. 增加超时时间：`IKUUU_TIMEOUT=60000`
3. 配置代理服务器

#### 3. "获取Cookie失败"
**可能原因**:
- 登录响应格式变化
- 网络中间件干扰

**解决方案**:
1. 启用调试模式查看详细日志
2. 检查网络代理设置
3. 更新到最新版本脚本

### 调试技巧

#### 启用详细日志
```bash
export IKUUU_DEBUG=true
node ikuuu.js
```

#### 手动测试登录
```bash
# 使用 curl 测试登录接口
curl -X POST "https://ikuuu.eu/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"your_email","passwd":"your_password","remember_me":"on"}'
```

### 日志分析

#### 成功日志示例
```
🚀 开始 ikuuu 签到任务...
✅ 配置初始化完成
🔍 正在处理: 主账号 (us****@example.com)
✅ 主账号 签到成功: 获得 500MB
📝 已记录 主账号 的签到历史
✅ 签到任务完成！
```

#### 错误日志示例
```
❌ 主账号 签到失败: 登录失败: 账户或密码错误
⚠️ 第1次尝试失败，2次重试机会
```

## ❓ 常见问题

### Q: 支持多少个账户？
A: 理论上无限制，但建议不超过 10 个账户，避免触发频率限制。

### Q: 签到频率有限制吗？
A: iKuuu 服务通常每天只能签到一次，重复签到会返回"重复签到"提示。

### Q: 数据存储在哪里？
A: 历史记录存储在 `history/` 目录下，每个账户有独立的文件夹。

### Q: 如何备份数据？
A: 备份 `history/` 目录即可，所有历史记录都在其中。

### Q: 脚本安全吗？
A: 脚本开源，不收集任何用户数据，所有敏感信息都通过环境变量配置。

### Q: 遇到问题如何求助？
A: 
1. 查看本文档的故障排除部分
2. 检查项目 Issues 中是否有类似问题
3. 提交新的 Issue 并附上详细日志

## 📞 获取帮助

- **文档**: 查看 [开发文档](DEVELOPMENT.md) 了解技术细节
- **Issues**: [提交问题](https://github.com/zzk1314/scripts/issues)
- **讨论**: 加入 [Telegram 群组](https://t.me/your_group)

## 🔄 更新日志

### v2.0 主要更新
- ✅ 域名自动切换功能
- ✅ 节点状态监控
- ✅ 多语言支持
- ✅ 健康检查模式
- ✅ 多种新通知方式
- ✅ 代理服务器支持

---

**如有其他问题，请查阅文档或提交 Issue！**

> ⭐ **如果这个项目对你有帮助，请给个 Star 支持一下！**