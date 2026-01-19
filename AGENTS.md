# Duix.Avatar 项目上下文文档

## 项目概述

**Duix.Avatar** 是由 Duix.com 开发的免费开源 AI 数字人项目，专注于离线视频生成和数字人克隆。该项目采用 Electron 框架开发的桌面应用程序，支持 Windows 和 Ubuntu 22.04 系统。

### 核心功能

- **精确的外观和声音克隆**：使用先进的 AI 算法捕获人脸特征和声音细节
- **文本和语音驱动的虚拟数字人**：通过自然语言处理理解文本内容，或直接使用语音输入驱动数字人
- **高效视频合成**：实现数字人视频图像与声音的高度同步，实现自然流畅的唇形同步
- **多语言支持**：支持英语、日语、韩语、中文、法语、德语、阿拉伯语和西班牙语
- **完全离线运行**：无需互联网连接，保护用户隐私

### 技术栈

- **前端框架**：Vue 3 + Vue Router + Pinia
- **桌面应用**：Electron 33.0.0
- **UI 组件库**：TDesign Vue Next
- **国际化**：vue-i18n
- **数据库**：better-sqlite3
- **构建工具**：electron-vite + Vite
- **后端服务**：Docker 容器化部署（三个服务）
  - `guiji2025/fun-asr`：语音识别服务
  - `guiji2025/fish-speech-ziming`：语音合成服务
  - `guiji2025/duix.avatar`：视频生成服务

### 项目架构

```
Duix.Avatar/
├── src/
│   ├── main/           # Electron 主进程
│   │   ├── api/        # API 调用层（f2f.js, tts.js）
│   │   ├── config/     # 配置文件
│   │   ├── dao/        # 数据访问层
│   │   ├── db/         # 数据库初始化和管理
│   │   ├── handlers/   # IPC 通信处理器
│   │   ├── interval/   # 定时任务
│   │   ├── service/    # 业务逻辑层（video.js, model.js, voice.js, context.js）
│   │   └── util/       # 工具函数（ffmpeg.js）
│   ├── preload/        # 预加载脚本
│   └── renderer/       # 渲染进程（Vue 应用）
│       ├── index.html
│       └── src/
│           ├── api/    # 前端 API 调用
│           ├── assets/ # 静态资源
│           ├── client/ # 客户端工具
│           ├── components/ # Vue 组件
│           ├── i18n/   # 国际化配置
│           ├── router/ # 路由配置
│           ├── stores/ # Pinia 状态管理
│           ├── utils/  # 工具函数
│           └── views/  # 页面视图（account, home, video-edit）
├── deploy/             # Docker 部署配置
├── resources/          # 应用资源（图标、ffmpeg）
└── build/              # 构建资源
```

## 构建和运行

### 环境要求

- **Node.js**: 18.x
- **Docker**: 用于运行后端服务
- **NVIDIA 显卡**: 必需，用于 GPU 加速
- **操作系统**: Windows 10 (19042.1526+) 或 Ubuntu 22.04

### 开发命令

```bash
# 安装依赖
npm install

# 开发模式运行
npm run dev

# 构建应用
npm run build

# 构建 Windows 安装包
npm run build:win

# 构建 Linux AppImage
npm run build:linux

# 代码格式化
npm run format

# 代码检查和修复
npm run lint
```

### Docker 服务部署

```bash
# 进入部署目录
cd deploy

# 启动完整版服务（Windows）
docker-compose up -d

# 启动轻量版服务
docker-compose -f docker-compose-lite.yml up -d

# 启动 Linux 版服务
docker-compose -f docker-compose-linux.yml up -d
```

### 后端服务端口

- **视频合成服务**: `http://127.0.0.1:8383/easy`
- **语音合成服务**: `http://127.0.0.1:18180`

## 开发约定

### 代码风格

- 使用 ESLint 和 Prettier 进行代码格式化和检查
- 遵循 Vue 3 Composition API 编写组件
- 使用 TypeScript 类型注释（虽然项目主要使用 JavaScript）

### 数据库

- 使用 better-sqlite3 作为本地数据库
- 数据库文件位置：`app.getPath('userData')/biz.db`
- 数据库版本管理通过 `src/main/db/sql.js` 中的版本号控制
- DAO 层位于 `src/main/dao/` 目录

### IPC 通信

