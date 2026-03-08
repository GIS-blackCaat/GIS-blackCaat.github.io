---
layout: post
title: "钉钉 Stream 模式图片接收问题 - 媒体文件发送协议详解与踩坑"
date: 2026-03-08
categories: [OpenClaw, 钉钉，媒体文件]
tags: [dingtalk, image, media, bugfix, 钉钉集成，图片发送]
excerpt: "详解钉钉渠道中图片/文件发送的媒体标签协议使用规范。包含标签格式、常见错误、解决方案和最佳实践，帮助开发者正确发送媒体文件。"
---

# 📸 钉钉 Stream 模式图片接收问题总结

**日期**: 2026 年 3 月 8 日  
**分类**: OpenClaw / 钉钉集成 / 媒体文件  
**问题级别**: 🟡 中优先级

---

## 📋 问题概述

在钉钉渠道中，AI 生成的图片或文件无法正确发送给用户，用户收不到媒体附件。

---

## 🔍 问题现象

### 现象 1：图片未发送

AI 回复：
```
图片已生成，请查收。
```

**结果**: 用户只看到文字，没有收到图片。

### 现象 2：标签格式错误

AI 回复：
```
[DING:IMAGE /tmp/image.png]
```

**结果**: 标签被当作普通文本发出，用户看到原始标签字符串。

### 现象 3：远程 URL 直接使用

AI 回复：
```
[DING:IMAGE https://example.com/image.png]
```

**结果**: 解析失败，图片未发送。

---

## 🧐 原因分析

### 根本原因 1：未使用媒体标签

在钉钉渠道，**要把图片/文件真正发给用户**，必须在回复里输出媒体标签。系统会识别标签、上传本地文件，然后把媒体发送给用户。

只说"已发送"但不输出标签 → 用户收不到任何媒体。

### 根本原因 2：标签格式错误

媒体标签格式非常严格，常见错误：

| 错误写法 | 问题 |
|----------|------|
| `[DING:IMAGE /tmp/test.png]` | 缺少 `path="..."` |
| `[DING:IMAGE path=/tmp/test.png]` | 缺少双引号 |
| `[DING:IMAGE path='/tmp/test.png']` | 用了单引号 |
| `[DING:IMAGE path="/tmp/test.png" ]` | `]` 前多了空格 |

### 根本原因 3：远程 URL 未下载

标签的 `path` 只允许本地绝对路径，`http/https` 远程地址不能直接放进标签。

---

## ✅ 解决方案

### 正确的标签格式

#### 发送图片
```
[DING:IMAGE path="/absolute/path/to/image.png"]
```

#### 发送文件
```
[DING:FILE path="/absolute/path/to/file.pdf" name="用户看到的文件名.pdf"]
```

### 完整工作流程

1. **确保文件落盘**
   - 已存在：直接进入第 2 步
   - 需要生成：先把文件写到磁盘
   - 如果只有远程 URL：先下载到本地临时文件（推荐 `/tmp/...`）

2. **检查基本约束**
   - 路径是绝对路径（不要用 `~`、相对路径）
   - 文件大小 ≤ 20MB

3. **在回复正文里输出标签**
   - 图片用 `[DING:IMAGE ...]`
   - 文件用 `[DING:FILE ...]`

4. **可一次发多个**
   - 同一条回复里放多个标签即可

### 正确示例

#### 示例 1：发送单张图片
```
销售趋势图已生成，请查收：
[DING:IMAGE path="/tmp/sales_trend_2026-03-08.png"]
```

#### 示例 2：图表 + 报告一起发
```
分析完成，请查收：

[DING:IMAGE path="/tmp/chart.png"]

[DING:FILE path="/tmp/report.pdf" name="分析报告.pdf"]
```

#### 示例 3：远程图片处理后发送
```bash
# 先下载远程图片
curl -o /tmp/temp_image.png "https://example.com/image.png"

# 然后输出标签
[DING:IMAGE path="/tmp/temp_image.png"]
```

---

## 📝 参数说明

| 参数 | 必填 | 说明 |
|------|------|------|
| `path` | ✅ | 本地绝对路径，必须用双引号 |
| `name` | ❌ | 可选，文件显示名（图片也可作为标题/alt） |

### 路径规则

- ✅ `/tmp/image.png`
- ✅ `/root/.openclaw/workspace/report.pdf`
- ❌ `~/Downloads/a.pdf`（`~` 可能找不到）
- ❌ `a.pdf`（相对路径）
- ❌ `https://example.com/a.png`（远程 URL）

---

## 🚫 常见错误检查清单

- [ ] 标签格式是否正确（`path="..."` 双引号）
- [ ] 路径是否为绝对路径
- [ ] 文件是否已落盘存在
- [ ] 文件大小是否 ≤ 20MB
- [ ] 是否没有用 Markdown 图片语法 `![alt](...)`
- [ ] 是否没有把远程 URL 直接塞进标签
- [ ] 路径是否没有做转义/编码（直接写原始路径）

---

## 🔧 调试技巧

### 检查文件是否存在
```bash
ls -lh /tmp/image.png
```

### 检查标签格式
确保标签完整且无多余空格：
```
[DING:IMAGE path="/tmp/image.png"]
```

### 测试发送
创建一个测试图片：
```bash
echo "test" > /tmp/test.txt
```

然后在回复中输出：
```
测试文件发送：
[DING:FILE path="/tmp/test.txt" name="测试文件.txt"]
```

---

## 📚 参考资料

- [dingtalk-send-media Skill 文档](https://github.com/GIS-blackCaat/chat-memory-keeper/tree/main/skills/dingtalk-send-media)
- [OpenClaw 钉钉扩展](https://github.com/openclaw/openclaw/tree/main/extensions/dingtalk)

---

## 💡 最佳实践

1. **统一使用 `/tmp/` 存放临时文件**
2. **生成文件后立即输出标签**，不要延迟
3. **标签放在回复末尾**，确保用户先看到说明文字
4. **多个文件时分开发送**，避免单个消息过大

---

*最后更新：2026 年 3 月 8 日 · 黑猫 🐈‍⬛*
