# Watch Docker

一个强大的 Docker 容器监控和自动更新工具，提供现代化的 Web 界面和 API 接口。

## 📖 概述

Watch Docker 是一个类似 Watchtower 的 Docker 容器管理工具，但提供了更强的可观测性、策略控制和用户界面。它可以自动监控运行中的容器，检测镜像更新，并支持自动或手动更新容器。

## ✨ 主要功能

### 🔍 容器监控

- **实时状态监控** - 监控所有 Docker 容器的运行状态
- **镜像更新检测** - 自动检查远端镜像仓库的更新
- **资源使用监控** - 实时显示容器的 CPU 和内存使用情况
- **详细日志查看** - 支持实时查看容器日志 （待实现）

### 🔄 自动更新

- **智能更新策略** - 支持多种跳过和强制策略
- **定时更新** - 支持 Cron 表达式和间隔时间调度
- **安全回滚** - 更新失败时自动回滚到原容器
- **批量操作** - 支持一键批量更新多个容器

### 🎯 策略控制

- **标签策略** - 通过 label 控制容器是否跳过或强制更新
- **版本固定** - 自动识别并跳过固定版本的镜像
- **本地构建** - 自动跳过本地构建的镜像
- **Compose 保护** - 支持跳过 Docker Compose 管理的容器

### 🌐 现代化界面

- **响应式设计** - 完美支持桌面和移动设备
- **实时数据** - WebSocket 连接提供实时更新
- **直观操作** - 简洁易用的用户界面
- **多主题支持** - 支持亮色和暗色主题

### 🐳 Docker Compose 管理

- **项目发现** - 自动扫描并识别 Docker Compose 项目
- **统一管理** - 查看、启动、停止、重启 Compose 项目
- **服务监控** - 实时查看项目中各服务的运行状态
- **日志查看** - 通过 WebSocket 实时查看项目日志

### 💻 Shell 终端访问

- **交互式终端** - 通过 Web 界面访问容器主机 Shell
- **安全控制** - 需要身份验证和显式配置才能开启
- **完整 TTY** - 支持彩色输出和交互式命令
- **实时通信** - 基于 WebSocket 的实时终端通信

## ⚠️ 风险提示

在使用本工具前，请仔细阅读以下风险提示：

### 🔐 安全风险

- **高权限访问** - 本工具需要访问 Docker socket (`/var/run/docker.sock`)，这意味着它拥有对宿主机 Docker 守护进程的完全控制权限
- **容器逃逸风险** - 任何能够访问 Docker socket 的容器理论上都可以访问宿主机系统，请确保：
  - 仅在受信任的环境中运行
  - 使用强密码保护 Web 界面
  - 限制网络访问（如使用防火墙规则）
- **Shell 访问风险** - 如果开启 Shell 终端功能，将提供对宿主机的直接命令行访问，这是极其危险的：
  - ⚠️ **切勿在生产环境或公网暴露的环境中开启**
  - ⚠️ 必须使用强密码并启用身份验证
  - ⚠️ 建议通过 VPN 或内网访问
  - ⚠️ 定期审查是否仍需要此功能

### 🔄 更新风险

- **服务中断** - 自动更新容器会导致服务短暂中断，可能影响业务连续性
- **镜像兼容性** - 新版本镜像可能包含破坏性变更，导致应用无法正常运行
- **配置丢失** - 如果容器配置不当（如未正确挂载卷），更新可能导致数据丢失
- **网络变更** - 重建容器可能改变容器的网络配置（如 IP 地址）

### ⚡ 特别注意

- 请勿在生产环境开启过于激进的自动更新策略
- 对于数据库、消息队列等有状态服务，建议设置 `watchdocker.skip=true`
- 更新前请确认新版本的 Release Notes 和变更日志

> **免责声明：本工具仅供学习和测试使用。使用本工具导致的任何直接或间接损失，开发者不承担任何责任。生产环境使用请自行评估风险。**

## 🚀 快速开始

### Docker Compose（推荐）

创建 `docker-compose.yaml` 文件：

```yaml
services:
  watch-docker:
    image: jianxcao/watch-docker:latest
    container_name: watch-docker
    hostname: watch-docker
    labels:
      - "watchdocker.skip=true" # 避免自己更新自己
    ports:
      - "8080:8080"
    volumes:
      - ./config:/config
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - TZ=Asia/Shanghai
      - USER_NAME=admin
      - USER_PASSWORD=admin
    restart: unless-stopped
```

启动服务：

```bash
docker-compose up -d
```

### Docker 命令

```bash
docker run -d \
  --name watch-docker \
  -p 8080:8080 \
  -v ./config:/config \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -e TZ=Asia/Shanghai \
  -e USER_NAME=admin \
  -e USER_PASSWORD=admin \
  --label watchdocker.skip=true \
  jianxcao/watch-docker:latest
```

