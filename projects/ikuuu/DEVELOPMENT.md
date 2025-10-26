# iKuuu 自动签到开发文档

## 项目结构

```
scripts/
 ├── ikuuu.js                 # 主程序文件
 ├── .github/
 │     └── workflows/
 │           └── ikuuu.yml       # GitHub Actions 工作流
 ├── history/                # 历史记录目录（运行时生成）
 │     └── user_email/
 │         ├── YYYY-MM-DD.log
 │         └── YYYY-MM_stats.json
 └── 其他/
```

## 核心模块

### 1. 配置管理 (`initializeConfig`)

**功能**: 从环境变量加载配置

**支持的环境变量**:
- `IKUUU_ACCOUNTS` - 账户列表（JSON 格式）
- `IKUUU_DOMAIN` - iKuuu 域名
- `IKUUU_MAX_RETRY` - 最大重试次数
- 各种通知相关的环境变量

**兼容性**: 支持新旧环境变量命名

### 2. 历史记录系统 (`HistoryLogger`)

**功能**: 管理用户签到历史和数据统计

**主要方法**:
- `logCheckin()` - 记录单次签到
- `logMonthlyStats()` - 更新月统计
- `getUserStats()` - 获取用户统计
- `generateHistoryReport()` - 生成统计报告

**数据格式**:
```javascript
// 日日志文件
[2024-01-01 12:00:00] SUCCESS - 获得了 500MB流量

// 月统计文件
{
  "total": 30,
  "success": 28,
  "failed": 2,
  "dates": {
    "2024-01-01": { "success": 1, "failed": 0 }
  }
}
```

### 3. GitHub 通知系统 (`GitHubNotifier`)

**功能**: 更新 GitHub 仓库的通知状态

**工作流程**:
1. 获取现有 data.json 文件
2. 更新 notifications 数组
3. 提交更改到仓库

**API 端点**: 
- `GET /repos/{owner}/{repo}/contents/data.json`
- `PUT /repos/{owner}/{repo}/contents/data.json`

### 4. 签到核心 (`checkin`)

**流程**:
1. 登录获取 Cookie
2. 执行签到请求
3. 处理签到结果

**异常处理**:
- 网络请求失败重试
- 重复签到特殊处理
- 登录失败立即终止

## API 接口

### iKuuu 接口

**登录接口**:
```http
POST https://{domain}/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "passwd": "password",
  "code": "",
  "remember_me": "on"
}
```

**签到接口**:
```http
POST https://{domain}/user/checkin
Cookie: {session_cookie}
X-Requested-With: XMLHttpRequest
```

### 通知接口

**Telegram**:
```http
POST https://api.telegram.org/bot{token}/sendMessage
```

**企业微信机器人**:
```http
POST {webhook_url}
```

**企业微信应用**:
```http
POST https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token={token}
```

## 数据结构

### 账户配置
```typescript
interface Account {
  name: string;      // 账户名称
  email: string;     // 邮箱地址
  passwd: string;    // 密码
}
```

### 签到结果
```typescript
interface CheckinResult {
  success: boolean;           // 是否成功
  displayName: string;        // 显示名称
  message: string;           // 原始消息
  simplifiedMessage: string;  // 简化消息
}
```

### 配置对象
```typescript
interface Config {
  DOMAIN: string;
  ACCOUNTS: Account[];
  TG_BOT_TOKEN: string;
  TG_CHAT_ID: string;
  QYWX_WEBHOOK: string;
  // ... 其他配置
}
```

## 错误处理

### 重试机制
```javascript
async function withRetry(fn, retries) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      await delay(2000 * (i + 1));
    }
  }
}
```

### 异常分类
1. **网络异常** - 自动重试
2. **认证异常** - 立即失败
3. **业务异常** - 根据具体逻辑处理

## 扩展开发

### 添加新的通知方式

1. 创建新的通知类
2. 实现发送方法
3. 在 `sendNotifications` 中集成

```javascript
class NewNotifier {
  async sendNotification(message) {
    // 实现发送逻辑
  }
}
```

### 修改签到逻辑

主要修改 `checkin` 函数：
- 调整请求参数
- 修改响应处理
- 更新错误处理逻辑

### 自定义统计报告

修改 `HistoryLogger.generateHistoryReport()` 方法：
- 调整报告格式
- 添加新的统计维度
- 修改显示样式

## 测试建议

### 单元测试
```javascript
// 测试配置加载
test('initializeConfig should load from env', () => {
  process.env.IKUUU_ACCOUNTS = '[{"name":"test","email":"test@test.com","passwd":"123"}]';
  initializeConfig();
  expect(config.ACCOUNTS).toHaveLength(1);
});
```

### 集成测试
- 使用测试账户进行真实签到
- 验证通知发送功能
- 检查历史记录生成

## 部署注意事项

### 环境要求
- Node.js 18+
- 网络访问权限
- 文件系统写入权限

### 安全考虑
- 环境变量加密存储
- 日志信息脱敏
- API 令牌定期更新

## 性能优化

### 异步处理
使用 `Promise.allSettled` 并行发送通知：
```javascript
await Promise.allSettled([
  sendTelegramNotification(message),
  sendWechatWorkNotification(message),
  githubNotifier.updateGitHubNotification(results)
]);
```

### 内存管理
- 及时释放大对象
- 流式处理大文件
- 合理设置缓存

## 版本更新

记录主要版本变更和兼容性说明，帮助用户平滑升级。

#