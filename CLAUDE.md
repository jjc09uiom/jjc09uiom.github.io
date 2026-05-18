# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 项目简介

这是一个部署在 GitHub Pages（`jjc09uiom.github.io`）的静态个人网站。博客页面（`index.html`、`archives/`）是使用 Butterfly 主题的 **Hexo 6.3.0 预编译输出** —— 仓库中没有 Hexo 源文件，只有生成好的 HTML。特殊页面（`love.html`、`audio-player.html`）是手写的独立 HTML 文件。

没有构建步骤、包管理器或测试套件。本地预览只需用浏览器打开任意 HTML 文件，或运行简易静态服务器：`python3 -m http.server 8080`。

## 架构

### 页面
- **`index.html` / `archives/`** — Hexo/Butterfly 预生成的博客页面。避免手动编辑，通常应从独立的 Hexo 源码仓库重新生成。
- **`love.html`** — 密码保护的个人页面，需输入 `我爱你么么哒特` 解锁。验证通过后展示打字机动效文字与爱心粒子画布，并通过 `sessionStorage.setItem('fromLovePage', 'true')` 跳转至 `audio-player.html`。
- **`audio-player.html`** — 音频播放器，后端依托部署在 `https://api.cj0421.site` 的 **Cloudflare R2 Worker**。页面加载时检查 `sessionStorage.getItem('fromLovePage')` 以防止直接访问。Worker 接口：`GET /list`、`POST /upload`、`POST /delete`、`GET /url`。

### JS
- **`js/utils.js`** — 导出全局 `btf` 对象，提供 DOM 工具函数（防抖、节流、滚动、wrap/unwrap、lightbox 初始化），被 `main.js` 依赖。
- **`js/main.js`** — Butterfly 主题运行时：导航栏布局自适应、移动端侧边栏、深色/浅色模式切换（通过 `saveToLocal` 持久化至 `localStorage`，有效期 2 天）、滚动吸顶导航、目录（TOC）、代码高亮工具栏、图片灯箱（Fancybox）、瀑布流图库。
- **`js/tw_cn.js`** — 繁体/简体中文切换。
- **`js/search/`** — Algolia 与本地搜索插件（通过 `GLOBAL_CONFIG` 按需加载）。

### 配置
Butterfly 页面的运行时配置以 `GLOBAL_CONFIG`（全站）和 `GLOBAL_CONFIG_SITE`（单页）的形式内联在各 HTML 文件中，没有独立的外部配置文件。

## 重要行为说明

- `love.html` 在 `<script>` 块中以明文硬编码密码，修改密码须直接编辑该处。
- `audio-player.html` 硬编码了 `WORKER_URL = 'https://api.cj0421.site'`，若 Worker 域名变更需同步修改此常量。
- 深色模式偏好与侧边栏状态通过 `saveToLocal`（localStorage + 2 天过期）存储，定义在 `index.html` 的 `<head>` 内联脚本中。
- CDN 资源（Font Awesome、Fancybox、fireworks）从 `cdn.jsdelivr.net` 加载。CDN 不可达时视觉特效会优雅降级，但页面布局仍可正常使用。
