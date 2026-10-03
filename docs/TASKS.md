# myserver 任务列表（按组件拆分）

> 规则：一次只做一个组件，做完可编译可测试才勾选。顺序自上而下，后面的依赖前面的接口。

## C0 构建基线（已完成）
- [x] `.clang-format / .clang-tidy / vcpkg.json / CMakeLists / Presets`

## C1 common 基础组件
- [ ] C1.1 Config：yaml/env 解析，端口/核数/Tick/队列上限，环境变量覆盖
- [ ] C1.2 Logger：spdlog 异步封装，分级 + session_id/room_id 字段
- [ ] C1.3 Metrics：prometheus 指标，在线数/延迟/QPS/Tick P99
- [ ] C1.4 Executor：io_context 池 + strand 工厂 + 业务线程池
- [ ] 验收：单测覆盖配置解析，demo 打印指标

## C2 net 网络组件
- [ ] C2.1 Buffer：可复用收发 Buffer，上限 1MB 校验
- [ ] C2.2 Codec：按 PROTOCOL.md 编解码 `[len][ver][msg_id][payload]`
- [ ] C2.3 Acceptor：多 Reactor 轮询分发，停服时先停 Accept
- [ ] C2.4 Session：awaitable 读写 + 心跳 30s + 写队列 1024 背压 + shared_from_this 保活
- [ ] C2.5 Shutdown：SIGTERM 优雅关闭 5s 预算
- [ ] 验收：echo 压测 + 慢连接背压单测 + TSan 全绿

## C3 room 房间组件
- [ ] C3.1 RoomState：玩家加入/离开/快照数据结构
- [ ] C3.2 TickLoop：30Hz steady_timer 驱动，批量处理输入
- [ ] C3.3 Room：固定 strand + 无锁输入队列 + 广播合并
- [ ] C3.4 RoomManager：房间创建/销毁/查找
- [ ] C3.5 Matchmaker：匹配队列 + 组队建房
- [ ] 验收：2 房 x 4 人模拟对战，Tick P99 < 33ms

## C4 gateway 接入组件
- [ ] C4.1 Hello 鉴权：token 校验，未鉴权禁 Input
- [ ] C4.2 限流：Input 60/s，超限 warn + 断连
- [ ] C4.3 路由：Session -> Room 投递，拒绝直改状态
- [ ] 验收：非法包/旧版本/超速输入全部正确拒绝

## C5 ops 运维组件
- [ ] C5.1 Dockerfile 多阶段构建 + docker-compose 本地编排
- [ ] C5.2 bench 机器人：10k 连接 + 对战流量回放
- [ ] C5.3 压测报告 + SLO 验收

## SLO 验收标准（C5 必须达标）
- 单机连接数：>= 10000 长连接稳定 10 分钟
- Room Tick P99：< 33ms（30Hz，单 Tick 预算）
- 端到端延迟 P99：< 100ms（同机房）
- 掉线率：< 0.1% / 小时
- 内存：10k 连接常驻 < 1GB
- 优雅关闭：SIGTERM 后 5s 内完成，无数据丢失
- TSan/ASan 全绿，无 data race 与泄漏
