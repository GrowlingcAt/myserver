# myserver 网络协议（v1）

## 1. 帧格式

```text
[4字节len BE][1字节ver][2字节msg_id BE][payload]
```

- `len` = 其后所有字节数，含 ver + msg_id + payload，`ver` 固定 `1`。
- `payload` 为 protobuf 序列化体，Codec 先校验 len 上限 1MB，超限直接断连。

## 2. 版本兼容

- ver 不一致直接拒绝并返回 `S2C_Reject{reason="version_mismatch"}` 后断连。
- msg_id 新增只增不改，废弃 id 永不复用。

## 3. 消息表（首版）

| 方向 | msg_id | 说明 |
|------|--------|------|
| C2S | 1 | `Hello{token, client_ver}` 鉴权入口 |
| S2C | 2 | `Welcome{session_id}` 或 `Reject` |
| 双向 | 3 | `Ping / Pong` 心跳，30s 超时踢除 |
| C2S | 10 | `Input{seq, dx, dy, buttons}` |
| S2C | 11 | `Snapshot{tick, state[]}` 广播 |
| S2C | 12 | `RoomEvent{join/leave}` |

## 4. 安全与限流

- 鉴权在 Hello 完成前不接受 Input，心跳前置。
- 单连接 Input 限流 60/s，超限计数 + warn，连续超限断连。
