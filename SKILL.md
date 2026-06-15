---
name: loop-go
description: |
  loop-go — GitHub 项目全链路处理入口。
  触发词: loop-go <GitHub-URL>
  子循环: loop-go-exec (#648-1)
version: 1.0.0
created: 2026-06-14
status: 正式
number: 648
---

# loop-go — GitHub 项目全链路处理

> **Skill ID**: 648
> **入口触发词**: `loop-go <GitHub-URL>`

## 一句话

给一个 GitHub 链接，AI 自动跑完：clone → 测试 → 写介绍 → 存 Wiki → 自检验收 → 人味儿沉淀。

## 子循环

| 子编号 | 名称 | 定位 |
|--------|------|------|
| **loop-go-exec** | GitHub 项目完整处理 loop | 执行层 |

## 触发方式

```
loop-go https://github.com/owner/repo
```

## 与 loop-go-exec 的关系

- 本文件：入口定义（触发词 + 子循环索引）
- loop-go-exec：完整执行层（clone → 测试 → Wiki → 自检 → CE → 复盘 → handoff → 问 cron）

## 入口 Agent 行为

收到 `loop-go <GitHub-URL>` 时：
1. 读取 `loop-go-exec/SKILL.md`
2. 启动 loop-go-exec 子循环
3. 监督子循环完整走完
4. 全链路走完才报告完成

## 设计原则

- **双 Agent 分离**：maker agent（执行）+ checker agent（验收），不能是同一个
- **真实优先**：测试结果必须是实际执行的，不是"看起来没问题"
- **全链路闭环**：8 步全部走完才算完成，只跑一半不算

## 下一步

See `loop-go-exec/` for the execution layer.
