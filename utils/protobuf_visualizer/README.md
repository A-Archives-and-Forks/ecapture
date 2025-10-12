# eCapture Protobuf Message Visualizer

一个用于可视化和调试 eCapture WebSocket 中 Protobuf 消息的工具。

## 功能特性

- 🎨 **彩色输出** - 使用 ANSI 颜色代码美化显示
- 📦 **事件可视化** - 详细显示捕获的网络事件
- 💓 **心跳监控** - 实时显示 WebSocket 心跳包
- 📋 **日志跟踪** - 捕获并显示进程日志
- 🔍 **多种格式** - 支持文本和十六进制格式显示 Payload
- 📊 **统计信息** - 退出时显示接收消息的统计

## 编译

```bash
go build -o pb_debugger pb_debugger.go
```

## 使用方法

### 基本用法

```bash
# 连接到默认的 WebSocket 服务器 (ws://127.0.0.1:28257)
./pb_debugger

# 指定自定义 WebSocket URL
./pb_debugger -url ws://192.168.1.100:28257
```

### 命令行参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `-url` | `ws://127.0.0.1:28257` | WebSocket 服务器 URL |
| `-hex` | `false` | 以十六进制格式显示 Payload |
| `-max-payload` | `1024` | 显示的最大 Payload 字节数 |
| `-no-color` | `false` | 禁用彩色输出 |
| `-compact` | `false` | 紧凑输出模式（单行显示） |

### 使用示例

#### 1. 标准详细模式

```bash
./pb_debugger
```

输出示例：
```
╔══════════════════════════════════════════════════════════════════════════════╗
║                                                                              ║
║                    eCapture Protobuf Message Visualizer                      ║
║                                                                              ║
║                         WebSocket Debugging Tool                             ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
Connected to: ws://127.0.0.1:28257
Listening for messages... (Press Ctrl+C to quit)

────────────────────────────────────────────────────────────────────────────────

┌─── 📦 EVENT ───────────────────────────────────────────────────────────
│ ▶ Metadata:
│   Sequence: #1
│   Timestamp: 2025-10-10 14:30:45
│   UUID: 12345_67890_curl
│ ▶ Process:
│   PID: 12345
│   Process Name: curl
│ ▶ Network:
│   Connection: 192.168.1.100:54321 → 93.184.216.34:443
│ ▶ Event Details:
│   Type: 0 (Send/Write)
│   Length: 517 bytes
│ ▶ Payload:
│   GET /index.html HTTP/1.1
│   Host: example.com
│   ...
└────────────────────────────────────────────────────────────────────────
```

#### 2. 紧凑模式

```bash
./pb_debugger -compact
```

输出示例：
```
[14:30:45] 💓 HEARTBEAT #1 heartbeat:1
[14:30:46] 📋 LOG Starting capture...
[14:30:47] 📦 EVENT #1 PID:12345 192.168.1.100:54321 → 93.184.216.34:443 [517 bytes]
[14:30:48] 📦 EVENT #2 PID:12345 192.168.1.100:54321 → 93.184.216.34:443 [1024 bytes]
```

#### 3. 十六进制模式

```bash
./pb_debugger -hex
```

显示 Payload 的完整十六进制转储。

#### 4. 无颜色模式（用于日志文件）

```bash
./pb_debugger -no-color > capture.log
```

#### 5. 限制 Payload 大小

```bash
./pb_debugger -max-payload 256
```

只显示前 256 字节的 Payload。

## 消息类型

工具支持三种 Protobuf 消息类型：

### 1. 心跳包 (Heartbeat)
- **LogType**: `LOG_TYPE_HEARTBEAT` (0)
- **图标**: 💓
- **颜色**: 紫色
- **内容**: 包含时间戳、计数和消息

### 2. 进程日志 (Process Log)
- **LogType**: `LOG_TYPE_PROCESS_LOG` (1)
- **图标**: 📋
- **颜色**: 绿色
- **内容**: eCapture 运行时的文本日志

### 3. 事件 (Event)
- **LogType**: `LOG_TYPE_EVENT` (2)
- **图标**: 📦
- **颜色**: 蓝色
- **内容**: 捕获的网络事件，包含：
  - 时间戳和 UUID
  - 进程信息（PID、进程名）
  - 网络连接信息（源/目标 IP 和端口）
  - 事件类型和数据长度
  - Payload 数据

## Protobuf 结构

### LogEntry
```protobuf
message LogEntry {
  LogType log_type = 1;
  oneof payload {
    Event event_payload = 2;
    Heartbeat heartbeat_payload = 3;
    string run_log = 4;
  }
}
```

### Event
```protobuf
message Event {
  int64 timestamp = 1;
  string uuid = 2;
  string src_ip = 3;
  uint32 src_port = 4;
  string dst_ip = 5;
  uint32 dst_port = 6;
  int64 pid = 7;
  string pname = 8;
  uint32 type = 9;
  uint32 length = 10;
  bytes payload = 11;
}
```

### Heartbeat
```protobuf
message Heartbeat {
  int64 timestamp = 1;
  int64 count = 2;
  string message = 3;
}
```

## 输出示例

### 完整事件显示

```
┌─── 📦 EVENT ───────────────────────────────────────────────────────────
│ ▶ Metadata:
│   Sequence: #42
│   Timestamp: 2025-10-10 14:30:45
│   UUID: 12345_67890_curl
│ ▶ Process:
│   PID: 12345
│   Process Name: curl
│ ▶ Network:
│   Connection: 192.168.1.100:54321 → 93.184.216.34:443
│ ▶ Event Details:
│   Type: 0 (Send/Write)
│   Length: 517 bytes
│ ▶ Payload:
│   GET /index.html HTTP/1.1
│   Host: example.com
│   User-Agent: curl/7.68.0
│   Accept: */*
│
└────────────────────────────────────────────────────────────────────────
```

### 统计信息

```
────────────────────────────────────────────────────────────────────────────────
Statistics:
  Events received: 156
  Heartbeats received: 12
  Logs received: 8
────────────────────────────────────────────────────────────────────────────────
```

## 颜色说明

- 🔵 **蓝色** - 事件 (Event)
- 🟣 **紫色** - 心跳 (Heartbeat)
- 🟢 **绿色** - 日志 (Log)
- 🟡 **黄色** - 字段名称
- ⚪ **白色** - 字段值
- ⚫ **灰色** - 辅助信息

## 故障排除

### 无法连接到 WebSocket 服务器

确保 eCapture 正在运行并启用了 WebSocket 服务器：

```bash
ecapture tls -w ws --ws-addr 127.0.0.1:28257
```

### 消息解析失败

确保 eCapture 和可视化工具使用相同版本的 Protobuf 定义。

### 性能问题

如果事件太多导致输出过快：
- 使用 `-compact` 模式
- 使用 `-max-payload` 限制 Payload 大小
- 将输出重定向到文件：`./pb_debugger > capture.log`

## 依赖项

- `golang.org/x/net/websocket` - WebSocket 客户端
- `google.golang.org/protobuf` - Protobuf 序列化/反序列化
- `github.com/gojue/ecapture/protobuf/gen/v1` - eCapture Protobuf 定义

## 许可证

Apache License 2.0

## 作者

eCapture Project Contributors
