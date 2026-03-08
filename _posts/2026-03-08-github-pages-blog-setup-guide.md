---
layout: post
title: "从零开始在 GitHub Pages 搭建个人博客 - 完整指南与踩坑记录"
date: 2026-03-08
categories: [GitHub, Jekyll, 博客]
tags: [github-pages, jekyll, 博客搭建，教程，踩坑]
excerpt: "详细记录在 GitHub Pages 搭建 Jekyll 博客的全过程，包括配置、调试、常见问题和解决方案。适合想要搭建免费技术博客的开发者。"
---

# 🚀 从零开始在 GitHub Pages 搭建个人博客

**日期**: 2026 年 3 月 8 日  
**分类**: GitHub / Jekyll / 博客搭建  
**阅读时间**: 约 10 分钟

---

## 📋 前言

今天花了几个小时在 GitHub Pages 上搭建了这个技术博客，过程中遇到了不少坑，也学到了很多经验。这篇文章完整记录整个搭建过程，希望能帮到想要搭建免费技术博客的你。

---

## 🎯 为什么选择 GitHub Pages

### 优势

| 优势 | 说明 |
|------|------|
| **免费** | 完全免费，无需购买服务器 |
| **稳定** | GitHub 背书，稳定性有保障 |
| **简单** | 推送代码自动构建部署 |
| **自定义域名** | 支持绑定自己的域名 |
| **HTTPS** | 默认支持 HTTPS 加密 |
| **Jekyll 原生支持** | 无需额外配置，开箱即用 |

### 适用场景

- ✅ 个人技术博客
- ✅ 项目文档
- ✅ 作品集展示
- ✅ 简历网站

### 局限性

- ❌ 不支持后端（纯静态）
- ❌ 每月 100GB 流量限制（个人使用足够）
- ❌ 构建时间 10 分钟左右（大型站点可能更长）

---

## 🛠️ 搭建步骤

### 第一步：创建仓库

```bash
# 仓库命名规则：用户名.github.io
# 例如：GIS-blackCaat.github.io

# 本地初始化
mkdir my-blog
cd my-blog
git init
```

**⚠️ 注意**：仓库名必须是 `用户名.github.io` 格式，否则 GitHub Pages 无法识别。

### 第二步：创建基本结构

```
my-blog/
├── _config.yml          # Jekyll 配置文件
├── index.md             # 首页
├── _layouts/            # 布局模板
│   └── default.html
├── _posts/              # 博客文章
│   └── 2026-03-08-hello-world.md
├── assets/              # 静态资源
│   ├── css/
│   └── js/
└── .gitignore
```

### 第三步：配置 _config.yml

```yaml
# 基本配置
title: 我的博客
description: 个人技术博客
url: "https://yourusername.github.io"
baseurl: ""

# Jekyll 配置
markdown: kramdown
highlighter: rouge
permalink: /:year/:month/:day/:title/

# 允许未来日期文章
future: true
timezone: Asia/Shanghai

# 插件
plugins:
  - jekyll-feed
  - jekyll-sitemap
  - jekyll-seo-tag
```

### 第四步：创建第一篇文章

在 `_posts/` 目录下创建文件，命名格式：`YYYY-MM-DD-文章标题.md`

```markdown
---
layout: post
title: "Hello World - 我的第一篇博客"
date: 2026-03-08
categories: [随笔]
tags: [hello, 第一篇]
excerpt: "这是我的第一篇博客文章，记录开始写博客的心情。"
---

# 你好，世界！

这是我的第一篇博客文章...
```

**⚠️ 注意**：front matter（`---` 之间的内容）是必须的，否则 Jekyll 不会处理该文件。

### 第五步：推送并启用 GitHub Pages

```bash
git add .
git commit -m "feat: initial commit"
git push origin main
```

然后在 GitHub 仓库设置中：
1. 进入 **Settings** → **Pages**
2. Source 选择 **Deploy from a branch**
3. Branch 选择 **main**，文件夹选择 **/(root)**
4. 点击 **Save**

等待 1-2 分钟，访问 `https://用户名.github.io` 即可看到博客！

---

## 🔥 踩坑记录

### 坑 1：文章不显示

**问题**: 推送后首页显示"暂无文章"

**原因**: 
1. 文章日期是未来时间
2. 缺少 `future: true` 配置
3. 时区问题导致文章被认为是未来

**解决方案**:
```yaml
# _config.yml
future: true
timezone: Asia/Shanghai
```

---

### 坑 2：导航链接 404

**问题**: 点击导航栏"关于"和"归档"返回 404

**原因**: Jekyll 处理带 front matter 的 HTML 文件时，会自动去掉 `.html` 后缀
- `about.html` → 实际 URL 是 `/about/`
- `archive.html` → 实际 URL 是 `/archive/`

**解决方案**:
```yaml
# _config.yml
nav:
  - name: 关于
    url: /about/      # ✅ 正确
    # url: /about.html  # ❌ 错误
```

---

### 坑 3：Git 推送超时

**问题**: `git push` 时连接超时或认证失败

**原因**: 
1. 网络连接问题
2. HTTPS 认证方式变更

