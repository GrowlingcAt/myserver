# 项目规范

- 项目目录仅为 `/home/xxx/code/myserver`，不碰同级其他目录。
- 开工必读 Skill：
  - `.cursor/skills/cpp20-asio-server/SKILL.md`
  - `.cursor/skills/incremental-dev/SKILL.md`
- 标准 C++20，CMake + Ninja，`-Wall -Wextra -Werror`。
- 网络统一用 `asio::awaitable`，禁用裸 detach，必须可取消。
- Session / Room 绑定 strand，会话与房间内不加 mutex。
- 新功能必须带测试 + 日志 + 指标。
- 代码规范见 `docs/CODE_STANDARD.md`，其中【强制】条目必须遵守。
- 协议见 `docs/PROTOCOL.md`，构建基线用 `vcpkg.json + CMakePresets.json`，格式化用 `.clang-format`。

## 迭代纪律【强制】

- 接口先行：改接口先改 `docs/PROTOCOL.md` / 头文件注释，再改实现，不允许实现与文档不一致合入。
- 向后兼容：`msg_id` 只增不改，废弃 id 永不复用；破坏兼容的变更必须升 `ver` 并写迁移说明。
- 小步合入：一次只动一个 `TASKS.md` 子项（如 C2.2），不允许跨组件顺手重构；跨组件改动必须拆成多轮。
- 回归门禁：改动所在组件单测 + `ctest` 全过 + TSan/ASan 无新增告警，否则不勾选完成。
