# myserver 代码规范（必须遵守）

> 适用范围：`myserver` 下所有 C++ 代码。违反以下带【强制】的条目，CI 直接失败。

## 1. 基础与构建【强制】

- 标准 C++20，CMake >= 3.20，Ninja，`-Wall -Wextra -Werror`。
- 禁止 C 风格转换，统一 `static_cast / dynamic_cast`，禁止裸 `new/delete`（除底层 Buffer 池）。
- 头文件用 `#pragma once`，禁止 `using namespace` 出现在头文件。
- 所有公开 API 必须有注释说明线程安全性：`线程安全 / 仅 strand 内调用 / 仅 Tick 调用`。

## 2. 命名与目录【强制】

- 目录：`src/net/ src/room/ src/common/ tests/ bench/`，不允许跨层直接依赖（`room` 不得直接调 `net` 细节，必须经接口）。
- 命名空间：`myserver::net / myserver::room / myserver::common`，测试 `myserver::tests`，禁止 `using namespace` 污染全局，`.cc` 内可用 `namespace net = myserver::net` 别名。

### 2.1 文件命名
- 全部 `snake_case`：`room_manager.h / room_manager.cc`，测试 `xxx_test.cc`，bench `xxx_bench.cc`。
- 头实现必须同名，禁止一个 `.cc` 对多个 `.h`，禁止 `util.h / common.h` 这种大杂烩名。

### 2.2 类型命名（PascalCase）
- 类/结构体/枚举/Concept：`Session / RoomManager / TickLoop / ErrorCode`。
- 协程返回统一 `asio::awaitable<T>`，函数名以 `async_` 开头：`async_run / async_read_loop`，与同步函数区分。
- 接口类以 `I` 开头：`ICodec / IRoomSink`，实现类不加前后缀。

### 2.3 函数与变量（snake_case）
- 函数/局部变量/参数：`start_tick / session_id / room_count`，布尔函数以 `is_/has_/should_` 开头：`is_running / has_room`。
- 成员变量后缀 `_`：`strand_ / timer_ / write_queue_`，静态成员 `s_` 开头，常量 `k` 开头：`kTickHz / kMaxQueueSize`。
- 枚举值全大写下划线：`MsgId::C2S_INPUT / Error::TIMEOUT`，命名空间内常量 `kPascalCase` 仅限编译期配置。

### 2.4 并发相关后缀（本项目特有）
- 仅 strand 内调用的函数后缀 `_in_strand` 可省略，但必须在注释写 `仅 strand 内调用`。
- 跨 strand 投递函数以 `post_` 开头：`post_input / post_broadcast`，一看就知道是异步投递。
- Tick 内函数以 `on_tick_` 开头：`on_tick_update / on_tick_broadcast`，禁止在 Tick 外调用。

### 2.5 禁止项
- 禁止拼音、单字母（循环变量 `i` 除外）、`data1 / tmp2` 无意义名。
- 禁止与 std 冲突：`thread / mutex / future / expected` 不得做变量名。

## 3. Asio 协程与线程模型【强制】

- 网络异步统一 `asio::awaitable<void>`，禁止回调式 `async_xxx` 裸写新代码。
- 禁止裸 `detach` / `co_spawn` 无追踪：必须绑定 `cancellation_signal`，支持超时取消。
- 每个 `Session` / `Room` 绑定固定 `strand`，strand 内部禁止加 `mutex`。
- 跨 strand 通信只允许无锁队列 + `asio::post / dispatch` 投递，禁止共享裸指针。
- 禁止在 IO 线程做阻塞操作：磁盘、加锁等待、耗时计算必须投递到业务池。
- `Session` 生命周期用 `shared_ptr + enable_shared_from_this`，协程内先 `auto self = shared_from_this()` 保活。

## 4. 内存与性能

- 热路径禁止异常做流程控制，禁止 `std::endl`，用 `'\n'`。
- 网络收发统一用 `flat_buffer / 固定 Buffer 池`，禁止每包 `new string`。
- Tick 内禁止分配：Snapshot 复用对象，广播合并后再写。
- 拷贝优先移动：大对象函数参数用 `const&` 入、`std::move` 出，返回大对象依赖 NRVO/RVO。

## 5. 错误处理与可观测【强制】

- 协程必须处理 `asio::error::operation_aborted`（取消）与 `eof`，不允许吞异常。
- 错误码统一 `std::error_code / expected` 风格，禁止 `bool` 表示多种失败。
- 新功能必须带：`spdlog` 关键日志 + prometheus 指标 + 单元测试，三者缺一不可。
- 日志分级：`debug` 只放 Tick 外，`info` 放建房/匹配/上下线，`warn/error` 必须带 `session_id / room_id`。

## 6. 并发与 Room Tick【强制】

- Tick 固定 30Hz，用 `steady_timer + co_await`，Tick 内只做批量状态推进。
- 输入路径：`Session -> 无锁队列 -> Room Tick`，禁止 Session 直接改 Room 状态。
- 背压：单连接写队列上限 1024，超限断连；房间广播队列满则丢旧快照保 Tick，不阻塞。
- 优雅关闭顺序：停 Accept -> 通知 Room 停 Tick -> drain 写队列 -> 停 `io_context`。

## 7. 测试与提交

- 单元测试用 GTest，路径 `tests/*_test.cc`，覆盖 Codec / Room 状态机 / 背压边界。
- 提交前本地必须过：`cmake --build + ctest + clang-format --dry-run`。
