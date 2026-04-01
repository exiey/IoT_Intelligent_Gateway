# IoT_Intelligent_Gateway
一款智能化的物联网网关项目，旨在打通异构物联网设备、协议与云平台之间的壁垒。该网关支持数据采集、协议转换、边缘计算及安全数据传输，适用于工业级和消费级物联网场景。

## 📋 目录
- [功能特性](#功能特性)
- [架构设计](#架构设计)
- [支持的协议](#支持的协议)
- [快速开始](#快速开始)
  - [环境依赖](#环境依赖)
  - [安装部署](#安装部署)
  - [配置说明](#配置说明)
  - [启动网关](#启动网关)
- [目录结构](#目录结构)
- [使用示例](#使用示例)
- [贡献指南](#贡献指南)
- [许可证](#许可证)
- [联系我们](#联系我们)

## ✨ 功能特性
- **多协议兼容**：无缝对接主流物联网协议（MQTT、Modbus、TCP/UDP、OPC UA 等）。
- **协议转换**：实现不同协议间的数据格式转换（如 Modbus RTU → MQTT、TCP → HTTP）。
- **边缘计算**：在边缘端完成轻量化数据处理（过滤、聚合、分析），降低云端带宽消耗。
- **安全保障**：支持 TLS/SSL 数据传输加密、设备身份认证及访问权限控制。
- **可扩展性**：模块化设计，便于扩展新协议或自定义边缘算法。
- **远程管理**：支持远程配置下发、固件升级及状态监控。
- **跨平台部署**：可运行于 Linux、Windows 及嵌入式系统（ARM/x86 架构）。

## 🏗️ 架构设计
网关采用分层模块化架构，核心分为五层：
1. **设备接入层**：管理物理设备连接与协议解析（Modbus、MQTT 等）。
2. **协议转换层**：实现设备与上层平台间的数据格式、协议转换。
3. **边缘计算层**：本地数据处理（过滤、计算、规则触发）。
4. **云端集成层**：对接公有/私有云平台（AWS IoT、Azure IoT、阿里云物联网平台等）。
5. **管理层**：提供配置管理、状态监控、日志记录能力。

```
┌─────────────────┐    ┌─────────────────────────────────────┐    ┌───────────────┐
│ 物联网设备      │    │ IoT 智能网关                        │    │ 云平台        │
│ （传感器/PLC）  │───▶│ 设备接入 → 协议转换 → 边缘计算 → 云端同步 │───▶│ （AWS/Azure/  │
│ （Modbus/TCP/MQTT）│    │                                    │    │ 阿里云物联网） │
└─────────────────┘    └─────────────────────────────────────┘    └───────────────┘
```

## 📡 支持的协议
### 工业协议
- Modbus RTU/ASCII/TCP
- OPC UA/DA
- DLT645（电力行业专用）
- BACnet/IP

### 物联网协议
- MQTT/MQTT-SN
- CoAP
- HTTP/HTTPS
- TCP/UDP（自定义私有协议）

### 无线协议
- LoRaWAN
- NB-IoT
- Bluetooth 5.0/BLE
- Zigbee

## 🚀 快速开始
### 环境依赖
- **硬件**：x86/ARM 架构设备（如树莓派 4、Jetson Nano、工业工控机）
- **软件**：
  - Python 3.8+（核心模块为 C++ 版本需 C++ 17+）
  - Docker（可选，容器化部署）
  - 依赖包：详见 `requirements.txt`（Python 版本）或 `CMakeLists.txt`（C++ 版本）

### 安装部署
#### 方式 1：从 GitHub 克隆源码
```bash
# 克隆仓库
git clone https://github.com/exiey/IoT_Intelligent_Gateway.git
cd IoT_Intelligent_Gateway

# 安装 Python 依赖（Python 版本）
pip install -r requirements.txt

# 编译 C++ 核心模块（C++ 版本）
mkdir build && cd build
cmake .. && make
```

#### 方式 2：Docker 容器部署
```bash
# 构建 Docker 镜像
docker build -t iot-gateway:latest .

# 启动容器
docker run -d --name iot-gateway -p 1883:1883 -p 502:502 --privileged iot-gateway:latest
```

### 配置说明
1. 复制示例配置文件：
   ```bash
   cp config/sample_config.yaml config/config.yaml
   ```
2. 编辑 `config/config.yaml`，配置以下内容：
   - 设备连接参数（如 Modbus RTU 串口、MQTT 代理地址）
   - 协议转换规则
   - 边缘计算规则（如数据过滤阈值）
   - 云平台认证信息（如 AWS IoT 端点、访问密钥）

配置示例片段（MQTT ↔ Modbus 互通）：
```yaml
device:
  - name: "modbus_sensor_01"
    protocol: "modbus_tcp"
    ip: "192.168.1.100"
    port: 502
    registers:
      - address: 0
        type: "holding"
        interval: 5  # 轮询间隔（秒）
cloud:
  mqtt_broker: "mqtt.eclipseprojects.io"
  port: 1883
  topic: "iot/gateway/data"
  qos: 1
protocol_conversion:
  - source: "modbus_sensor_01"
    target: "mqtt_cloud"
    mapping:
      "temperature": "register_0"
```

### 启动网关
#### Python 版本
```bash
python main.py --config config/config.yaml
```

#### C++ 版本
```bash
# 进入编译后的 build 目录
cd build
./iot_gateway --config ../config/config.yaml
```

#### Docker 容器版本
```bash
# 查看容器运行状态
docker logs iot-gateway

# 重新加载配置（如需）
docker restart iot-gateway
```

## 📂 目录结构
```
IoT_Intelligent_Gateway/
├── build/              # C++ 编译目录（手动创建）
├── config/             # 配置文件目录
│   ├── sample_config.yaml  # 示例配置
│   └── config.yaml     # 主配置文件
├── docs/               # 文档目录
├── examples/           # 使用示例（如设备对接、协议转换demo）
├── src/                # 核心源码
│   ├── device/         # 设备接入模块
│   ├── protocol/       # 协议解析/转换模块
│   ├── edge/           # 边缘计算模块
│   ├── cloud/          # 云端对接模块
│   └── utils/          # 工具类
├── tests/              # 单元测试/集成测试
├── Dockerfile          # Docker 构建文件
├── requirements.txt    # Python 依赖
├── CMakeLists.txt      # C++ 编译配置
└── main.py             # Python 版本入口
```

## 📝 使用示例
### 示例 1：Modbus 传感器数据上报至 MQTT 云平台
1. 配置 `config/config.yaml` 中 Modbus 设备地址、寄存器信息；
2. 配置 MQTT 云平台地址、主题；
3. 启动网关后，执行以下命令查看上报数据：
   ```bash
   mosquitto_sub -h mqtt.eclipseprojects.io -t "iot/gateway/data"
   ```

### 示例 2：边缘端数据过滤
在配置文件中设置温度阈值，仅当传感器温度超过 30℃ 时才上报数据：
```yaml
edge_computing:
  - device: "modbus_sensor_01"
    rule: "temperature > 30"
    action: "upload_to_cloud"
```

## 🤝 贡献指南
1. Fork 本仓库；
2. 创建特性分支（`git checkout -b feature/AmazingFeature`）；
3. 提交代码（`git commit -m 'Add some AmazingFeature'`）；
4. 推送至分支（`git push origin feature/AmazingFeature`）；
5. 打开 Pull Request。

## 📄 许可证
本项目采用 [MIT 许可证](LICENSE) 开源 - 详见 LICENSE 文件。

## 📞 联系我们
- 项目维护者：exiey
- 问题反馈：提交 [Issues](https://github.com/exiey/IoT_Intelligent_Gateway/issues)
- 邮件沟通：xxx@example.com（可替换为实际邮箱）

---
⭐ 如果该项目对你有帮助，欢迎点 Star 支持！
