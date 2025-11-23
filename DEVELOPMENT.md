# iKuuu 自动签到开发文档

## 项目概述

iKuuu 自动签到项目是一个基于 Node.js 的自动化签到工具，专为 iKuuu VPN 服务设计。项目采用模块化架构，支持多账户管理、多种通知方式、历史记录统计等功能。

## 项目结构

```
scripts/
 ├── ikuuu.js                 # 主程序文件
 ├── config.json              # 配置文件（可选）
 ├── .github/
 │     └── workflows/
 │           └── ikuuu.yml    # GitHub Actions 工作流
 ├── history/                 # 历史记录目录（运行时生成）
 │     └── user_email/
 │         ├── YYYY-MM-DD.log
 │         └── YYYY-MM_stats.json
 ├── node_status.json         # 节点状态缓存
 └── traffic_stats.json       # 流量统计缓存（如果启用）
```

## 核心架构

### 模块设计

```
┌─────────────────┐
│   主程序入口     │
│   (ikuuu.js)    │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   配置管理器     │
│ (ConfigManager) │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐    ┌─────────────────┐
│   域名管理器     │    │   历史记录器    │
│ (DomainManager) │    │ (HistoryLogger) │
└─────────┬───────┘    └─────────┬───────┘
          │                      │
          ▼                      ▼
┌─────────────────┐    ┌─────────────────┐
│   签到执行器     │    │   通知发送器    │
│   (checkin)     │    │ (Notifiers)     │
└─────────┬───────┘    └─────────┬───────┘
          │                      │
          └──────────┬───────────┘
                     │
                     ▼
              ┌─────────────┐
              │  结果处理器  │
              └─────────────┘
```

## 核心模块详解

### 1. 配置管理 (`ConfigManager`)

**功能**: 统一管理应用配置，支持环境变量和配置文件

**配置源优先级**:
1. 环境变量（最高优先级）
2. 配置文件 (`config.json`)
3. 默认配置 (`DEFAULT_CONFIG`)

**支持的配置项**:
```javascript
{
  // 基础配置
  DOMAIN: 'ikuuu.eu',
  ACCOUNTS: [],
  MAX_RETRY: 3,
  ENABLE_HISTORY: true,
  
  // 通知配置
  TG_BOT_TOKEN: '',
  TG_CHAT_ID: '',
  QYWX_WEBHOOK: '',
  WECOM_CORPID: '',
  WECOM_AGENT_ID: '',
  WECOM_SECRET: '',
  WECOM_TO_USER: '@all',
  
  // 高级功能
  AUTO_SWITCH_DOMAIN: true,
  BACKUP_DOMAINS: ['ikuuu.eu', 'ikuuu.art', 'ikuuu.club'],
  ENABLE_NODE_STATUS: false,
  PROXY_URL: '',
  TIMEOUT: 30000,
  LANGUAGE: 'zh-CN',
  
  // GitHub 集成
  GH_TOKEN: '',
  GH_REPO: '',
  GH_BRANCH: 'main'
}
```

### 2. 域名管理 (`DomainManager`)

**功能**: 自动检测和切换可用域名

**算法流程**:
1. 测试所有配置域名的响应时间和可用性
2. 选择响应最快的可用域名
3. 如果所有域名都不可用，抛出错误

**域名列表**:
1. ikuuu.de
2.kuuu.boo

### 3. 历史记录系统 (`HistoryLogger`)

**功能**: 管理用户签到历史和数据统计

**目录结构**:
```
history/
 └── user_example_com/          # 用户目录（邮箱转换）
     ├── 2024-01-15.log         # 每日日志
     ├── 2024-01_stats.json     # 月统计
     └── 2024-02_stats.json
```

**数据格式**:
```javascript
// 日日志文件格式
[2024-01-15 08:30:25] SUCCESS - 签到成功，获得了 500MB流量
[2024-01-15 08:30:25] FAILED - 登录失败: 账户或密码错误

// 月统计文件格式
{
  "total": 15,
  "success": 14,
  "failed": 1,
  "dates": {
    "2024-01-15": {
      "success": 1,
      "failed": 0
    }
  }
}
```

