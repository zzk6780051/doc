# 天气通知脚本开发文档

## 项目架构

### 核心模块

```
weather.py
├── 配置管理 (get_config)
├── 天气数据获取 (get_weather_all)
├── 消息格式化 (format_weather_message)
├── 通知发送模块
│   ├── Telegram通知 (send_telegram_message)
│   ├── 企业微信通知 (send_wechat_message)
│   └── GitHub通知 (update_github_notification)
└── 运行入口
    ├── 主函数 (main)
    └── 青龙模式 (ql_weather)
```

## 核心函数详解

### 配置管理

```python
def get_config():
    """从环境变量获取所有配置"""
    # 返回包含所有配置的字典
```

### 天气数据获取

```python
def get_weather_all(location_id, config):
    """一次性获取所有天气信息
    包括：实时天气、空气质量、3天预报
    """
```

### 消息格式化

```python
def format_weather_message(data, city_name, format_type="html"):
    """将天气数据格式化为可读消息
    支持HTML格式（Telegram）和纯文本格式（企业微信）
    """
```

### 通知发送

```python
def send_telegram_message(message, config):
    """发送HTML格式消息到Telegram"""

def send_wechat_message(message, config):
    """发送纯文本消息到企业微信"""

def update_github_notification(weather_message, config):
    """更新GitHub仓库中的通知文件"""
```

## API接口说明

### 和风天气API

- **实时天气**: `/v7/weather/now`
- **空气质量**: `/v7/air/now` 
- **3天预报**: `/v7/weather/3d`

### Telegram Bot API

- **发送消息**: `https://api.telegram.org/bot{token}/sendMessage`

### 企业微信机器人

- **Webhook发送**: 配置的Webhook URL

### GitHub API

- **获取文件**: `GET /repos/{owner}/{repo}/contents/{path}`
- **更新文件**: `PUT /repos/{owner}/{repo}/contents/{path}`

## 扩展开发

### 添加新的通知渠道

1. 在配置函数中添加新的环境变量
2. 创建新的发送函数
3. 在主函数中调用新的发送函数

示例：添加钉钉机器人通知

```python
def send_dingtalk_message(message, config):
    """发送消息到钉钉机器人"""
    if not config['dingtalk']['webhook']:
        return False
    
    payload = {
        "msgtype": "text",
        "text": {
            "content": message
        }
    }
    # 实现发送逻辑
```

### 添加新的天气信息

1. 在`get_weather_all`函数中添加新的API调用
2. 在`format_weather_message`中添加格式化逻辑

### 自定义消息格式

修改`format_weather_message`函数来改变消息的显示样式：

```python
# 自定义消息模板
custom_template = [
    f"📍 {city_name}",
    f"🌡️ 当前温度: {temp}°C",
    # ... 更多自定义字段
]
```

## 错误处理

项目包含完善的错误处理机制：

- API请求超时处理
- 配置缺失检查
- 网络异常捕获
- 各平台返回状态验证

## 调试

### 本地调试

```bash
# 启用详细日志
DEBUG=1 python3 weather.py

# 测试特定功能
python3 -c "from weather import get_weather_all; print(get_weather_all('101010100', get_config()))"
```

### 环境变量验证

```python
# 在代码中添加配置验证
required_vars = ['WEATHER_API_KEY', 'WEATHER_API_HOST']
missing_vars = [var for var in required_vars if not os.environ.get(var)]
if missing_vars:
    print(f"缺少必需环境变量: {missing_vars}")
```

## 性能优化

1. **请求合并**: 使用单个函数获取所有天气数据
2. **超时设置**: 所有网络请求设置超时时间
3. **错误重试**: 可添加重试机制提高稳定性
4. **缓存机制**: 对不常变的数据添加缓存

## 安全考虑

- 所有敏感信息通过环境变量管理
- GitHub Token仅需repo权限
- API请求使用HTTPS加密
- 输入数据验证和转义

## 部署注意事项

### 青龙面板
- 确保Python版本兼容
- 设置正确的文件路径权限
- 配置合适的定时任务间隔

### GitHub Actions
- Secrets配置大小写敏感
- 注意API调用频率限制
- 工作流超时时间设置

## 常见问题排查

1. **天气数据获取失败**
   - 检查API密钥和主机配置
   - 验证城市ID格式

2. **通知发送失败**
   - 检查各平台Token/Webhook配置
   - 验证网络连通性

3. **GitHub更新失败**
   - 检查Token权限
   - 验证仓库文件路径

#