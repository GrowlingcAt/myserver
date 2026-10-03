---
name: incremental-dev
description: 多轮增量开发工作流：计划驱动、每轮只做一件事、更新任务与计划、可验收交付。长周期服务器开发时使用。
---

# 增量开发工作流

## 适用范围
myserver 项目的长周期、多轮对话开发。目标：不断点、不返工。

## 1. 开工必读
- 先读 `.cursor/skills/cpp20-asio-server/SKILL.md` 技术规范。
- 再读 `docs/TASKS.md`，只做下一个未完成项，不跳步、不扩范围。

## 2. 每轮只做一件事
- 一轮 = 一个 TASKS 项，例如"阶段1 echo 验证"。
- 必须做到可编译、可运行、可验证才算完成。
- 不做计划外的重构与优化。

## 3. 项目边界
- 只动 `/home/xxx/code/myserver`，不碰 `test`、`the-game-server` 等其他目录。

## 4. 收尾动作
- 更新 `docs/TASKS.md` 勾选完成项。
- 给出下一轮入口，例如"下一轮：Session + Codec"。
- 停下等待用户确认，不擅自开新阶段。