### 4. 节点状态检查 (`NodeStatusChecker`)

**功能**: 监控各节点健康状态

**检查指标**:
- 响应状态码
- 响应时间
- 可用性状态

**缓存机制**: 结果缓存到 `node_status.json`，避免频繁检查

### 5. 通知系统

**支持的通知渠道**:
- **Telegram Bot** - 实时消息推送
- **企业微信机器人** - Webhook 方式
- **企业微信应用** - API 方式
- **Bark** - iOS 设备推送
- **钉钉机器人** - 群消息推送
- **GitHub** - 更新仓库通知

## API 接口文档

### iKuuu 服务接口

**登录接口**:
```http
POST https://{domain}/auth/login
Content-Type: application/json

请求体:
{
  "email": "user@example.com",
  "passwd": "password",
  "code": "",
  "remember_me": "on"
}

响应:
{
  "ret": 1,
  "msg": "登录成功"
}
```

**签到接口**:
```http
POST https://{domain}/user/checkin
Cookie: {session_cookie}
X-Requested-With: XMLHttpRequest

响应:
{
  "ret": 1,
  "msg": "签到成功，获得了 500MB流量"
}
```

### 第三方通知接口

**Telegram Bot API**:
```http
POST https://api.telegram.org/bot{token}/sendMessage
Content-Type: application/json

{
  "chat_id": "123456",
  "text": "消息内容",
  "parse_mode": "HTML"
}
```

**企业微信机器人**:
```http
POST {webhook_url}
Content-Type: application/json

{
  "msgtype": "text",
  "text": {
    "content": "消息内容"
  }
}
```

**GitHub API**:
```http
PUT https://api.github.com/repos/{owner}/{repo}/contents/data.json
Authorization: token {gh_token}
Content-Type: application/json

{
  "message": "更新通知",
  "content": "base64编码的内容",
  "sha": "文件SHA"
}
```

## 错误处理机制

### 重试策略
```javascript
async function withRetry(fn, retries) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      console.log(`第${i + 1}次尝试失败，${retries - i - 1}次重试机会`);
      await delay(2000 * (i + 1)); // 指数退避
    }
  }
}
```

### 异常分类处理

| 异常类型 | 处理方式 | 重试策略 |
|---------|---------|---------|
| 网络超时 | 自动重试 | 3次，指数退避 |
| 登录失败 | 立即终止 | 不重试 |
| 重复签到 | 视为成功 | 不重试 |
| API限制 | 等待后重试 | 2次，固定间隔 |

## 扩展开发指南

### 添加新的通知方式

1. **创建通知类**:
```javascript
class NewNotifier {
  constructor() {
    // 初始化配置
  }
  
  async sendNotification(message) {
    // 实现发送逻辑
    try {
      const response = await fetch(this.webhookUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text: message })
      });
      return response.ok;
    } catch (error) {
      console.error('通知发送失败:', error);
      return false;
    }
  }
}
```

2. **集成到通知系统**:
```javascript
// 在 sendNotifications 函数中添加
const newNotifier = new NewNotifier();
await Promise.allSettled([
  // ... 其他通知
  newNotifier.sendNotification(fullMessage)
]);
```

3. **更新配置管理**:
```javascript
// 在 DEFAULT_CONFIG 中添加配置项
NEW_NOTIFIER_URL: '',

// 在 initializeConfig 中加载配置
NEW_NOTIFIER_URL: env.IKUUU_NEW_NOTIFIER_URL || fileConfig.NEW_NOTIFIER_URL
```

### 自定义签到逻辑

修改 `checkin` 函数来适应不同的签到流程：

```javascript
async function customCheckin(account, domain) {
  // 1. 自定义登录逻辑
  const loginResult = await customLogin(account, domain);
  
  // 2. 自定义签到逻辑
  const checkinResult = await customCheckinRequest(loginResult.cookies);
  
  // 3. 自定义结果处理
  return processCustomResult(checkinResult);
}
```

### 添加新的统计维度

在 `HistoryLogger` 类中添加新的统计方法：

