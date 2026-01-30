# IM Application 项目 README

一款基于 C++ 高性能后端（集成 Drogon 框架）与 Vue3 + Electron 跨平台前端构建的即时通讯应用，支持实时消息收发、用户在线状态同步等核心功能。全程适配 WSL2+Ubuntu 开发环境，采用 Docker 简化部署流程，技术栈选型兼顾性能与跨平台兼容性。

## 📁 项目结构

```bash

im_app/
├── im_server/          # 后端服务 (C++ / CMake 构建，集成 Drogon 框架)
│   ├── controller/     # Drogon 控制器（路由处理/业务逻辑）
│   ├── filter/         # Drogon 过滤器（鉴权/日志/参数校验）
│   ├── model/          # Drogon 数据模型（MySQL 表映射）
│   ├── db/             # 数据库初始化脚本 (MySQL + Redis)
│   ├── config.json     # 核心配置文件 (JSON 格式，配置数据库/端口/Socket/Drogon等)
│   ├── Dockerfile      # 后端容器化构建文件
│   └── docker-compose.yml # 后端服务编排 (MySQL + Redis + IM服务)
├── im_client/          # 前端客户端 (Vue3 + Electron + Vite + Socket)
│   ├── src/
│   │   ├── main/       # Electron 主进程
│   │   ├── preload/    # 预加载脚本 (隔离主/渲染进程，提供Socket基础桥接)
│   │   └── renderer/   # Vue渲染进程 (含Socket客户端封装、页面组件等)
│   ├── package.json    # 前端依赖配置
│   └── vite.config.ts  # Vite 构建配置
├── setup_frontend.sh   # 前端环境一键配置脚本 (适配 WSL2 Ubuntu 环境)
└── README.md           # 项目说明文档 (本文档)
```

## 🚀 快速开始

### 前置环境准备 (WSL2 + Ubuntu)

确保你的 WSL2 (推荐 Ubuntu 20.04/22.04) 中已安装以下基础工具，所有操作均在 WSL2 Ubuntu 环境下执行：

1. **Docker & Docker Compose**`# Ubuntu 下安装 Docker 及 Compose 插件
sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo usermod -aG docker $USER  # 免 sudo 运行 Docker，需重启 WSL2 生效
# 重启 WSL2 (在 Windows 终端执行，非 Ubuntu 内)
wsl --shutdown
wsl`

2. **VSCode 及必备插件**本地安装 VSCode，安装以下核心插件：

    - Remote - WSL：连接 WSL2 开发环境

    - C/C++：后端 C++ 代码编译与调试

    - Vue - Official：前端 Vue3 语法高亮与提示

    - Docker：容器可视化管理

    - Prettier：代码格式化，保持风格统一

3. **Node.js (v18+)**`# 安装 nvm 版本管理工具 (推荐)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc  # 生效 nvm 命令
# 安装并使用 Node.js 18.x 版本
nvm install 18
nvm use 18
# 验证安装
node -v  # 输出 v18.x.x 即为成功
npm -v   # 输出对应版本号`

### 1. 后端服务 (im_server) 部署

后端基于 C++17 开发，集成 Drogon 高性能 HTTP/WebSocket 框架，采用 CMake 构建，支持两种部署方式：Docker 容器化（推荐，无需手动配置依赖）、本地原生编译（适合调试）。

#### 方式 A: Docker 启动 (推荐，适配 WSL2 Ubuntu)

1. 进入后端目录
        `cd im_app/im_server`

2. 构建并启动服务（自动拉起 MySQL、Redis 依赖服务，内置 Drogon 框架运行环境）`docker-compose up --build -d`

3. 启动成功验证
       服务启动后，各组件端口映射（宿主机可访问）：

    - IM 服务 Socket 端口：**9090**（核心，实时消息传输，基于 Drogon WebSocket 实现）

    - IM 服务辅助 API 端口：**8080**（用户注册、登录等接口，基于 Drogon HTTP 实现）

    - MySQL 数据库端口：**3307**（用户名/密码在 config.json 中配置）

    - Redis 缓存端口：**6379**（缓存用户在线状态、临时消息）

4. 查看服务运行状态`docker-compose ps  # 查看所有服务状态
` `docker-compose logs -f  # 实时查看服务日志（含 Drogon 框架运行日志）`

#### 方式 B: 本地编译运行 (WSL2 Ubuntu 原生环境)

适合后端开发调试，需手动安装所有依赖库（含 Drogon 框架及相关依赖）：

1. 安装依赖包`sudo apt update
sudo apt install -y cmake g++ libmysqlclient-dev libhiredis-dev \
libssl-dev libjsoncpp-dev libuuid-dev zlib1g-dev libtrantor-dev git
# 编译安装 Drogon 框架（适配 C++17）
git clone https://github.com/drogonframework/drogon.git
cd drogon && mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DUSE_BOOST=OFF ..
make -j$(nproc) && sudo make install
cd ../.. && rm -rf drogon
# 验证依赖版本 (需满足：g++ 支持 C++17+，CMake ≥ 3.16，Drogon ≥ 1.8.0)
g++ --version
cmake --version
` `drogon_ctl --version`

2. 编译项目`cd im_app/im_server
mkdir build && cd build
cmake ..  # 生成编译配置（自动关联 Drogon 库）
` `make -j$(nproc)  # 多线程编译，提升速度`

3. 启动服务`# 启动前需确保本地 MySQL、Redis 已启动并配置正确
./my_im_server  # 运行编译产物（Drogon 框架初始化并启动服务）
` `# 配置修改：需编辑 im_server/config.json，将数据库连接地址改为 localhost`

