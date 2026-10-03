---
name: cpp20-asio-server
description: C++20高性能实时对战服务器规范：Asio协程、Reactor多线程、Room Tick、背压与生产要求。涉及网络会话房间设计时使用。
---

# C++20 Asio 对战服务器规范

## 1. 技术约束
- 标准 C++20，CMake >= 3.20，`-Wall -Wextra -Werror`。
- 网络库：standalone Asio（或 Boost.Asio），统一用 `asio::awaitable<void>` 协程。
- 禁止裸 `detach`，所有 `co_spawn` 必须绑定 `cancellation_signal`，支持超时取消。
- 日志 `spdlog` 异步，指标 `prometheus-cpp`，序列化 `protobuf`。

## 2. 线程与执行模型
- Reactor 数 = `hardware_concurrency`，每个 `io_context` 跑一个线程。
- Acceptor 轮询分发新连接到各 Reactor。
- 每个 Session 绑定一个 `strand`，会话内串行，禁止在会话内加 mutex。
- 每个 Room 绑定固定 `strand`，不迁移；跨房间通信用无锁队列投递。
- 耗时计算投递到业务线程池，禁止阻塞 IO 线程。

## 3. 网络会话规范
- 协议：`[4字节len][payload]`，`Codec` 抽象编解码，首版 protobuf。
- 协程读写：`co_await async_read/async_write`，读超时 + 心跳，超时踢除。
- 背压：每连接写队列上限 1024，超限断连；广播合并，避免慢连接拖垮 Tick。
- 关闭：处理 `SIGTERM`，先停 Accept，再 drain 会话队列，最后停 `io_context`。

## 4. Room Tick 规范
- 固定 Tick 30Hz：`co_await steady_timer` 驱动。
- 输入 -> 队列 -> Tick 内批量处理 -> 生成 Snapshot -> 合并广播。
- 协议类型：`C2S_Input / S2C_Snapshot`，版本号兼容。
- Matchmaker 独立模块，只做排队与建房，不碰 Tick 逻辑。

## 5. 生产要求
- 每个功能必须带：单元测试 + 日志 + 指标（在线数、延迟、QPS、Tick P99）。
- 目录：`src/net/ src/room/ src/common/ tests/ bench/`。
- Dockerfile 多阶段构建，bench 给出压测报告。