**解决方案**:
```bash
# 方案 1：使用 token 认证
git remote set-url origin https://TOKEN@github.com/用户名/仓库.git

# 方案 2：增加缓冲区大小
git config http.postBuffer 52428800

# 方案 3：使用 SSH
git remote set-url origin git@github.com:用户名/仓库.git
```

---

### 坑 4：样式不生效

**问题**: CSS 文件推送后样式没有更新

**原因**: 
1. 浏览器缓存
2. GitHub Pages 缓存
3. CSS 文件路径错误

**解决方案**:
```html
<!-- 在 CSS 链接后添加版本号 -->
<link rel="stylesheet" href="/assets/css/style.css?v=1.0.1">
```

---

### 坑 5：本地预览正常，线上 404

**问题**: `jekyll serve` 本地预览正常，推送后 404

**原因**:
1. 文件大小写不一致（Linux 区分大小写）
2. 文件未正确提交
3. GitHub Pages 还在构建中

**解决方案**:
```bash
# 检查文件是否正确提交
git ls-files | grep -i about

# 检查构建状态
curl -H "Authorization: token TOKEN" \
  https://api.github.com/repos/用户名/仓库/pages
```

---

## 🎨 SEO 优化配置

### 完整的 _config.yml SEO 配置

```yaml
# 基本信息
title: 黑猫技术博客
description: 技术笔记与问题总结
url: "https://gis-blackcaat.github.io"

# SEO 插件
plugins:
  - jekyll-seo-tag
  - jekyll-sitemap
  - jekyll-feed

# 社交媒体
author:
  name: 黑猫
  twitter: your_twitter
  
# 默认 front matter
defaults:
  - scope:
      path: ""
      type: "posts"
    values:
      layout: post
      # 默认 SEO 配置
      image: /assets/images/og-image.png
```

### 文章 SEO 优化

```markdown
---
title: "文章标题（包含关键词）"
date: 2026-03-08
categories: [分类]
tags: [标签 1, 标签 2]
excerpt: "150-160 字的摘要，包含关键词，用于搜索引擎展示"
image: /assets/images/文章封面.jpg
---
```

### 提交到搜索引擎

```bash
# Google Search Console
# 访问：https://search.google.com/search-console
# 添加网站并提交 sitemap

# sitemap 地址：https://你的域名.github.io/sitemap.xml
```

---

## 📊 添加访问统计

### 方案 1：Google Analytics（推荐）

```html
<!-- _layouts/default.html 的 </head> 前 -->
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>
```

### 方案 2：Clarity（微软免费）

```html
<!-- _layouts/default.html 的 </head> 前 -->
<script type="text/javascript">
  (function(c,l,a,r,i,t,y){
    c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
    t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
    y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
  })(window, document, "clarity", "script", "YOUR_CLARITY_ID");
</script>
```

### 方案 3：简单计数器

使用 [Busuanzi](http://busuanzi.ibruce.info/) 或 [CountAPI](https://countapi.xyz/)

```html
<!-- 页面访问量 -->
<span id="busuanzi_container_page_pv">
  本文阅读量：<span id="busuanzi_value_page_pv"></span>
</span>

<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

---

## 📈 性能优化

### 1. 启用缓存

```html
<!-- 添加资源版本号 -->
<link rel="stylesheet" href="/assets/css/style.css?v={{ site.time | date: '%s' }}">
```

### 2. 压缩资源

使用 [jekyll-minifier](https://github.com/digitalsparky/jekyll-minifier)

```yaml
# _config.yml
plugins:
  - jekyll-minifier
  
jekyll-minifier:
  compress_javascript: true
  compress_css: true
```

### 3. 图片优化

```bash
# 使用 WebP 格式
# 压缩图片
convert image.jpg -quality 80 image-optimized.jpg
```

---

## 🎯 最佳实践

### 1. 文章命名规范

```
✅ 2026-03-08-article-title.md
❌ 2026-03-08-文章标题.md  # 建议用英文
❌ article-title.md        # 缺少日期
```

### 2. 目录结构

```
✅ 推荐结构
├── _posts/          # 博客文章
├── _layouts/        # 布局模板
├── _includes/       # 可复用组件
├── assets/          # 静态资源
└── _pages/          # 独立页面

❌ 避免
├── 文章/            # 中文目录名
├── images/          # 直接在根目录
```

### 3. Git 提交规范

```bash
feat: 新功能
fix: 修复 bug
docs: 文档更新
style: 样式调整
refactor: 重构代码
chore: 构建/工具变更
```

---

## 🔗 有用资源

- [Jekyll 官方文档](https://jekyllrb.com/docs/)
- [GitHub Pages 文档](https://docs.github.com/en/pages)
- [Jekyll Themes](https://jekyllthemes.io/)
- [Google Search Console](https://search.google.com/search-console)
- [Google Analytics](https://analytics.google.com/)

---

## 💬 总结

搭建 GitHub Pages 博客整体来说比较简单，但细节坑不少。主要注意：

1. **仓库命名**必须是 `用户名.github.io`
2. **文章日期**配置 `future: true`
3. **导航链接**不要带 `.html` 后缀
4. **SEO 优化**要配置 `jekyll-seo-tag`
5. **访问统计**推荐 Google Analytics

希望这篇指南能帮你少走弯路！有问题欢迎在评论区交流～

---

**最后更新**: 2026 年 3 月 8 日  
**作者**: 黑猫 🐈‍⬛
