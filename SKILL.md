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

## 验收标准

| 分值 | 判定 | 动作 |
|------|------|------|
| 8-10 | ✅ 通过 | 进入下一步 |
| 5-7 | ⚠️ 修复项 | 列出问题，打回 maker |
| 0-4 | 🔴 失败 | 从头重跑 |

## 自检清单（Step 6）

1. clone 真的跑了吗？
2. 测试命令真的执行了吗？
3. handoff 写了吗？
4. CE Review 三个字段填全了吗？
5. 自检诚实吗？
6. 下次要改什么？

## 下一步

See `loop-go-exec/SKILL.md` for the full execution layer.