### 2. 前端客户端 (im_client) 运行

前端基于 Vue3 + Electron 构建，集成 Socket 客户端实现与后端 Drogon WebSocket 的实时消息交互，提供一键配置脚本简化操作。

#### 方式 A: 一键配置与运行 (推荐)

```bash

# 回到项目根目录
cd im_app
# 赋予脚本执行权限
chmod +x setup_frontend.sh
# 运行脚本 (自动安装依赖、检查环境、启动开发模式)
bash setup_frontend.sh
```

#### 方式 B: 手动运行

1. 进入前端目录
        `cd im_app/im_client`

2. 安装依赖
        `npm install  # 安装所有前端依赖`

3. 启动开发模式
        `npm run dev`

4. 运行效果
        启动后将自动打开两个进程：

    - Vite 开发服务器：提供前端页面热更新支持

    - Electron 窗口：前端应用主界面，自动连接后端 Drogon WebSocket (ws://localhost:9090)

## 🛠️ 技术栈详情

|分类|技术选型|核心作用|
|---|---|---|
|开发环境|WSL2 + Ubuntu + VSCode|跨平台开发底座，兼顾 Linux 后端编译与前端开发，VSCode 插件提供一站式支持|
|前端技术|Vue 3 + TypeScript|构建高效、可维护的前端页面，TypeScript 保障类型安全|
||Electron|实现 Windows/Mac/Linux 跨平台桌面应用，整合主进程与渲染进程|
||Vite|快速构建与热更新，大幅提升前端开发效率|
||Socket|前端与后端 Drogon WebSocket 双向实时通信，支撑即时消息核心功能|
|后端技术|C++|高性能后端开发语言，保障服务并发处理能力|
||Drogon|高性能 C++ Web 框架，提供 HTTP/WebSocket 服务能力，简化网络层开发|
||CMake|跨平台构建工具，简化 C++ 项目编译与依赖管理（含 Drogon 库关联）|
||MySQL|关系型数据库，持久化存储用户信息、会话记录、聊天历史等核心数据|
||Redis|缓存数据库，提升用户在线状态、临时消息等场景的响应速度|
||JSON|轻量数据交换格式，用于后端配置文件与前后端数据传输|
|部署工具|Docker|容器化打包服务与依赖（含 Drogon 运行环境），实现环境隔离、一键部署|
## 📝 开发指南

### 1. VSCode 开发配置

- **后端调试配置**：在 im_server 目录下创建 .vscode/launch.json，配置 CMake 调试参数，支持 Drogon 服务断点调试、变量监控。

- **前端调试配置**：Electron 开发模式下，可直接在 VSCode 中调试渲染进程，支持页面断点、控制台输出查看。

- **代码格式化**：启用 Prettier 插件，配置保存自动格式化，保持前后端代码风格统一。

### 2. 配置文件说明

- 后端核心配置：`im_server/config.json`，可修改以下关键配置：
        

    - Socket 监听端口、连接超时时间（Drogon WebSocket 配置）

    - MySQL 连接地址、端口、用户名、密码、数据库名

    - Redis 连接地址、端口、密码、数据库索引

    - 日志输出路径、日志级别（Drogon 日志配置）

- 前端 Socket 配置：`im_client/src/renderer/utils/socket.ts`，配置后端 Socket 连接地址（默认 ws://localhost:9090）、重连机制参数。

### 3. Git 提交规范

- 已配置 .gitignore 文件，自动忽略以下目录/文件：
        

    - 前端：node_modules/、dist/、.env*、electron-builder 产物

    - 后端：build/、编译产物、日志文件、临时配置、Drogon 编译中间文件

    - Docker：容器日志、镜像缓存文件

- 提交信息格式：`[模块] 操作描述`，示例：
        

    - [im_server] 修复 Drogon WebSocket 断连重连异常问题

    - [im_client] 新增聊天页面表情发送功能，适配后端 Drogon 消息格式

    - [docs] 更新 README 环境配置步骤

### 4. 常见问题排查

- **Docker 启动失败**：检查 WSL2 虚拟化是否开启，执行 `sudo systemctl status docker` 查看服务状态，修复权限或依赖问题；确认 Drogon 镜像构建依赖是否完整。

- **前端 Socket 连接失败**：确认后端服务已启动，9090 端口未被占用（执行 `netstat -an | grep 9090` 检查），WSL2 端口映射正常。

- **后端本地编译失败**：核对依赖库是否安装完整（重点检查 Drogon 及 trantor 依赖），C++ 编译器是否支持 C++17+，CMake 版本是否达标。

- **数据库连接异常**：检查 MySQL/Redis 服务是否启动，配置文件中的连接地址、端口、用户名密码是否正确；确认 Drogon 框架数据库配置项无误。

## 📦 后续部署

1. **后端生产部署**：`# 构建生产环境镜像（包含 Drogon 生产环境依赖）
cd im_app/im_server
docker-compose -f docker-compose.prod.yml build
# 启动生产服务
` `docker-compose -f docker-compose.prod.yml up -d`

2. **前端打包分发**：
        `cd im_app/im_client
# 打包对应平台的可执行文件 (Windows/Mac/Linux)
npm run build:win
npm run build:mac
npm run build:linux`

3. **数据备份策略**：
        

    - MySQL：配置定时备份脚本，定期导出数据库文件。

    - Redis：开启 RDB/AOF 持久化，防止数据丢失。

    - Drogon 日志：配置日志轮转策略，避免日志文件过大。