# Copilot Instructions for Trojan Project

## 项目概述
这是一个 Go 语言编写的 Trojan 代理服务器管理工具，提供命令行和 Web 两种管理方式。

## 技术栈
- **语言**: Go 1.21+
- **Web 框架**: Gin (github.com/gin-gonic/gin)
- **认证**: JWT (github.com/appleboy/gin-jwt/v2)
- **命令行**: Cobra (github.com/spf13/cobra)
- **数据库**: LevelDB (配置存储) + MySQL/MariaDB (用户数据)
- **前端**: Vue.js (独立仓库 Jrohy/trojan-web，通过 go:embed 嵌入)

## 目录结构
- `cmd/` - Cobra 命令定义，每个文件对应一个子命令
- `core/` - 核心功能：数据库操作、配置管理、服务器基础设施
- `trojan/` - Trojan 服务相关：安装、用户管理、客户端配置生成
- `web/` - Web 服务：Gin 路由、控制器、JWT 认证
- `util/` - 工具函数：命令执行、WebSocket、字符串处理
- `asset/` - 静态资源和嵌入文件

## 编码规范
1. 函数和变量使用驼峰命名，导出函数首字母大写
2. 中文注释描述函数用途
3. 错误处理使用 `if err != nil` 模式
4. Web 响应统一使用 `controller.ResponseBody` 结构

## 关键模式
- **命令注册**: 在 `cmd/*.go` 的 `init()` 函数中使用 `rootCmd.AddCommand()`
- **路由定义**: 在 `web/web.go` 中按功能分组 (userRouter, trojanRouter, etc.)
- **配置存储**: 使用 `core.SetValue()`/`core.GetValue()` 操作 LevelDB
- **系统服务**: 使用 `util.Systemctl*()` 系列函数管理 systemd 服务

## 安全注意事项
- `/auth/register` 端点需要检查密码是否已存在（CVE-2024-55215 修复）
- JWT Token 超时时间通过 `web` 命令的 `-t` 参数配置
- 管理员密码使用 SHA256 哈希存储

## 构建说明

### 前端模板准备
项目使用 `go:embed` 嵌入前端资源，构建前需要准备 `web/templates/` 目录：
```bash
# 克隆并构建前端项目
git clone https://github.com/Jrohy/trojan-web.git frontend
cd frontend && npm install && npm run build && cd ..
mkdir -p web/templates && cp -r frontend/dist/* web/templates/
```

### 本地构建
```bash
# 获取版本信息
VERSION=$(git describe --tags $(git rev-list --tags --max-count=1))
NOW=$(TZ=Asia/Shanghai date "+%Y%m%d-%H%M")
GO_VERSION=$(go version | awk '{print $3,$4}')
GIT_VERSION=$(git rev-parse HEAD)

# 构建 Linux amd64
LDFLAGS="-w -s -X 'trojan/trojan.MVersion=$VERSION' -X 'trojan/trojan.BuildDate=$NOW' -X 'trojan/trojan.GoVersion=$GO_VERSION' -X 'trojan/trojan.GitVersion=$GIT_VERSION'"
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags "$LDFLAGS" -o trojan-linux-amd64 .

# 构建 Linux arm64
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -ldflags "$LDFLAGS" -o trojan-linux-arm64 .
```

### ldflags 变量说明
| 变量 | 说明 |
|------|------|
| `trojan/trojan.MVersion` | 版本号 (如 v2.15.7) |
| `trojan/trojan.BuildDate` | 构建日期时间 |
| `trojan/trojan.GoVersion` | Go 编译器版本 |
| `trojan/trojan.GitVersion` | Git commit hash |

### CI/CD 自动构建
- 推送 `v*` 格式的 tag 触发 GitHub Actions 自动构建
- 工作流文件: `.github/workflows/release.yml`
- 自动生成 `trojan-linux-amd64` 和 `trojan-linux-arm64` 两个二进制文件

### 构建注意事项
1. **CGO_ENABLED=0**: 必须禁用 CGO 以生成静态链接的二进制文件
2. **-w -s**: 去除调试信息和符号表，减小二进制体积
3. **前端模板**: 如果 `web/templates/` 目录不存在或为空，Web 界面将无法正常显示
4. **跨平台**: 仅支持 Linux (amd64/arm64)，trojan 原版不支持 arm64

### 快速构建脚本
使用项目自带的 `build.sh` 可快速构建并上传到 GitHub Release：
```bash
# 仅构建
./build.sh local

# 构建并上传 (需配置 github_token)
./build.sh
```

## 测试环境
- 目标系统: Linux (amd64/arm64)
- 依赖服务: Docker (MySQL)、acme.sh (证书)、systemd