```javascript
class EnhancedHistoryLogger extends HistoryLogger {
  // 添加周统计
  logWeeklyStats(account, date, isSuccess) {
    // 实现周统计逻辑
  }
  
  // 添加自定义报告
  generateCustomReport(accounts) {
    // 生成自定义格式的报告
  }
}
```

## 测试策略

### 单元测试示例
```javascript
// 测试配置加载
describe('Config Management', () => {
  test('should load accounts from environment', () => {
    process.env.IKUUU_ACCOUNTS = JSON.stringify([
      { name: 'test', email: 'test@test.com', passwd: '123' }
    ]);
    initializeConfig();
    expect(config.ACCOUNTS).toHaveLength(1);
    expect(config.ACCOUNTS[0].email).toBe('test@test.com');
  });
  
  test('should validate configuration', () => {
    const errors = configManager.validateConfig({});
    expect(errors).toContain('DOMAIN 未配置');
    expect(errors).toContain('ACCOUNTS 未配置');
  });
});

// 测试重试机制
describe('Retry Mechanism', () => {
  test('should retry on failure', async () => {
    let attempts = 0;
    const failingFn = () => {
      attempts++;
      throw new Error('Temporary failure');
    };
    
    await expect(withRetry(failingFn, 3)).rejects.toThrow();
    expect(attempts).toBe(3);
  });
});
```

### 集成测试
```javascript
// 模拟完整签到流程
describe('Checkin Integration', () => {
  test('should complete full checkin process', async () => {
    // 模拟登录响应
    mockLoginResponse(200, { ret: 1, msg: '登录成功' });
    
    // 模拟签到响应  
    mockCheckinResponse(200, { ret: 1, msg: '签到成功' });
    
    const result = await checkin(testAccount);
    expect(result).toContain('签到成功');
  });
});
```

## 性能优化

### 并发处理
使用 `Promise.allSettled` 实现通知的并行发送：
```javascript
const notificationPromises = [
  sendTelegramNotification(message),
  sendWechatWorkNotification(message),
  sendBarkNotification(message),
  githubNotifier.updateGitHubNotification(results)
];

await Promise.allSettled(notificationPromises);
```

### 内存管理
- 及时释放大对象引用
- 使用流式处理大文件
- 合理设置缓存过期时间

### 网络优化
- 连接复用
- 请求超时设置
- 代理支持

## 安全考虑

### 敏感信息处理
```javascript
// 信息脱敏
function maskString(str, visibleStart = 2, visibleEnd = 2) {
  if (!str) return '';
  if (str.length <= visibleStart + visibleEnd) return str;
  return `${str.substring(0, visibleStart)}****${str.substring(str.length - visibleEnd)}`;
}
```

### 安全最佳实践
1. **环境变量加密**: 敏感信息通过环境变量传递
2. **文件权限控制**: 历史记录文件设置适当权限
3. **输入验证**: 验证所有外部输入
4. **错误信息脱敏**: 避免泄露敏感信息

## 部署指南

### 环境要求
- Node.js 18.0 或更高版本
- 网络访问权限（访问 iKuuu 服务和外网通知）
- 文件系统写入权限（历史记录）

### 监控建议
- 日志轮转和清理
- 错误报警机制
- 性能监控指标

## 版本兼容性

### Node.js 版本支持
- Node.js 18.x: ✅ 完全支持
- Node.js 16.x: ⚠️ 部分功能可能受限
- Node.js 14.x: ❌ 不支持

### 平台兼容性
- Linux: ✅ 完全支持
- Windows: ✅ 完全支持  
- macOS: ✅ 完全支持
- Docker: ✅ 完全支持

## 故障排查

### 常见问题解决
1. **网络连接问题**: 检查代理配置和防火墙设置
2. **认证失败**: 验证账户密码和域名配置
3. **通知发送失败**: 检查各平台 API 配置和限制
4. **内存泄漏**: 监控内存使用，及时重启服务

### 调试模式
设置 `DEBUG=true` 环境变量启用详细日志：
```bash
export IKUUU_DEBUG=true
node ikuuu.js
```

## 贡献指南

欢迎提交 Issue 和 Pull Request 来改进项目。在提交代码前请确保：
1. 通过所有测试
2. 更新相关文档
3. 遵循代码风格规范
```
