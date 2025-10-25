# 开发文档

## 项目架构

### 文件结构
```
project/
├── index.html     # 主页面 - 负责UI展示和用户交互
└── data.json      # 数据文件 - 存储所有通知信息
```

### 技术实现
- **前端**: 纯原生 HTML/CSS/JavaScript
- **数据交互**: Fetch API 异步加载 JSON 数据
- **样式**: CSS3 变量 + 媒体查询实现响应式
- **部署**: 静态文件部署，兼容 GitHub Pages

## 核心代码解析

### HTML 结构
```html
<!-- 头部区域 -->
<header>
    <div class="header-content">
        <h1>实时通知中心</h1>
        <p class="last-updated">最后更新: <span id="lastUpdateTime"></span></p>
    </div>
    <button id="refreshBtn" class="refresh-btn">↻ 刷新</button>
</header>

<!-- 通知容器 -->
<div id="notificationsContainer" class="notifications-container">
    <!-- 动态加载的通知卡片 -->
</div>
```

### JavaScript 核心功能

#### 数据加载
```javascript
async function loadNotifications() {
    try {
        let url = './data.json?t=' + Date.now(); // 缓存破坏
        const response = await fetch(url);
        const data = await response.json();
        renderNotifications(data.notifications);
    } catch (error) {
        showErrorState('加载失败');
    }
}
```

#### 通知渲染
```javascript
function renderNotifications(notifications) {
    // 按时间倒序排序
    const sorted = [...notifications].sort((a, b) => 
        new Date(b.date) - new Date(a.date)
    );
    
    // 生成HTML
    notificationsContainer.innerHTML = sorted.map(notification => `
        <div class="notification">
            <div class="notification-header">
                <div class="notification-title">${notification.title}</div>
                <div class="notification-date">${formatDate(notification.date)}</div>
            </div>
            <div class="notification-content">
                ${notification.content}
            </div>
        </div>
    `).join('');
}
```

### CSS 响应式设计

#### 桌面端样式
```css
.notification {
    background: white;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    border-left: 4px solid #3498db;
}
```

#### 移动端适配
```css
@media (max-width: 600px) {
    .refresh-btn {
        border-radius: 50%; /* 圆形按钮 */
        width: 44px;
        height: 44px;
    }
    
    .refresh-btn .text {
        display: none; /* 隐藏文字 */
    }
}
```

## 与青龙脚本对接

### 数据格式约定
青龙脚本需要输出符合以下格式的 JSON 数据：

```json
{
  "notifications": [
    {
      "id": 2,
      "title": "任务执行结果",
      "content": "任务执行完成<br>✅ 成功: 10个任务",
      "date": "2025-10-07T00:43:09.889Z"
    }
  ]
}
```

### 集成方式

#### 方式一：直接文件写入
青龙脚本通过写入 `data.json` 文件更新通知：

```bash
#!/bin/bash
# 青龙脚本示例
echo '{
  "notifications": [
    {
      "id": 2,
      "title": "任务完成",
      "content": "脚本执行成功",
      "date": "'"$(date -Iseconds)"'"
    }
  ]
}' > data.json
```

#### 方式二：API 调用 + GitHub Actions
1. 青龙脚本调用 GitHub API
2. 触发 GitHub Actions 工作流
3. 自动更新 `data.json` 文件

### 时间格式处理
确保日期时间使用 ISO 8601 格式：
```javascript
// 正确格式
"2025-10-07T12:58:02"           // 简单格式
"2025-10-07T12:58:02.000Z"      // 完整格式

// 在脚本中生成
const now = new Date().toISOString();
```

## 扩展开发

### 添加新功能

#### 1. 通知分类筛选
```javascript
// 添加分类字段
{
  "id": 1,
  "title": "通知标题",
  "content": "通知内容",
  "date": "2025-10-07T00:43:09.889Z",
  "category": "signin" // 新增分类字段
}
```

#### 2. 自动轮询刷新
```javascript
// 每5分钟自动刷新
setInterval(loadNotifications, 5 * 60 * 1000);
```

#### 3. 通知状态持久化
添加本地存储记录已读状态。

### 样式自定义
通过修改 CSS 变量快速调整主题：
```css
:root {
    --primary-color: #3498db;    /* 主色调 */
    --secondary-color: #2980b9;  /* 次要色调 */
    --light-bg: #f8f9fa;         /* 背景色 */
    --border-radius: 8px;        /* 圆角大小 */
}
```

## 故障排除

### 常见问题

1. **通知不显示**
   - 检查 `data.json` 格式是否正确
   - 确认 JSON 文件路径配置
   - 查看浏览器控制台错误信息

2. **时间显示异常**
   - 确保日期格式为 ISO 8601
   - 检查时区设置

3. **移动端布局问题**
   - 验证 viewport 设置
   - 测试不同屏幕尺寸

### 调试技巧
- 打开浏览器开发者工具
- 检查 Network 标签确认数据加载
- 查看 Console 标签的错误信息

#