访问 `http://localhost:8080` 并使用默认账户 `admin/admin` 登录。

## ⚙️ 配置

### 环境变量

| 变量名                 | 默认值          | 描述                                 |
| ---------------------- | --------------- | ------------------------------------ |
| `CONFIG_PATH`          | `/config`       | 配置文件目录                         |
| `CONFIG_FILE`          | `config.yaml`   | 配置文件名                           |
| `USER_NAME`            | `admin`         | 登录用户名                           |
| `USER_PASSWORD`        | `admin`         | 登录密码                             |
| `TZ`                   | `Asia/Shanghai` | 时区设置                             |
| `PORT`                 | `8088`          | 服务端口                             |
| `IS_OPEN_DOCKER_SHELL` | `false`         | 是否开启 Shell 终端功能（⚠️ 高风险） |

### 配置文件示例

在 `./config/config.yaml` 中配置：

```yaml
server:
  addr: ":8080"

docker:
  host: "unix:///var/run/docker.sock"
  includeStopped: false

scan:
  interval: "10m" # 扫描间隔
  initialScanOnStart: true # 启动时立即扫描
  concurrency: 3 # 并发数
  cacheTTL: "5m" # 缓存时间

update:
  enabled: true # 启用自动更新
  autoUpdateCron: "0 3 * * *" # 每天凌晨3点自动更新
  allowComposeUpdate: false # 是否允许更新 Compose 容器
  removeOldContainer: true # 更新后删除旧容器

policy:
  skipLabels: ["watchdocker.skip=true"] # 跳过标签
  skipLocalBuild: true # 跳过本地构建
  skipPinnedDigest: true # 跳过固定 digest
  skipSemverPinned: true # 跳过语义化版本

registry:
  auth:
    - host: "registry-1.docker.io"
      username: ""
      password: ""
    - host: "ghcr.io"
      username: ""
      password: ""

logging:
  level: "info"
```

## 🏷️ 容器标签

通过以下标签控制容器更新行为：

```yaml
# 跳过更新
labels:
  - "watchdocker.skip=true"

# 强制更新（即使是固定版本）
labels:
  - "watchdocker.force=true"

# 在更新开关打开的情况下，只跳过更新，不跳过检测
labels:
  - "watchdocker.skipUpdate=true"
```

## 📚 API 文档

### 主要端点

**容器管理**

- `GET /api/containers` - 获取所有容器状态
- `POST /api/containers/:id/update` - 更新指定容器
- `POST /api/containers/:id/start` - 启动容器
- `POST /api/containers/:id/stop` - 停止容器
- `DELETE /api/containers/:id` - 删除容器
- `POST /api/updates/run` - 批量更新
- `GET /api/images` - 获取镜像列表

**Compose 管理**

- `GET /api/compose` - 获取 Compose 项目列表
- `POST /api/compose/start` - 启动 Compose 项目
- `POST /api/compose/stop` - 停止 Compose 项目
- `POST /api/compose/restart` - 重启 Compose 项目
- `DELETE /api/compose/delete` - 删除 Compose 项目
- `POST /api/compose/create` - 创建 Compose 项目
- `GET /api/compose/logs/ws` - Compose 日志 WebSocket

**终端访问**

- `GET /api/shell/ws` - Shell 终端 WebSocket

**其他**

- `GET /healthz` - 健康检查

### 响应格式

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "containers": [...],
    "total": 10
  }
}
```

## 🐳 Docker Compose 管理

Watch Docker 提供了完整的 Docker Compose 项目管理功能，让你可以通过 Web 界面统一管理所有 Compose 项目。

### 功能特性

- **自动发现** - 自动扫描指定目录，发现所有 `docker-compose.yml` 或 `compose.yml` 文件
- **项目管理** - 启动、停止、重启、删除 Compose 项目
- **状态监控** - 实时查看项目及其服务的运行状态
- **日志查看** - 通过 WebSocket 实时查看项目日志
- **服务详情** - 查看项目中的服务、网络、卷等详细信息

### 配置 Compose 项目路径

Docker 挂载 Compose 项目目录：

```yaml
services:
  watch-docker:
    image: jianxcao/watch-docker:latest
    volumes:
      - ./config:/config
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /opt/compose-projects:/compose-projects:ro # 挂载 Compose 项目目录,注意左右 2 侧需要相同
    environment:
      - /opt/compose-projects
