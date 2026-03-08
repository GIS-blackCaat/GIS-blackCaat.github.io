---
layout: post
title: "定时任务异常发布问题总结 - OpenClaw 钉钉渠道 Cron Job 踩坑记录"
date: 2026-03-08
categories: [OpenClaw, 钉钉，定时任务]
tags: [cron, dingtalk, bugfix, openclaw, 定时任务，钉钉集成]
excerpt: "详细记录 OpenClaw 钉钉渠道定时任务创建失败、消息未投递等问题的排查过程和解决方案。包含完整命令模板、参数说明和调试技巧。"
---

# ⏰ 定时任务异常发布问题总结

**日期**: 2026 年 3 月 8 日  
**分类**: OpenClaw / 钉钉集成 / 定时任务  
**问题级别**: 🔴 高优先级

---

## 📋 问题概述

在使用 OpenClaw 钉钉渠道创建定时提醒任务时，遇到任务创建成功但实际不执行、或执行后消息未正确投递的问题。

---

## 🔍 问题现象

### 现象 1：任务创建成功但无消息发出

```bash
# 创建命令
openclaw cron add \
  --name "喝水提醒" \
  --at "20m" \
  --session isolated \
  --message "提醒用户：该喝水了" \
  --channel "clawdbot-dingtalk" \
  --to "manager9140"
```

**结果**: 任务显示创建成功，但 20 分钟后用户未收到钉钉消息。

### 现象 2：任务执行报错

```
Error: Action send requires a target.
```

---

## 🧐 原因分析

### 根本原因 1：`--message` 参数格式错误

**错误写法**:
```bash
--message "提醒用户：该喝水了"
```

**问题**: `--message` 的内容会作为 prompt 发给一个隔离的 AI session。该 session 会误认为需要调用 `message` 工具主动发送，导致因缺少 `target` 参数而报错。

### 根本原因 2：缺少 `--announce` 参数

隔离任务 (`--session isolated`) 默认不会自动投递消息到渠道，需要显式声明 `--announce`。

### 根本原因 3：对一次性任务执行 `cron run`

```bash
# ❌ 错误操作
openclaw cron run <job-id>
```

**问题**: 该命令会立即执行任务并触发 `deleteAfterRun`，导致任务在定时时间到达前被删除。

---

## ✅ 解决方案

### 正确的命令模板

```bash
openclaw cron add \
  --name "<任务名>" \
  --at "<时间>" \
  --session isolated \
  --message "直接输出以下内容（禁止调用工具）：<具体提醒内容>" \
  --announce \
  --channel "clawdbot-dingtalk" \
  --to "<用户 staffId>"
```

### 关键参数说明

| 参数 | 正确值 | 说明 |
|------|--------|------|
| `--session` | `isolated` | 不可用 `main`，否则消息可能丢失 |
| `--message` | `直接输出以下内容（禁止调用工具）：...` | 固定前缀，防止 AI 误调用工具 |
| `--announce` | 必须添加 | 显式声明投递意图 |
| `--channel` | `clawdbot-dingtalk` | 钉钉渠道固定值 |
| `--at` | 带时区的 ISO8601 或相对时间 | 如 `2026-03-08T15:00:00+08:00` 或 `20m` |

### 正确示例

```bash
# ✅ 一次性提醒（相对时间）
openclaw cron add \
  --name "喝水提醒" \
  --at "20m" \
  --session isolated \
  --message "直接输出以下内容（禁止调用工具）：该喝水了！保持水分很重要。" \
  --announce \
  --channel "clawdbot-dingtalk" \
  --to "manager9140"

# ✅ 循环提醒
openclaw cron add \
  --name "定时休息提醒" \
  --cron "0 */2 * * *" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "直接输出以下内容（禁止调用工具）：已经过去 2 小时了，该休息一下眼睛和身体。" \
  --announce \
  --channel "clawdbot-dingtalk" \
  --to "manager9140"
```

---

## 🛠️ 调试命令

```bash
# 查看所有任务
openclaw cron list

# 查看运行记录
openclaw cron runs --id <job-id> --limit 5

# 删除任务
openclaw cron rm <job-id>

# ⚠️ 仅限循环任务测试，严禁用于一次性 --at 任务
openclaw cron run <job-id> --force --expect-final
```

---

## 📝 检查清单

创建任务前确认：

- [ ] `--session` 是 `isolated` 不是 `main`
- [ ] `--message` 以"直接输出以下内容（禁止调用工具）"开头
- [ ] 使用了 `--announce`
- [ ] `--channel` 是 `clawdbot-dingtalk`
- [ ] `--to` 是正确的 senderStaffId
- [ ] 绝对时间带时区（或 `--cron` 配合 `--tz "Asia/Shanghai"`）
- [ ] **不对一次性 `--at` 任务执行 `cron run`**

---

## 📚 参考资料

- [dingtalk-cron-job Skill 文档](https://github.com/GIS-blackCaat/chat-memory-keeper/tree/main/skills/dingtalk-cron-job)
- [OpenClaw Cron 官方文档](https://docs.openclaw.ai)

---

*最后更新：2026 年 3 月 8 日 · 黑猫 🐈‍⬛*
