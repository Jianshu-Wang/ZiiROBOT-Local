# ZiiRobot

智能机器人控制系统，支持语音识别和 Web 配置界面。专为树莓派 5 设计。

## 项目简介

ZiiRobot 是一个集成了语音识别和 Web 管理界面的智能机器人控制系统。项目包含两个核心服务：

- **KWS (Keyword Spotting)**: 基于 TC-ResNet8 的实时语音关键词识别服务
- **Web**: WiFi 配置和机器人管理的 Web 界面

## 主要特性

### 语音识别服务 (KWS)
- 实时语音关键词识别
- 基于 TC-ResNet8 深度学习模型
- RESTful API 接口
- 识别结果历史记录
- 低延迟推理

### Web 管理界面
- WiFi 网络配置
- 自动热点模式切换（无 WiFi 时自动启用热点）
- 响应式设计，支持移动设备
- mDNS 服务发现（可通过 `ziirobot.local` 访问）
- 友好的用户界面


## 快速开始

### 前置要求

- 树莓派 5（或其他 Linux 设备）
- Docker 和 Docker Compose
- 音频输入设备（用于语音识别）
- 网络适配器（用于 WiFi/热点）

### 安装步骤

1. **克隆项目**
```bash
git clone <repository-url>
cd ZiiRobot
```

2. **准备模型文件**
确保以下文件存在：
- `KWS/TCResNet8_best_model/` - 训练好的模型目录
- `KWS/labels.txt` - 标签文件

3. **使用 Docker Compose 启动**

```bash
# 启动所有服务
docker compose up -d

# 查看日志
docker compose logs -f

# 停止服务
docker compose down
```

### 单独启动服务

#### 启动 KWS 服务

```bash
docker compose up -d kws
```

#### 启动 Web 服务

```bash
docker compose up -d web
```

## 📡 API 文档

### KWS 语音识别 API

服务运行在 `http://localhost:8000`

#### 1. 获取最新识别结果

```bash
GET /api/latest
```

**响应示例：**
```json
{
  "status": "success",
  "data": {
    "label": "go",
    "confidence": 0.9876,
    "predicted_class": 2,
    "inference_time_ms": 45.23,
    "timestamp": "2024-01-01T12:00:00.123456"
  }
}
```

#### 2. 获取标签文本（纯文本格式）

```bash
GET /api/label/text
```

**响应示例：**
```
go
```

#### 3. 获取历史记录

```bash
GET /api/history?limit=10
```

**响应示例：**
```json
{
  "status": "success",
  "count": 10,
  "data": [...]
}
```

#### 4. 健康检查

```bash
GET /api/health
```

### Web 管理 API

服务运行在 `http://localhost:5000`

#### 1. 获取网络状态

```bash
GET /api/status
```

**响应示例：**
```json
{
  "mode": "wifi",
  "ip": "192.168.1.100"
}
```

#### 2. 配置 WiFi

```bash
POST /api/set_wifi
Content-Type: application/json

{
  "ssid": "YourWiFiName",
  "password": "YourPassword"
}
```

#### 3. 获取 WiFi 连接状态

```bash
GET /api/wifi_status
```

## 访问方式

### WiFi 模式
当设备连接到 WiFi 时，可通过以下方式访问：
- IP 地址：`http://<RaspberryPi-IP>:5000`
- mDNS：`http://ziirobot.local:5000`

### 热点模式
当设备未连接 WiFi 时，会自动启用热点模式：
- IP 地址：`http://10.42.0.1:5000`
- mDNS：`http://ziirobot.local:5000`

连接热点后，可通过 Web 界面配置 WiFi。

## 配置说明

### Docker Compose 配置

主要配置项位于 `docker-compose.yml`：

- **KWS 服务**：
  - 端口：8000（API）
  - 需要访问音频设备：`/dev/snd`
  - 网络模式：host

- **Web 服务**：
  - 端口：5000（Web 界面）
  - 需要访问 NetworkManager 配置
  - 网络模式：host

### 环境变量

#### KWS 服务
- `TF_CPP_MIN_LOG_LEVEL=2` - TensorFlow 日志级别
- `TF_ENABLE_ONEDNN_OPTS=0` - 禁用 OneDNN 优化

#### Web 服务
- `FLASK_APP=app.py` - Flask 应用入口
- `FLASK_ENV=production` - Flask 环境

## 开发说明

### 本地开发

#### KWS 服务开发

```bash
cd KWS
pip install -r requirements.txt
python3 voice_recognition.py --labels labels.txt --api-mode
```

#### Web 服务开发

**前端开发：**
```bash
cd Web
npm install
npm run dev
```

**后端开发：**
```bash
cd Web
pip install -r requirements.txt
python app.py
```

### 构建 Docker 镜像

```bash
# 构建 KWS 镜像
docker build -t ziirobot-kws ./KWS

# 构建 Web 镜像
docker build -t ziirobot-web ./Web
```

## 故障排查

### KWS 服务问题

**问题：无法识别语音**
- 检查音频设备是否正确挂载：`ls -l /dev/snd`
- 查看容器日志：`docker compose logs kws`
- 确认模型文件存在且路径正确

**问题：API 无法访问**
- 检查服务是否运行：`docker compose ps`
- 测试健康检查：`curl http://localhost:8000/api/health`
- 检查端口是否被占用

### Web 服务问题

**问题：无法访问 Web 界面**
- 检查服务状态：`docker compose ps`
- 查看日志：`docker compose logs web`
- 确认端口 5000 未被占用

**问题：WiFi 配置失败**
- 检查 NetworkManager 是否运行
- 确认容器有足够的权限（privileged: true）
- 查看详细错误日志

**问题：mDNS 不工作**
- 确认 avahi-daemon 服务运行（宿主机）
- 检查防火墙设置
- 尝试直接使用 IP 地址访问

## 相关文档

- [KWS 服务详细文档](KWS/README.md)
- [Web 服务配置文档](Web/config/README.md)