```

### 项目状态

Compose 项目具有以下几种状态：

- **running** - 所有服务都在运行
- **stopped** - 所有服务都已停止
- **partial** - 部分服务在运行
- **error** - 项目存在错误

### 使用建议

1. **目录结构** - 建议将每个 Compose 项目放在独立的目录中
2. **命名规范** - 使用有意义的项目名称和目录名
3. **权限控制** - 确保 Watch Docker 有权限访问 Compose 项目目录
4. **备份配置** - 建议定期备份 Compose 配置文件

## 💻 Shell 终端访问

Watch Docker 提供了通过 Web 界面访问容器主机 Shell 的功能，方便进行调试和管理操作。

### ⚠️ 安全警告

**Shell 功能具有极高的安全风险，请务必仔细阅读以下警告：**

- ⚠️ Shell 访问提供了对宿主机的直接命令行访问权限
- ⚠️ 可以执行任何系统命令，包括危险操作（如删除文件、修改配置等）
- ⚠️ 可以访问所有挂载的 Docker Socket，具有完全的 Docker 控制权限
- ⚠️ 如果被恶意利用，可能导致严重的安全事故
- ⚠️ 仅在完全信任的环境中使用此功能
- ⚠️ 必须使用强密码并启用身份验证

> **严重警告：不要在生产环境或公网暴露的环境中开启此功能！**

### 启用 Shell 功能

Shell 功能默认关闭，需要满足以下条件才能开启：

1. **必须启用身份验证** - 必须设置 `USER_NAME` 和 `USER_PASSWORD`
2. **显式开启功能** - 必须设置环境变量 `IS_OPEN_DOCKER_SHELL=true`

通过环境变量启用：

```bash
docker run -d \
  --name watch-docker \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -e USER_NAME=admin \
  -e USER_PASSWORD=your_strong_password \
  -e IS_OPEN_DOCKER_SHELL=true \
  jianxcao/watch-docker:latest
```

或通过 Docker Compose：

```yaml
services:
  watch-docker:
    image: jianxcao/watch-docker:latest
    environment:
      - USER_NAME=admin
      - USER_PASSWORD=your_strong_password
      - IS_OPEN_DOCKER_SHELL=true # 开启 Shell 功能
```

### 功能特性

- **交互式终端** - 完整的 PTY (伪终端) 支持
- **彩色输出** - 支持 ANSI 颜色和格式化输出
- **实时通信** - 基于 WebSocket 的低延迟通信
- **中文支持** - 支持中文字符显示（UTF-8）
- **会话管理** - 自动处理终端大小调整和会话超时

### 访问方式

启用后，在 Web 界面中可以找到 "终端" 或 "Shell" 菜单，点击即可打开交互式终端。

终端使用以下配置：

- **默认 Shell**: 使用系统环境变量 `$SHELL`，如未设置则使用 `/bin/sh`
- **终端类型**: `xterm-256color`
- **字符编码**: `UTF-8 (zh_CN.UTF-8)`
- **心跳检测**: 30 秒
- **会话超时**: 90 秒无活动后断开

### 安全建议

1. **强密码** - 使用复杂的密码，至少 16 位，包含大小写字母、数字和特殊字符
2. **网络隔离** - 使用防火墙规则限制访问 IP 范围
3. **审计日志** - 定期检查系统日志，监控异常活动
4. **最小权限** - 如果可能，使用受限的用户账户而非 root
5. **VPN 访问** - 建议通过 VPN 访问而非直接暴露在公网
6. **定期审查** - 定期审查是否仍需要此功能，不用时应关闭

### 禁用 Shell 功能

如果不再需要 Shell 功能，强烈建议将其关闭：

```bash
# 移除环境变量或设置为 false
-e IS_OPEN_DOCKER_SHELL=false
```

或重启容器时不传递该环境变量。

## 🔧 开发

### 技术栈

**后端:**

- Go 1.25+
- Gin Web 框架
- Docker SDK
- Zap 日志库
- Cron 调度器

**前端:**

- Vue 3 + TypeScript
- Naive UI 组件库
- Pinia 状态管理
- Vite 构建工具
- UnoCSS 样式框架

### 本地开发

1. **克隆仓库**

```bash
git clone https://github.com/jianxcao/watch-docker.git
cd watch-docker
```

2. **启动后端**

```bash
cd backend
go mod download
go run cmd/watch-docker/main.go
```

3. **启动前端**

```bash
cd frontend
pnpm install
pnpm dev
```

4. **构建**

```bash
# 后端构建
cd backend && go build -o watch-docker cmd/watch-docker/main.go

# 前端构建
cd frontend && pnpm build

# Docker 构建
docker build -t watch-docker .
```

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交改动 (`git commit -m 'Add some amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 创建 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 🙏 致谢

- 感谢 [Watchtower](https://github.com/containrrr/watchtower) 项目的启发
- 感谢所有贡献者的支持

## 📞 支持

- 提交 [Issue](https://github.com/jianxcao/watch-docker/issues)
- 查看 [Wiki](https://github.com/jianxcao/watch-docker/wiki)
- 关注项目获取最新动态

---

⭐ 如果这个项目对你有帮助，请给个 Star 支持一下！
