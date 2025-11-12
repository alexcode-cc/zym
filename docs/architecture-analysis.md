# ZyFun (ZyPlayer) 项目架构深度分析

> 专业视角的 Electron 桌面应用架构解析
>
> 文档版本: 1.0
> 项目版本: 3.4.1-beta20250602
> 分析日期: 2025-11-12

---

## 📋 目录

- [1. 项目概览](#1-项目概览)
- [2. 技术栈分析](#2-技术栈分析)
- [3. 项目架构设计](#3-项目架构设计)
- [4. 目录结构详解](#4-目录结构详解)
- [5. 核心模块分析](#5-核心模块分析)
- [6. 数据流与通信机制](#6-数据流与通信机制)
- [7. 构建与打包](#7-构建与打包)
- [8. 开发指南](#8-开发指南)
- [9. 架构优势与挑战](#9-架构优势与挑战)

---

## 1. 项目概览

### 1.1 项目定位

**ZyFun (原名 ZyPlayer)** 是一款基于 Electron 的跨平台媒体播放器桌面应用，专注于提供流畅、高效的娱乐体验。

**核心特性:**
- 🎬 影视资源聚合播放
- 📺 IPTV 直播支持
- 🔍 多源搜索与解析
- 💾 网盘资源集成
- 🎨 深色模式支持
- 🖥️ 跨平台 (Windows/MacOS/Linux)

### 1.2 应用类型

这是一个典型的 **Electron 三层架构应用**：
- **Main Process**: Node.js 主进程（业务逻辑、系统调用）
- **Renderer Process**: 浏览器渲染进程（UI 界面）
- **Preload Script**: 预加载脚本（安全桥接层）

---

## 2. 技术栈分析

### 2.1 核心框架

| 技术 | 版本 | 用途 |
|------|------|------|
| **Electron** | ^36.4.0 | 桌面应用框架 |
| **Electron-Vite** | ^3.1.0 | 构建工具 |
| **Vue 3** | ^3.5.16 | 前端框架 |
| **TypeScript** | ^5.8.3 | 类型系统 |
| **Vite** | ^6.3.5 | 开发服务器 |

### 2.2 前端技术栈

#### UI 组件库
```typescript
// TDesign - 腾讯开源 Vue 组件库
import { TDesign } from 'tdesign-vue-next';  // v1.13.2
import { TDesignIcons } from 'tdesign-icons-vue-next';  // v0.3.6
import { TDesignChat } from '@tdesign-vue-next/chat';  // v0.3.0
```

#### 状态管理
```typescript
// Pinia - Vue 3 官方推荐状态管理
import { createPinia } from 'pinia';  // v3.0.3
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate';  // v4.3.0
```

#### 路由系统
```typescript
// Vue Router - 采用 Hash 模式
import { createRouter, createWebHashHistory } from 'vue-router';  // v4.5.1
```

### 2.3 后端技术栈

#### HTTP 服务框架
```typescript
// Fastify - 高性能 Node.js Web 框架
import fastify from 'fastify';  // v5.3.3
import fastifyCors from '@fastify/cors';  // v11.0.1
import fastifyMultipart from '@fastify/multipart';  // v9.0.3
```

**监听端口:** `9978`
**服务地址:** `http://0.0.0.0:9978`

#### 数据库方案
```typescript
// PGlite - 轻量级 PostgreSQL 实现
import { PGlite } from '@electric-sql/pglite';  // v0.3.3
import pgliteServer from 'pglite-server';  // v0.1.4

// Drizzle ORM - TypeScript ORM
import { drizzle } from 'drizzle-orm';  // v0.44.2
import drizzleKit from 'drizzle-kit';  // v0.31.1

// Node-JSON-DB - 临时缓存数据库
import { JsonDB } from 'node-json-db';  // v2.3.1
```

**数据存储架构:**
- **PGlite**: 主数据库（结构化数据）
- **JsonDB**: 临时缓存（session 数据）
- **WebDAV**: 远程备份同步

### 2.4 播放器引擎

项目集成了**多种播放器内核**以支持不同的媒体格式：

| 播放器 | 版本 | 特点 |
|--------|------|------|
| **XGPlayer** | ^3.0.22 | 西瓜播放器，H.265 兼容性好 |
| **ArtPlayer** | ^5.2.3 | 现代化播放器，轻量级 |
| **DPlayer** | ^1.27.1 | 弹幕播放器 |
| **NPlayer** | ^1.0.15 | 原生播放器 |
| **OPlayer** | ^1.2.38 | 插件化播放器 |

**视频解码库:**
```typescript
import Hls from 'hls.js';           // v1.6.5 - HLS 直播
import dashjs from 'dashjs';        // v5.0.3 - DASH 流媒体
import flvjs from 'flv.js';         // v1.6.2 - FLV 格式
import mpegts from 'mpegts.js';     // v1.8.0 - MPEG-TS
import shaka from 'shaka-player';   // v4.15.0 - 自适应流
```

### 2.5 爬虫与解析

```typescript
// 网页解析
import cheerio from 'cheerio';              // v1.0.0
import { XMLParser } from 'fast-xml-parser';  // v5.2.3
import xpath from 'xpath';                   // v0.0.34

// 浏览器自动化
import puppeteer from 'puppeteer-core';     // v24.10.0
import pie from 'puppeteer-in-electron';    // v3.0.5
```

### 2.6 工具库

```typescript
// HTTP 请求
import axios from 'axios';                   // v1.9.0

// 加密解密
import CryptoJS from 'crypto-js';           // v4.2.0
import NodeRSA from 'node-rsa';             // v1.1.1
import sm from 'sm-crypto';                  // v0.3.13

// 实用工具
import lodash from 'lodash-es';             // v4.17.21
import moment from 'moment';                 // v2.30.1
import mitt from 'mitt';                     // v3.0.1 - 事件总线
import { VueUse } from '@vueuse/core';       // v13.3.0
```

---

## 3. 项目架构设计

### 3.1 Electron 三层架构

```
┌─────────────────────────────────────────────────────────────┐
│                      Application                             │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────┐         ┌─────────────────┐           │
│  │  Main Process   │◄────────┤  Preload Script │           │
│  │  (Node.js)      │   IPC   │  (Bridge)       │           │
│  └────────┬────────┘         └────────▲────────┘           │
│           │                            │                     │
│           │ IPC Communication          │                     │
│           │                            │                     │
│  ┌────────▼────────────────────────────┴────────┐           │
│  │         Renderer Process (Chromium)          │           │
│  │         Vue 3 + TDesign UI                   │           │
│  └──────────────────────────────────────────────┘           │
│                                                               │
│  ┌──────────────────────────────────────────────┐           │
│  │       Internal Fastify Server :9978          │           │
│  │       API Routes + Business Logic            │           │
│  └──────────────────────────────────────────────┘           │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 内部 HTTP 服务架构

**设计亮点:** 在 Electron 主进程中嵌入 Fastify HTTP 服务器

**优势:**
1. **进程隔离**: 业务逻辑与 UI 解耦
2. **性能优化**: 避免主进程阻塞
3. **安全性**: Renderer 进程通过 HTTP API 调用，限制权限
4. **可测试**: API 可独立测试

```typescript
// src/main/core/server/index.ts
const server = fastify({
  logger: { level: 'info' },
  bodyLimit: 3 * 1024 * 1024,  // 3MB
  maxParamLength: 10240
});

await server.listen({
  port: 9978,
  host: '0.0.0.0'
});
```

### 3.3 模块化架构

```
src/
├── main/                 # 主进程
│   ├── core/            # 核心功能
│   │   ├── db/          # 数据库层
│   │   ├── server/      # HTTP 服务层
│   │   ├── ipc/         # IPC 通信层
│   │   ├── winManger/   # 窗口管理
│   │   ├── logger/      # 日志系统
│   │   ├── shortcut/    # 快捷键
│   │   ├── tray/        # 系统托盘
│   │   ├── menu/        # 菜单
│   │   ├── update/      # 自动更新
│   │   └── protocol/    # 自定义协议
│   └── utils/           # 工具函数
│
├── renderer/            # 渲染进程
│   └── src/
│       ├── pages/       # 页面组件
│       ├── components/  # 公共组件
│       ├── layouts/     # 布局组件
│       ├── api/         # API 调用
│       ├── store/       # Pinia 状态
│       ├── router/      # 路由配置
│       ├── utils/       # 工具函数
│       ├── types/       # TypeScript 类型
│       ├── locales/     # 国际化
│       ├── config/      # 配置文件
│       ├── style/       # 全局样式
│       └── assets/      # 静态资源
│
└── preload/             # 预加载脚本
    ├── index.ts         # 入口
    └── utils/           # 工具函数
```

---

## 4. 目录结构详解

### 4.1 主进程核心模块 (src/main/core/)

#### 4.1.1 数据库模块 (db/)

```typescript
src/main/core/db/
├── index.ts              # 数据库初始化 & 版本迁移
├── common/
│   ├── client.ts         # PGlite 客户端实例
│   ├── schema.ts         # Drizzle ORM Schema
│   ├── server.ts         # PGlite Server
│   └── webdev.ts         # WebDAV 同步
├── migration/            # 数据库迁移脚本
│   ├── update3_3_1to3_3_2.ts
│   ├── update3_3_3to3_3_4.ts
│   └── ...
└── service/              # 数据服务层
    ├── analyze.ts        # 解析源服务
    ├── channel.ts        # 频道服务
    ├── drive.ts          # 网盘服务
    ├── history.ts        # 历史记录服务
    ├── iptv.ts           # IPTV 服务
    ├── setting.ts        # 设置服务
    └── site.ts           # 站点服务
```

**版本迁移机制:**
```typescript
const updates = [
  { version: '3.3.2', update: migration.update3_3_1to3_3_2 },
  { version: '3.3.4', update: migration.update3_3_3to3_3_4 },
  // ...
];

// 自动检测并执行增量迁移
if (compare(dbVersion, appVersion, '<')) {
  for (const { version, update } of updates) {
    if (compare(dbVersion, version, '<')) {
      await update();
    }
  }
}
```

#### 4.1.2 HTTP 服务模块 (server/)

```typescript
src/main/core/server/
├── index.ts              # Fastify 服务器初始化
└── routes/
    └── v1/               # API v1 版本
        ├── site/         # 站点资源 API
        │   └── cms/
        │       └── adapter/  # CMS 适配器
        │           ├── drpy/    # Drpy JS 解析
        │           ├── py/      # Python 解析
        │           ├── xbpq/    # XBPQ 解析
        │           └── xyq/     # XYQ 解析
        ├── live/         # 直播 API
        ├── drive/        # 网盘 API
        ├── parse/        # 解析 API
        ├── player/       # 播放器 API
        ├── history/      # 历史记录 API
        ├── star/         # 收藏 API
        ├── db/           # 数据库操作 API
        ├── proxy/        # 代理 API
        ├── system/       # 系统 API
        ├── plugin/       # 插件 API
        ├── lab/          # 实验室功能 API
        ├── setting/      # 设置 API
        └── file/         # 文件操作 API
```

**API 路由示例:**
```
GET  /api/v1/site/list          # 获取站点列表
POST /api/v1/site/search        # 搜索影视
GET  /api/v1/site/detail/:id    # 获取详情
GET  /api/v1/live/channels      # 获取直播频道
POST /api/v1/parse/video        # 解析视频
GET  /api/v1/history/list       # 历史记录
```

**CMS 适配器架构:**

项目支持多种视频 CMS 协议：

| 适配器 | 类型 | 说明 |
|--------|------|------|
| **drpy** | JavaScript | DrPy 动态解析框架 |
| **py** | Python | Hipy/T4 Python 解析 |
| **xbpq** | XBPQ | XBPQ 规则解析 |
| **xyq** | XYQ | 香雅情规则解析 |
| **T0** | XML | CMS V10 XML 格式 |
| **T1** | JSON | CMS V1 JSON 格式 |

#### 4.1.3 IPC 通信模块 (ipc/)

```typescript
// 主进程监听渲染进程消息
ipcMain.handle('get-app-version', () => {
  return app.getVersion();
});

// 渲染进程调用
const version = await ipcRenderer.invoke('get-app-version');
```

#### 4.1.4 窗口管理 (winManger/)

```typescript
// 多窗口管理
export const createMain = () => { /* 创建主窗口 */ };
export const createPlay = () => { /* 创建播放窗口 */ };

// 窗口状态记忆
windowPosition: {
  position_main: { width: 1000, height: 640 },
  position_play: { width: 875, height: 550 }
}
```

### 4.2 渲染进程结构 (src/renderer/)

#### 4.2.1 页面模块 (pages/)

```typescript
src/renderer/src/pages/
├── film/                 # 影视模块
│   ├── index.vue         # 影视首页
│   └── components/       # 影视组件
│       ├── SearchBar.vue
│       ├── VideoCard.vue
│       └── VideoDetail.vue
├── iptv/                 # 直播模块
│   ├── index.vue
│   └── components/
├── play/                 # 播放页面
│   ├── index.vue
│   └── componets/
│       ├── Player.vue
│       └── Danmaku.vue
├── analyze/              # 解析模块
├── drive/                # 网盘模块
├── chase/                # 追剧模块
├── lab/                  # 实验室
│   ├── index.vue
│   └── components/
│       ├── AiChat.vue     # AI 聊天
│       ├── Terminal.vue   # 终端
│       └── Sniffer.vue    # 嗅探器
├── setting/              # 设置模块
└── test/                 # 测试页面
```

#### 4.2.2 API 封装 (api/)

```typescript
src/renderer/src/api/
├── index.ts              # API 入口
├── site.ts               # 站点 API
├── iptv.ts               # IPTV API
├── drive.ts              # 网盘 API
├── analyze.ts            # 解析 API
├── history.ts            # 历史 API
├── star.ts               # 收藏 API
├── setting.ts            # 设置 API
├── lab.ts                # 实验室 API
├── plugin.ts             # 插件 API
└── proxy.ts              # 代理 API
```

**API 调用示例:**
```typescript
// src/renderer/src/api/site.ts
export const fetchSiteList = async () => {
  return axios.get('http://localhost:9978/api/v1/site/list');
};

export const searchVideo = async (keyword: string) => {
  return axios.post('http://localhost:9978/api/v1/site/search', {
    keyword
  });
};
```

#### 4.2.3 状态管理 (store/)

```typescript
src/renderer/src/store/
├── index.ts              # Pinia 实例
├── plugins/
│   └── persist.ts        # 持久化插件
└── modules/
    ├── play.ts           # 播放状态
    └── setting.ts        # 设置状态
```

**状态管理示例:**
```typescript
// store/modules/play.ts
export const usePlayStore = defineStore('play', {
  state: () => ({
    currentVideo: null,
    playHistory: [],
    playerType: 'xgplayer'
  }),

  actions: {
    setCurrentVideo(video) {
      this.currentVideo = video;
    }
  },

  // 持久化配置
  persist: {
    enabled: true,
    strategies: [
      {
        key: 'play-store',
        storage: localStorage
      }
    ]
  }
});
```

#### 4.2.4 路由配置 (router/)

```typescript
src/renderer/src/router/
├── index.ts              # 路由入口
└── modules/
    ├── film/
    │   └── homepage.ts   # 影视路由
    ├── iptv/
    │   └── homepage.ts   # 直播路由
    ├── analyze/
    │   └── homepage.ts   # 解析路由
    └── setting/
        └── homepage.ts   # 设置路由
```

**路由自动加载机制:**
```typescript
// 自动导入所有 homepage.ts 路由模块
const homepageModules = import.meta.glob('./modules/**/homepage.ts', {
  eager: true
});

export const homepageRouterList = mapModuleRouterList(homepageModules);
```

**路由示例:**
```typescript
// router/modules/film/homepage.ts
export default [
  {
    path: '/film',
    component: () => import('@/layouts/MainLayout.vue'),
    children: [
      {
        path: 'index',
        name: 'FilmIndex',
        component: () => import('@/pages/film/index.vue'),
        meta: { title: '影视' }
      }
    ]
  }
];
```

### 4.3 配置文件

#### 4.3.1 Electron Vite 配置

```typescript
// electron.vite.config.ts
export default defineConfig({
  main: {
    // 主进程配置
    resolve: {
      alias: { '@main': resolve('src/main') }
    },
    plugins: [externalizeDepsPlugin(), swcPlugin()],
    build: {
      rollupOptions: {
        input: {
          index: resolve(__dirname, 'src/main/index.ts'),
          // Worker 线程入口
          site_drpy_worker: resolve(__dirname, 'src/main/core/server/routes/v1/site/cms/adapter/drpy/worker.ts')
        },
        output: {
          manualChunks: {
            fastify: ['fastify', '@fastify/cors'],
            db: ['drizzle-kit', 'drizzle-orm'],
            crypto: ['crypto-js', 'node-rsa']
          }
        }
      }
    }
  },

  preload: {
    // 预加载脚本配置
    plugins: [externalizeDepsPlugin()]
  },

  renderer: {
    // 渲染进程配置
    resolve: {
      alias: {
        '@renderer': resolve('src/renderer'),
        '@': resolve('src/renderer/src')
      }
    },
    plugins: [
      vue(),
      vueJsx(),
      svgLoader(),
      AutoImport({
        resolvers: [TDesignResolver({ library: 'vue-next' })]
      }),
      Components({
        resolvers: [TDesignResolver({ library: 'vue-next' })]
      })
    ],
    build: {
      rollupOptions: {
        output: {
          manualChunks: {
            'monaco-editor': ['monaco-editor'],
            xgplayer: ['xgplayer', 'xgplayer-flv', 'xgplayer-hls'],
            artplayer: ['artplayer', 'artplayer-plugin-danmuku'],
            tdesign: ['tdesign-vue-next', 'tdesign-icons-vue-next'],
            vue: ['vue', 'vue-router', 'pinia']
          }
        }
      }
    }
  }
});
```

#### 4.3.2 TypeScript 配置

```typescript
// tsconfig.json
{
  "extends": "@electron-toolkit/tsconfig/tsconfig.json",
  "references": [
    { "path": "./tsconfig.node.json" },  // 主进程
    { "path": "./tsconfig.web.json" }    // 渲染进程
  ]
}

// tsconfig.node.json - 主进程配置
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "Bundler"
  }
}

// tsconfig.web.json - 渲染进程配置
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "preserve"
  }
}
```

#### 4.3.3 Electron Builder 配置

```yaml
# electron-builder.yml
appId: com.hiram.zyplayer
productName: zyfun
directories:
  output: release/${version}

files:
  - '!**/.vscode/*'
  - '!src/*'
  - '!electron.vite.config.{js,ts,mjs,cjs}'

asarUnpack:
  - resources/**

win:
  target:
    - target: nsis
      arch: [x64, ia32, arm64]

mac:
  target:
    - target: dmg
      arch: [x64, arm64, universal]
  hardenedRuntime: true
  gatekeeperAssess: false
  entitlements: build/entitlements.mac.plist

linux:
  target:
    - AppImage
    - deb
    - rpm
  category: AudioVideo
```

---

## 5. 核心模块分析

### 5.1 播放器系统

#### 5.1.1 多播放器架构

```typescript
// 播放器配置
playerMode: {
  type: 'xgplayer' | 'artplayer' | 'dplayer' | 'nplayer' | 'oplayer' | 'custom',
  external: ''  // custom 模式下的外部播放器路径
}
```

#### 5.1.2 播放器特性对比

| 播放器 | H.264 | H.265 | 弹幕 | 自定义UI | 体积 |
|--------|-------|-------|------|----------|------|
| XGPlayer | ✅ | ✅ | ✅ | ✅ | 中 |
| ArtPlayer | ✅ | ⚠️ | ✅ | ✅ | 小 |
| DPlayer | ✅ | ❌ | ✅ | ❌ | 小 |
| NPlayer | ✅ | ⚠️ | ✅ | ✅ | 中 |
| OPlayer | ✅ | ✅ | ✅ | ✅ | 大 |

**说明:**
- ✅ 完整支持
- ⚠️ 部分支持（依赖浏览器）
- ❌ 不支持

#### 5.1.3 视频解析流程

```
用户播放请求
    ↓
检测视频源类型
    ↓
┌───────────────┬───────────────┬───────────────┐
│   直连播放    │   需要解析    │   第三方解析  │
└───────┬───────┴───────┬───────┴───────┬───────┘
        ↓               ↓               ↓
   直接播放      CMS解析适配器    解析源API
        ↓               ↓               ↓
        └───────────────┴───────────────┘
                        ↓
                  获取真实播放地址
                        ↓
                  选择播放器引擎
                        ↓
                    开始播放
```

### 5.2 CMS 解析系统

#### 5.2.1 DrPy 动态解析

```typescript
// src/main/core/server/routes/v1/site/cms/adapter/drpy/
├── adapter.ts            # DrPy 适配器
├── worker.ts             # Worker 线程执行
└── libs/                 # DrPy 库文件
```

**DrPy 特点:**
- JavaScript 动态规则
- 支持自定义函数
- 支持网络请求
- Worker 隔离执行

**执行流程:**
```typescript
// 1. 主线程接收解析请求
const result = await drpyAdapter.parse(rule, url);

// 2. Worker 线程执行规则
// worker.ts
import { workerData, parentPort } from 'worker_threads';
const result = eval(workerData.code);
parentPort.postMessage(result);
```

#### 5.2.2 XPath/CSS 选择器解析

```typescript
// XBPQ/XYQ 解析器
import cheerio from 'cheerio';
import xpath from 'xpath';

// CSS 选择器
const $ = cheerio.load(html);
const title = $('.video-title').text();

// XPath 选择器
const doc = new DOMParser().parseFromString(html);
const title = xpath.select('//div[@class="title"]/text()', doc);
```

### 5.3 IPTV 直播系统

#### 5.3.1 M3U 解析

```typescript
// M3U 格式解析
#EXTM3U
#EXTINF:-1 tvg-id="cctv1" tvg-name="CCTV1" tvg-logo="logo.png" group-title="央视",CCTV1
http://example.com/cctv1.m3u8
```

**解析配置:**
```typescript
iptv: {
  type: 'remote' | 'local' | 'json',
  url: 'https://example.com/iptv.m3u8',
  epg: 'https://epg.112114.eu.org/?ch={name}&date={date}',
  logo: 'https://epg.112114.eu.org/logo/{name}.png'
}
```

#### 5.3.2 EPG 电子节目单

```typescript
// EPG 数据结构
interface EPGData {
  channel: string;
  programme: Array<{
    start: string;      // 20251112120000
    stop: string;       // 20251112130000
    title: string;
    desc: string;
  }>;
}
```

### 5.4 网盘集成

#### 5.4.1 WebDAV 协议

```typescript
import { createClient } from 'webdav';

const client = createClient(url, {
  username: user,
  password: password
});

// 备份数据
await client.putFileContents('/backup/data.json', JSON.stringify(data));

// 恢复数据
const data = await client.getFileContents('/backup/data.json');
```

#### 5.4.2 Alist 集成

```typescript
// Alist API 调用
drive: {
  type: 'alist',
  server: 'http://alist.example.com',
  showAll: false,  // 只显示视频文件
  startPage: '/电影'
}
```

### 5.5 嗅探器系统

#### 5.5.1 Puppeteer 嗅探

```typescript
// src/main/utils/sniffer/
import puppeteer from 'puppeteer-core';
import pie from 'puppeteer-in-electron';

// 在 Electron 中使用 Puppeteer
await pie.connect(app, puppeteer);

const browser = await pie.launch({
  show: false
});

// 拦截网络请求
page.on('response', async (response) => {
  const url = response.url();
  if (url.includes('.m3u8') || url.includes('.mp4')) {
    // 捕获视频地址
    sniffedUrls.push(url);
  }
});
```

#### 5.5.2 嗅探模式

```typescript
snifferMode: {
  type: 'pie' | 'iframe' | 'custom',
  url: ''  // custom 模式下的自定义嗅探页面
}
```

---

## 6. 数据流与通信机制

### 6.1 进程通信架构

```
┌─────────────────────────────────────────────────────────┐
│                    Renderer Process                      │
│  ┌──────────────────────────────────────────────────┐  │
│  │               Vue Application                     │  │
│  │  ┌────────────┐     ┌────────────┐              │  │
│  │  │   Pinia    │────▶│ API Caller │              │  │
│  │  │   Store    │◀────│  (Axios)   │              │  │
│  │  └────────────┘     └──────┬─────┘              │  │
│  └────────────────────────────│──────────────────────┘  │
│                               │                         │
│                               │ HTTP Request            │
│                               │ (localhost:9978)        │
└───────────────────────────────┼─────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────┐
│                     Main Process                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │            Fastify HTTP Server                    │  │
│  │  ┌────────────┐     ┌────────────┐              │  │
│  │  │   Routes   │────▶│  Services  │              │  │
│  │  │   Handler  │◀────│   Layer    │              │  │
│  │  └────────────┘     └──────┬─────┘              │  │
│  └────────────────────────────│──────────────────────┘  │
│                               │                         │
│                               ▼                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │              Database Layer                       │  │
│  │  ┌──────────┐   ┌──────────┐   ┌──────────┐    │  │
│  │  │  PGlite  │   │ JsonDB   │   │  WebDAV  │    │  │
│  │  │(Drizzle) │   │ (Cache)  │   │ (Backup) │    │  │
│  │  └──────────┘   └──────────┘   └──────────┘    │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 6.2 API 调用流程

```typescript
// 1. Renderer 发起请求
// src/renderer/src/pages/film/index.vue
import { fetchSiteList } from '@/api/site';

const loadSites = async () => {
  const response = await fetchSiteList();
  sites.value = response.data;
};

// 2. API 层封装
// src/renderer/src/api/site.ts
export const fetchSiteList = async () => {
  return axios.get('http://localhost:9978/api/v1/site/list');
};

// 3. Main 进程路由处理
// src/main/core/server/routes/v1/site/index.ts
server.get('/api/v1/site/list', async (request, reply) => {
  const sites = await service.site.getAll();
  return { code: 0, data: sites };
});

// 4. Service 层业务逻辑
// src/main/core/db/service/site.ts
export const getAll = async () => {
  return db.select().from(schema.site);
};
```

### 6.3 IPC 通信

```typescript
// 主进程注册
// src/main/core/ipc/index.ts
ipcMain.handle('window:minimize', () => {
  BrowserWindow.getFocusedWindow()?.minimize();
});

// 渲染进程调用
// src/renderer/src/components/TitleBar.vue
const minimize = () => {
  window.electron.ipcRenderer.invoke('window:minimize');
};
```

### 6.4 事件总线

```typescript
// src/renderer/src/utils/mitt.ts
import mitt from 'mitt';

export const emitter = mitt();

// 发布事件
emitter.emit('video:play', { id: '123' });

// 订阅事件
emitter.on('video:play', (data) => {
  console.log('Playing video:', data.id);
});
```

---

## 7. 构建与打包

### 7.1 开发模式

```bash
# 安装依赖
yarn install

# 启动开发服务
yarn dev

# 类型检查
yarn typecheck

# 代码检查
yarn lint
```

**开发服务启动流程:**
```
yarn dev
  ↓
electron-vite dev -w
  ↓
┌─────────────────────────────────────┐
│ 1. 启动 Vite 开发服务器 (Renderer)   │
│    http://localhost:5173            │
├─────────────────────────────────────┤
│ 2. 编译主进程代码 (esbuild)          │
│    out/main/index.js                │
├─────────────────────────────────────┤
│ 3. 编译预加载脚本                    │
│    out/preload/index.js             │
├─────────────────────────────────────┤
│ 4. 启动 Electron                    │
│    electron .                       │
└─────────────────────────────────────┘
```

### 7.2 生产构建

```bash
# 构建所有平台
yarn build

# 构建 Windows
yarn build:win

# 构建 MacOS
yarn build:mac

# 构建 Linux
yarn build:linux
```

**构建流程:**
```
yarn build
  ↓
electron-vite build
  ↓
┌─────────────────────────────────────┐
│ 1. 编译主进程 (Rollup)               │
│    - out/main/index.js              │
│    - out/main/site_drpy_worker.js   │
├─────────────────────────────────────┤
│ 2. 编译预加载脚本                    │
│    - out/preload/index.js           │
├─────────────────────────────────────┤
│ 3. 编译渲染进程 (Vite)               │
│    - out/renderer/index.html        │
│    - out/renderer/assets/           │
├─────────────────────────────────────┤
│ 4. Electron Builder 打包             │
│    - release/${version}/            │
│      ├── zyfun-3.4.1-win-x64.exe   │
│      ├── zyfun-3.4.1-mac-arm64.dmg │
│      └── zyfun-3.4.1-linux-x64.AppImage │
└─────────────────────────────────────┘
```

### 7.3 代码分割策略

#### 7.3.1 主进程分包

```typescript
manualChunks: {
  fastify: ['fastify', 'fastify-plugin', '@fastify/cors', '@fastify/multipart'],
  db: ['drizzle-kit', 'drizzle-orm'],
  crypto: ['crypto-js', 'he', 'pako', 'wxmp-rsa', 'node-rsa']
}
```

#### 7.3.2 渲染进程分包

```typescript
manualChunks: {
  'monaco-editor': ['monaco-editor'],
  xgplayer: ['xgplayer', 'xgplayer-flv', 'xgplayer-hls', 'xgplayer-mp4'],
  artplayer: ['artplayer', 'artplayer-plugin-danmuku'],
  dplayer: ['dplayer'],
  nplayer: ['nplayer', '@nplayer/danmaku'],
  oplayer: ['@oplayer/core', '@oplayer/plugins', '@oplayer/ui'],
  'video-decoder': ['dashjs', 'flv.js', 'hls.js', 'mpegts.js'],
  tdesign: ['tdesign-vue-next', 'tdesign-icons-vue-next'],
  vue: ['vue', 'vue-router', 'pinia', 'vue-i18n']
}
```

**优势:**
- 减少主包体积
- 按需加载
- 利用浏览器缓存
- 提升首屏速度

### 7.4 资源优化

```typescript
build: {
  assetsInlineLimit: 4096,  // 小于 4KB 的资源转为 base64
  chunkSizeWarningLimit: 2000,  // 超过 2MB 的包警告
  minify: false,  // 关闭压缩（开发模式）
  sourcemap: false  // 关闭 sourcemap
}
```

---

## 8. 开发指南

### 8.1 开发环境要求

```json
{
  "engines": {
    "node": "20 || >= 22"
  }
}
```

**必备工具:**
- Node.js 20+ / 22+
- Yarn 包管理器
- Git 版本控制

### 8.2 快速开始

```bash
# 1. 克隆项目
git clone https://github.com/Hiram-Wong/ZyPlayer.git
cd ZyPlayer

# 2. 安装依赖
yarn install

# 3. 启动开发服务
yarn dev

# 4. 构建生产版本
yarn build:win  # Windows
yarn build:mac  # MacOS
yarn build:linux  # Linux
```

### 8.3 项目规范

#### 8.3.1 代码规范

```json
{
  "scripts": {
    "lint": "eslint --ext .vue,.js,.jsx,.ts,.tsx ./ --max-warnings 0",
    "lint:fix": "eslint --fix",
    "stylelint": "stylelint src/**/*.{html,vue,sass,less}",
    "format": "prettier --write ."
  }
}
```

**配置文件:**
- `.eslintrc` - ESLint 配置
- `.prettierrc.yaml` - Prettier 配置
- `stylelint.config.js` - StyleLint 配置

#### 8.3.2 Git 提交规范

```bash
# Husky + Lint-staged
.husky/
├── pre-commit  # 提交前检查
└── commit-msg  # 提交信息检查

# lint-staged 配置
"lint-staged": {
  "*.{js,jsx,vue,ts,tsx}": [
    "prettier --write",
    "npm run lint:fix"
  ],
  "*.{html,vue,css,sass,less}": [
    "npm run stylelint:fix"
  ]
}
```

### 8.4 添加新功能

#### 8.4.1 添加新页面

```typescript
// 1. 创建页面组件
// src/renderer/src/pages/example/index.vue
<template>
  <div class="example-page">
    <h1>Example Page</h1>
  </div>
</template>

<script setup lang="ts">
// 页面逻辑
</script>

// 2. 添加路由
// src/renderer/src/router/modules/example/homepage.ts
export default [
  {
    path: '/example',
    component: () => import('@/layouts/MainLayout.vue'),
    children: [
      {
        path: 'index',
        name: 'ExampleIndex',
        component: () => import('@/pages/example/index.vue'),
        meta: { title: '示例' }
      }
    ]
  }
];
```

#### 8.4.2 添加新 API

```typescript
// 1. 定义路由处理器
// src/main/core/server/routes/v1/example/index.ts
import { FastifyInstance } from 'fastify';

export default async (fastify: FastifyInstance) => {
  fastify.get('/api/v1/example/list', async (request, reply) => {
    return { code: 0, data: [] };
  });
};

// 2. 添加到路由注册
// src/main/core/server/routes/v1/index.ts
import example from './example';

export default {
  example,
  // ...
};

// 3. 渲染进程 API 封装
// src/renderer/src/api/example.ts
export const fetchExampleList = async () => {
  return axios.get('http://localhost:9978/api/v1/example/list');
};
```

#### 8.4.3 添加数据库表

```typescript
// 1. 定义 Schema
// src/main/core/db/common/schema.ts
export const exampleTable = pgTable('tbl_example', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  createdAt: timestamp('created_at').defaultNow()
});

// 2. 添加迁移脚本
// src/main/core/db/migration/update3_4_0to3_4_1.ts
export const update3_4_0to3_4_1 = async () => {
  await db.execute(`
    CREATE TABLE IF NOT EXISTS tbl_example (
      id SERIAL PRIMARY KEY,
      name TEXT NOT NULL,
      created_at TIMESTAMP DEFAULT NOW()
    )
  `);
};

// 3. 添加 Service
// src/main/core/db/service/example.ts
export const getAll = async () => {
  return db.select().from(schema.exampleTable);
};

export const create = async (data: any) => {
  return db.insert(schema.exampleTable).values(data);
};
```

### 8.5 调试技巧

#### 8.5.1 主进程调试

```typescript
// src/main/core/logger/index.ts
import logger from '@main/core/logger';

logger.info('Info message');
logger.error('Error message');
logger.debug('Debug message');
```

**日志位置:**
- Windows: `%USERPROFILE%\AppData\Roaming\zyfun\log\`
- MacOS: `~/Library/Logs/zyfun/log/`
- Linux: `~/.config/zyfun/log/`

#### 8.5.2 渲染进程调试

```typescript
// 开启开发者工具
if (import.meta.env.DEV) {
  mainWindow.webContents.openDevTools();
}
```

#### 8.5.3 HTTP 服务调试

```bash
# Fastify 日志会输出到控制台和日志文件
# 日志文件: APP_LOG_PATH/fastify.log

# 使用 curl 测试 API
curl http://localhost:9978/api/v1/site/list
```

---

## 9. 架构优势与挑战

### 9.1 架构优势

#### 9.1.1 模块化设计

**优点:**
- 清晰的分层架构
- 高内聚低耦合
- 易于维护和扩展

**体现:**
```
核心模块分离
├── 数据库层 (db/)
├── 服务层 (server/)
├── IPC 层 (ipc/)
└── 工具层 (utils/)
```

#### 9.1.2 内嵌 HTTP 服务

**优点:**
- 业务逻辑与 UI 解耦
- 便于单元测试
- 性能优化（避免 IPC 性能损耗）
- 安全性提升（限制渲染进程权限）

**对比传统方案:**
```
传统 IPC 方案:
Renderer ──IPC──> Main Process ──处理──> Response

HTTP 服务方案:
Renderer ──HTTP──> Fastify Server ──处理──> Response
                        ↓
                   Worker Pool (多线程)
```

#### 9.1.3 多播放器架构

**优点:**
- 兼容性好（覆盖不同编码格式）
- 用户可自由切换
- 支持外部播放器调用

#### 9.1.4 插件化 CMS 适配器

**优点:**
- 支持多种视频源协议
- 易于扩展新协议
- Worker 隔离执行（安全性）

### 9.2 架构挑战

#### 9.2.1 性能挑战

**问题:**
- Electron 应用体积大（~200MB+）
- 内存占用高（Chromium 内核）
- 启动速度慢

**优化方案:**
```typescript
// 1. 代码分割
manualChunks: { /* ... */ }

// 2. 懒加载
const Component = () => import('./Component.vue');

// 3. 关闭不必要的功能
app.commandLine.appendSwitch('disable-renderer-backgrounding');
```

#### 9.2.2 安全性挑战

**问题:**
- 禁用了 Web Security (`disable-web-security`)
- 禁用了沙箱 (`no-sandbox`)
- 忽略证书错误 (`ignore-certificate-errors`)

**风险:**
- XSS 攻击风险
- 恶意脚本执行
- MITM 攻击

**建议:**
```typescript
// 生产环境应启用安全特性
if (import.meta.env.PROD) {
  // 移除 disable-web-security
  // 启用 sandbox
  // 验证证书
}
```

#### 9.2.3 跨平台兼容性

**问题:**
- MacOS 签名和公证
- Linux 依赖问题（libfuse2, libnss3）
- Windows 7 兼容性（需 Electron 22.x）

**解决方案:**
```yaml
# electron-builder.yml
mac:
  hardenedRuntime: true
  entitlements: build/entitlements.mac.plist

linux:
  target:
    - AppImage
    - deb  # 包含依赖
    - rpm
```

#### 9.2.4 视频播放兼容性

**问题:**
- H.265/HEVC 支持依赖浏览器
- MKV 容器格式支持有限
- 硬件解码支持不完整

**解决方案:**
```typescript
// 启用 HEVC 硬件解码
app.commandLine.appendSwitch('enable-features', 'PlatformHEVCDecoderSupport');

// 提供多播放器切换
playerMode: {
  type: 'xgplayer'  // H.265 兼容性最好
}

// 外部播放器兜底
playerMode: {
  type: 'custom',
  external: 'C:/Program Files/PotPlayer/PotPlayerMini64.exe'
}
```

### 9.3 技术债务

#### 9.3.1 TypeScript 类型覆盖

**问题:**
- 部分模块使用 `@ts-ignore`
- any 类型过多

**改进:**
```typescript
// 不推荐
const data: any = await fetchData();

// 推荐
interface DataResponse {
  code: number;
  data: SiteData[];
}
const response: DataResponse = await fetchData();
```

#### 9.3.2 错误处理

**问题:**
- 缺乏统一的错误处理机制
- 错误日志不够详细

**改进:**
```typescript
// 全局错误处理
app.on('uncaughtException', (error) => {
  logger.error('[Uncaught Exception]', error);
});

// HTTP 错误处理
server.setErrorHandler((err, request, reply) => {
  logger.error('[Fastify Error]', {
    url: request.url,
    method: request.method,
    error: err
  });
  reply.status(500).send({
    code: -1,
    msg: 'Internal Server Error',
    data: err.message
  });
});
```

#### 9.3.3 测试覆盖

**问题:**
- 缺乏单元测试
- 缺乏集成测试

**建议:**
```typescript
// 添加测试框架
import { describe, it, expect } from 'vitest';

describe('Site API', () => {
  it('should fetch site list', async () => {
    const sites = await service.site.getAll();
    expect(sites).toBeInstanceOf(Array);
  });
});
```

---

## 10. 最佳实践建议

### 10.1 代码组织

#### 10.1.1 单一职责原则

```typescript
// ❌ 不推荐：一个文件做所有事情
export const sitePage = {
  fetchData() { /* ... */ },
  renderUI() { /* ... */ },
  handleClick() { /* ... */ }
};

// ✅ 推荐：分离关注点
// api/site.ts
export const fetchSiteList = () => { /* ... */ };

// components/SiteList.vue
<template><!-- UI --></template>

// composables/useSite.ts
export const useSite = () => { /* 业务逻辑 */ };
```

#### 10.1.2 组合式 API

```typescript
// 推荐使用 Vue 3 Composition API
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

const count = ref(0);
const doubleCount = computed(() => count.value * 2);

onMounted(() => {
  console.log('Component mounted');
});
</script>
```

### 10.2 性能优化

#### 10.2.1 虚拟滚动

```typescript
// 大列表使用虚拟滚动
import { useVirtualList } from '@vueuse/core';

const { list, containerProps, wrapperProps } = useVirtualList(
  largeList,
  { itemHeight: 50 }
);
```

#### 10.2.2 图片懒加载

```typescript
// 使用 Intersection Observer
import { useIntersectionObserver } from '@vueuse/core';

const { stop } = useIntersectionObserver(
  target,
  ([{ isIntersecting }]) => {
    if (isIntersecting) {
      loadImage();
      stop();
    }
  }
);
```

### 10.3 安全建议

#### 10.3.1 输入验证

```typescript
// API 参数验证
server.post('/api/v1/site/search', {
  schema: {
    body: {
      type: 'object',
      required: ['keyword'],
      properties: {
        keyword: { type: 'string', minLength: 1, maxLength: 100 }
      }
    }
  }
}, async (request, reply) => {
  const { keyword } = request.body;
  // ...
});
```

#### 10.3.2 XSS 防护

```typescript
// 使用 DOMPurify 清理 HTML
import DOMPurify from 'dompurify';

const cleanHTML = DOMPurify.sanitize(dirtyHTML);
```

---

## 11. 总结

### 11.1 架构特点

ZyFun 项目采用了**现代化的 Electron 架构设计**，具有以下显著特点：

1. **三层架构**: Main/Renderer/Preload 清晰分离
2. **内嵌 HTTP 服务**: Fastify 提供高性能 API 服务
3. **模块化设计**: 核心功能模块化组织
4. **插件化架构**: CMS 适配器支持多种协议
5. **多播放器引擎**: 兼容各种视频格式
6. **跨平台支持**: Windows/MacOS/Linux 全平台覆盖

### 11.2 技术亮点

- **Vue 3 + TypeScript**: 类型安全的现代前端开发
- **Electron-Vite**: 快速的开发体验
- **PGlite + Drizzle**: 轻量级数据库方案
- **Worker 多线程**: 隔离执行第三方脚本
- **TDesign 组件库**: 企业级 UI 组件

### 11.3 适用场景

本项目适合作为以下场景的参考：

- Electron 桌面应用开发
- 媒体播放器开发
- 爬虫与数据解析
- 多源数据聚合
- 跨平台应用打包

### 11.4 学习路径

**初级开发者:**
1. 理解 Electron 基础概念
2. 学习 Vue 3 Composition API
3. 掌握 TypeScript 基础

**中级开发者:**
1. 深入 Electron 进程通信
2. 理解 Vite 构建原理
3. 掌握状态管理（Pinia）

**高级开发者:**
1. 性能优化与监控
2. 安全加固
3. CI/CD 自动化

---

## 12. 参考资源

### 12.1 官方文档

- [Electron 官方文档](https://www.electronjs.org/)
- [Vue 3 官方文档](https://vuejs.org/)
- [Vite 官方文档](https://vitejs.dev/)
- [TDesign 文档](https://tdesign.tencent.com/)

### 12.2 项目链接

- **GitHub**: https://github.com/Hiram-Wong/ZyPlayer
- **官方文档**: https://zy.catni.cn
- **Wiki**: https://github.com/Hiram-Wong/ZyPlayer/wiki
- **问题反馈**: https://github.com/Hiram-Wong/ZyPlayer/issues

### 12.3 相关技术

- [Electron Vite](https://electron-vite.org/)
- [Fastify](https://fastify.dev/)
- [Drizzle ORM](https://orm.drizzle.team/)
- [PGlite](https://github.com/electric-sql/pglite)

---

## 附录

### A. 常见问题

**Q: 为什么使用内嵌 HTTP 服务而不是 IPC？**
A: HTTP 服务提供更好的解耦、性能和安全性，同时便于测试。

**Q: 如何添加新的播放器？**
A: 参考现有播放器实现，在 `src/renderer/src/pages/play/` 中添加新的播放器组件。

**Q: 数据库迁移失败怎么办？**
A: 检查日志文件，确认数据库版本，必要时手动执行 SQL。

### B. 版本历史

| 版本 | 日期 | 主要变更 |
|------|------|----------|
| 3.4.1 | 2025-06-02 | 最新 Beta 版本 |
| 3.3.8 | - | IPTV 功能增强 |
| 3.3.4 | - | 播放器模式重构 |
| 3.3.2 | - | 数据库架构迁移 |

### C. 贡献者

感谢所有为项目做出贡献的开发者！

---

**文档维护**: 本文档将随项目更新持续维护
**最后更新**: 2025-11-12
**文档作者**: Claude (AI Assistant)
