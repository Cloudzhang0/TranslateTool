# 个人翻译工具（桌面版）

一款基于 Google 翻译风格设计的免费翻译工具，支持文本翻译、图片 OCR、文档解析、网页抓取和语音输入，提供标准/口语/正式三种翻译风格。

**在线体验**：https://cloudzhang0.github.io/TranslateTool/

## 功能特性

- **五种输入方式**：文字翻译、图片 OCR、文档解析、网页抓取、语音输入
- **智能翻译**：支持标准、口语、正式三种翻译风格
- **语言自动检测**：自动识别输入语言（中/英/日/韩/阿/泰等）
- **语音功能**：语音朗读（Web Speech API，美式英语优先）
- **翻译历史**：本地存储，支持搜索和一键复用
- **响应式设计**：桌面端双栏并排，移动端单栏堆叠
- **简洁界面**：Google 翻译风格 UI，纯白背景，扁平设计
- **部署友好**：纯前端部署到 GitHub Pages，翻译 API 通过 Cloudflare Worker 代理

## 技术栈

| 层级 | 技术 |
|------|------|
| **前端框架** | React 19 + TypeScript |
| **构建工具** | Vite |
| **样式方案** | Tailwind CSS |
| **状态管理** | Zustand |
| **图标库** | Lucide React |
| **翻译 API** | 百度翻译 API（通过 Cloudflare Worker 代理，保护密钥安全） |
| **部署** | GitHub Pages + GitHub Actions 自动构建 |

## 快速开始

### 1. 安装依赖

```bash
npm install
```

### 2. 配置 Cloudflare Worker

翻译功能依赖 Cloudflare Worker 代理百度翻译 API 请求（用于保护 API 密钥不暴露在前端代码中）。

- 部署 `cloudflare-worker.js` 到你的 Cloudflare Workers
- 在 `.env` 或 Vite 环境变量中设置 `VITE_WORKER_URL`

### 3. 启动开发服务器

```bash
npm run dev
```

前端默认运行在 `http://localhost:5173`。

### 4. 构建并部署

```bash
npm run build
```

构建产物输出到 `dist/`，通过 GitHub Actions 自动部署到 GitHub Pages。

## 项目结构

```
translate-tool/
├── index.html                    # 入口 HTML
├── package.json                  # 前端依赖配置
├── vite.config.ts                # Vite 配置
├── tsconfig.json                 # TypeScript 配置
├── .github/
│   └── workflows/
│       └── deploy.yml            # GitHub Actions 自动部署
├── public/
│   ├── favicon.svg               # 标签页图标
│   └── icons.svg                 # 功能图标
├── server/                       # Flask 后端（OCR/文档/网页抓取，翻译已迁移到 Worker）
│   └── app.py
└── src/
    ├── main.tsx                  # React 入口
    ├── App.tsx                   # 应用根组件（布局编排）
    ├── index.css                 # 全局样式（Google 色板）
    ├── types/index.ts            # TypeScript 类型定义
    ├── stores/appStore.ts        # Zustand 全局状态
    ├── services/
    │   ├── translation.ts        # 百度翻译 API（通过 Cloudflare Worker）+ 语音合成
    │   ├── ocr.ts                # 图片 OCR 服务
    │   ├── document.ts           # 文档解析服务
    │   └── website.ts            # 网页抓取服务
    ├── hooks/
    │   ├── useTranslation.ts     # 翻译逻辑 Hook
    │   ├── useSpeech.ts          # 语音朗读 Hook
    │   └── useHistory.ts         # 历史记录 Hook
    ├── utils/
    │   ├── language.ts           # 语言列表和工具函数
    │   └── debounce.ts           # 防抖工具函数
    └── components/
        ├── layout/
        │   ├── Header.tsx        # 顶部导航（含设置下拉）
        │   ├── LanguageBar.tsx   # 语言选择栏
        │   ├── InputPanel.tsx    # 输入面板
        │   ├── OutputPanel.tsx   # 输出面板
        │   ├── FeatureBar.tsx    # 底部功能栏（5 个图标按钮）
        │   └── HistoryPanel.tsx  # 历史记录侧边栏
        └── input/
            ├── TextInput.tsx     # 文本输入
            ├── ImageInput.tsx    # 图片上传
            ├── DocumentInput.tsx # 文档上传
            ├── WebsiteInput.tsx  # 网址输入
            └── VoiceInput.tsx    # 语音输入
```

## 设计规范

采用 Google 翻译风格配色方案：

- 背景色：`#FFFFFF`
- 主色：`#1A73E8`
- 文字主色：`#202124`
- 文字次要：`#5F6368`
- 边框：`#DADCE0`
- 悬停背景：`#F1F3F4`
- 输出区背景：`#F8F9FA`
- 占位符：`#80868B`
- 字体：Roboto + 微软雅黑

## 环境要求

- Node.js 18+
- 现代浏览器（Chrome、Edge 推荐）

---

## 开发历程

### 架构演进

1. **初版**：Express.js 后端 + `translators` 库调用第三方翻译引擎
2. **Flask 迁移**：后端从 Express.js 迁移到 Python Flask，使用 `translators` 库（引擎回退链：alibaba → bing → google → baidu）
3. **纯前端化**：移除 Flask 后端，翻译 API 改为通过 Cloudflare Worker 代理百度翻译 API，前端可独立部署到 GitHub Pages

### UI 演进

1. **初版**：渐变背景、毛玻璃导航栏、圆角卡片
2. **Google 风格重构**：全面改为纯白背景、扁平设计、Google 翻译配色
3. **布局优化**：调整输入/输出面板高度，优化桌面端双栏比例

### 解决的关键问题

#### 1. 翻译引擎超时导致 500 错误
- **原因**：Bing 引擎极慢（>15s），Flask 单线程被阻塞
- **解决**：改用 alibaba 引擎、8 秒超时、4 引擎回退链 + 2 次重试
- **经验**：调用第三方库必须设计超时和降级策略

#### 2. 百度翻译 Invalid Sign 错误
- **原因**：MD5 签名计算时未正确处理中文 UTF-8 编码
- **解决**：在 MD5 实现中使用 `unescape(encodeURIComponent(s))` 确保 UTF-8 字节编码
- **经验**：涉及多语言文本的 API 签名必须统一使用 UTF-8 编码

#### 3. 语音朗读英国腔调
- **原因**：浏览器默认英语语音是英式的（Microsoft David）
- **解决**：加载语音列表后优先选择 en-US 美式语音（Zira/Mark/Steffan）
- **经验**：`speechSynthesis.getVoices()` 需等待异步加载完成

#### 4. Cloudflare Worker 代理
- **目的**：百度翻译 API 不支持浏览器 CORS 跨域请求，且密钥不应暴露在前端代码中
- **方案**：Cloudflare Worker 接收前端请求 → 服务器端签名 + 调用百度 API → 返回结果
- **经验**：边缘计算 Worker 是保护 API 密钥的轻量级方案

### 经验总结

- **先预览再修改**：大规模 UI 重构前先做独立预览页，确认效果后再改源码
- **选择器模式**：Zustand 的选择器按需订阅，避免不必要的渲染
- **配色变量化**：CSS 变量集中管理色板，便于统一修改
- **边缘代理**：使用 Cloudflare Worker 保护 API 密钥，同时解决 CORS 问题