- 主进程和渲染进程通过 IPC 通信
- 主进程处理器注册在 `src/main/service/index.js`
- 预加载脚本暴露 API 在 `src/preload/index.js`
- 渲染进程通过 `window.client` 和 `window.electron` 访问主进程 API

### 配置管理

- 主进程配置文件：`src/main/config/config.js`
- 根据开发/生产环境动态配置服务 URL
- Windows 和 Linux 使用不同的数据存储路径：
  - Windows: `D:\duix_avatar_data\`
  - Linux: `~/duix_avatar_data/`

### 国际化

- 使用 vue-i18n 实现多语言支持
- 语言配置存储在 localStorage 和 Pinia store 中
- 支持中文（zh）和英文（en）

### 日志记录

- 使用 electron-log 进行日志记录
- 日志级别：info, debug, error
- 数据库操作会自动记录 SQL 语句（可通过 `{ silent: true }` 选项禁用）

## 核心模块说明

### 主进程服务（src/main/service/）

- **video.js**: 视频合成服务，处理视频生成任务
- **model.js**: 模型训练服务，处理数字人模型训练
- **voice.js**: 语音服务，处理语音克隆和合成
- **context.js**: 上下文管理，处理用户配置和状态

### API 层（src/main/api/）

- **f2f.js**: Face-to-Face 视频合成 API
- **tts.js**: 文本转语音 API
- **request.js**: HTTP 请求封装

### 渲染进程页面（src/renderer/src/views/）

- **account**: 账户相关页面
- **home**: 主页和仪表板
- **video-edit**: 视频编辑页面

## 重要注意事项

### 安全性

- 主进程中禁用了 `webSecurity` 和 `contextIsolation`（`src/main/index.js`）
- 仅用于开发环境，生产环境需要重新评估安全配置

### 文件路径处理

- 所有文件路径使用绝对路径
- 使用 `path.join()` 构建跨平台路径
- Windows 和 Linux 的数据存储路径不同

### GPU 支持

- 必须有 NVIDIA 显卡
- 需要正确安装 NVIDIA 驱动
- NVIDIA 50 系列显卡需要使用 CUDA 12.8 的 PyTorch 预览版本

### Docker 服务

- 三个服务必须全部运行才能正常工作
- 首次部署需要下载约 70GB 的镜像
- 服务启动需要约 30 分钟（取决于网络速度）

## API 接口

### 模型训练

接口：`POST http://127.0.0.1:18180/v1/speaker`

参考实现：`src/main/service/model.js`

### 音频合成

接口：`POST http://127.0.0.1:18180/v1/invoke`

参数示例：

```json
{
  "speaker": "{uuid}",
  "text": "xxxxxxxxxx",
  "format": "wav",
  "topP": 0.7,
  "max_new_tokens": 1024,
  "chunk_length": 100,
  "repetition_penalty": 1.2,
  "temperature": 0.7,
  "need_asr": false,
  "streaming": false,
  "is_fixed_seed": 0,
  "is_norm": 0,
  "reference_audio": "{voice.asr_format_audio_url}",
  "reference_text": "{voice.reference_audio_text}"
}
```

### 视频合成

接口：`POST http://127.0.0.1:8383/easy/submit`

参数示例：

```json
{
  "audio_url": "{audioPath}",
  "video_url": "{videoPath}",
  "code": "{uuid}",
  "chaofen": 0,
  "watermark_switch": 0,
  "pn": 1
}
```

进度查询：`GET http://127.0.0.1:8383/easy/query?code=${taskCode}`

## 故障排查

### 常见问题

1. **Docker 服务未运行**：检查三个服务是否都在 Running 状态
2. **GPU 驱动问题**：确认 NVIDIA 驱动已正确安装（运行 `nvidia-smi` 检查）
3. **版本不匹配**：确保服务器和客户端都是最新版本
4. **磁盘空间不足**：Windows 需要 D 盘（30GB+）和 C 盘（100GB+）

### 日志获取

- **客户端日志**：通过应用界面导出
- **服务器日志**：通过 Docker 服务日志查看

## 许可证

本项目支持全球免费商业使用（用户数超过 10 万或年收入超过 1000 万美元的企业需要签署商业许可协议）。

## 联系方式

- 官网：<https://www.duix.com>
- 问题反馈：<https://github.com/duixcom/Duix.Avatar/issues>
- 邮箱：<james@duix.com>

## 致谢

- ASR 基于 fun-asr
- TTS 基于 fish-speech-ziming
