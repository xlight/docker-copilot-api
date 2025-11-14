# Docker Copilot API

[![Docker Build](https://github.com/xlight/docker-copilot-api/actions/workflows/docker-build.yml/badge.svg)](https://github.com/xlight/docker-copilot-api/actions/workflows/docker-build.yml)
[![Docker Image Size](https://img.shields.io/docker/image-size/xlight/copilot-api/latest)](https://hub.docker.com/r/xlight/copilot-api)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

这是一个基于 GitHub Actions 自动构建的 Docker 镜像项目，封装了 [copilot-api](https://github.com/ericc-ch/copilot-api) 服务。该服务可以将 GitHub Copilot 转换为 OpenAI/Anthropic API 兼容的服务器。

## 🚀 特性

- 🤖 将 GitHub Copilot 转换为 OpenAI/Anthropic 兼容 API
- 🐳 完全容器化，开箱即用
- 🔄 通过 GitHub Actions 自动构建和推送 Docker 镜像
- 📊 内置使用情况监控面板
- 🔒 支持多种 GitHub Copilot 账户类型（个人版、商业版、企业版）
- ⚡ 支持速率限制和手动请求批准
- 🌐 同时支持 OpenAI 和 Anthropic API 格式

## 📋 前置要求

- Docker 和 Docker Compose（如果本地运行）
- GitHub 账户并订阅了 Copilot（个人版、商业版或企业版）
- （可选）GitHub Token 用于非交互式部署

## 🎯 快速开始

### 使用 Docker Hub 镜像

```bash
# 拉取最新镜像
docker pull xlight/copilot-api:latest

# 创建数据持久化目录
mkdir -p ./copilot-data

# 运行容器
docker run -d \
  --name copilot-api \
  -p 4141:4141 \
  -v $(pwd)/copilot-data:/root/.local/share/copilot-api \
  xlight/copilot-api:latest
```

### 使用 Docker Compose

创建 `docker-compose.yml` 文件：

```yaml
version: "3.8"

services:
  copilot-api:
    image: xlight/copilot-api:latest
    container_name: copilot-api
    ports:
      - "4141:4141"
    volumes:
      - ./copilot-data:/root/.local/share/copilot-api
    environment:
      # 可选：直接提供 GitHub Token
      # - GH_TOKEN=your_github_token_here
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:4141/"]
      interval: 30s
      timeout: 5s
      retries: 3
```

然后运行：

```bash
docker-compose up -d
```

### 本地构建

```bash
# 克隆本项目
git clone https://github.com/xlight/docker-copilot-api.git
cd docker-copilot-api

# 构建镜像
docker build -t copilot-api .

# 运行容器
docker run -d -p 4141:4141 -v $(pwd)/copilot-data:/root/.local/share/copilot-api copilot-api
```

## 🔐 认证方式

### 交互式认证（推荐用于本地开发）

首次运行时，容器会引导您完成 GitHub 认证流程。Token 会保存在挂载的卷中，后续重启会自动使用。

```bash
docker run -it -p 4141:4141 -v $(pwd)/copilot-data:/root/.local/share/copilot-api xlight/copilot-api:latest
```

### 使用环境变量（推荐用于生产环境）

```bash
# 使用 GitHub Token
docker run -d \
  -p 4141:4141 \
  -e GH_TOKEN=your_github_token_here \
  xlight/copilot-api:latest
```

或在 Docker Compose 中：

```yaml
environment:
  - GH_TOKEN=your_github_token_here
```

## 📡 API 端点

### OpenAI 兼容端点

| 端点 | 方法 | 描述 |
|------|------|------|
| `/v1/chat/completions` | POST | 创建聊天补全 |
| `/v1/models` | GET | 列出可用模型 |
| `/v1/embeddings` | POST | 创建文本嵌入向量 |

### Anthropic 兼容端点

| 端点 | 方法 | 描述 |
|------|------|------|
| `/v1/messages` | POST | 创建消息响应 |
| `/v1/messages/count_tokens` | POST | 计算消息的 token 数量 |

### 监控端点

| 端点 | 方法 | 描述 |
|------|------|------|
| `/usage` | GET | 获取详细的使用统计和配额信息 |
| `/token` | GET | 获取当前使用的 Copilot Token |

## 💡 使用示例

### 使用 curl

```bash
# 测试 API
curl http://localhost:4141/v1/models

# 发送聊天请求
curl -X POST http://localhost:4141/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy" \
  -d '{
    "model": "gpt-4",
    "messages": [
      {"role": "user", "content": "Hello!"}
    ]
  }'
```

### 使用 Python

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:4141/v1",
    api_key="dummy"  # API key 可以是任意值
)

response = client.chat.completions.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "Hello!"}
    ]
)

print(response.choices[0].message.content)
```

### 使用 Claude Code

配置 Claude Code 使用此代理服务，在项目根目录创建 `.claude/settings.json`：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:4141",
    "ANTHROPIC_AUTH_TOKEN": "dummy",
    "ANTHROPIC_MODEL": "gpt-4.1",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "gpt-4.1",
    "ANTHROPIC_SMALL_FAST_MODEL": "gpt-4.1",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "gpt-4.1",
    "DISABLE_NON_ESSENTIAL_MODEL_CALLS": "1",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  },
  "permissions": {
    "deny": ["WebSearch"]
  }
}
```

## 📊 使用监控面板

启动服务后，访问以下 URL 查看使用情况：

```
https://ericc-ch.github.io/copilot-api?endpoint=http://localhost:4141/usage
```

面板功能：
- 📈 实时使用配额显示
- 📊 详细的 API 使用统计
- 🎯 进度条可视化展示
- 🔄 一键刷新数据

## ⚙️ 高级配置

### 环境变量

| 变量 | 描述 | 默认值 |
|------|------|--------|
| `GH_TOKEN` | GitHub Token | - |
| `PORT` | 服务监听端口 | 4141 |

### 运行参数

在 Dockerfile 的 ENTRYPOINT 中可以传递额外参数：

```bash
# 启用详细日志
docker run -d -p 4141:4141 xlight/copilot-api:latest start --verbose

# 设置账户类型为商业版
docker run -d -p 4141:4141 xlight/copilot-api:latest start --account-type business

# 启用速率限制（30秒间隔）
docker run -d -p 4141:4141 xlight/copilot-api:latest start --rate-limit 30 --wait

# 启用手动批准模式
docker run -d -p 4141:4141 xlight/copilot-api:latest start --manual
```

可用参数：
- `--port <number>`: 指定监听端口
- `--verbose`: 启用详细日志
- `--account-type <type>`: 账户类型（individual/business/enterprise）
- `--rate-limit <seconds>`: 请求之间的最小间隔（秒）
- `--wait`: 遇到速率限制时等待而非报错
- `--manual`: 启用手动请求批准
- `--show-token`: 显示 GitHub 和 Copilot Token

## 🔄 CI/CD 自动构建

本项目使用 GitHub Actions 自动构建和推送 Docker 镜像。每次推送到 `main` 分支或创建新的 tag 时，会自动触发构建流程。

### 构建触发条件

- 推送到 `main` 分支
- 创建 `v*` 格式的 tag（如 `v1.0.0`）
- 手动触发 workflow

### 镜像标签策略

- `latest`: 最新的 main 分支构建
- `v*`: 版本标签（如 `v1.0.0`）
- `<commit-sha>`: 特定提交的构建

### 配置 GitHub Actions

1. 在 GitHub 仓库设置中添加以下 Secrets：
   - `DOCKERHUB_USERNAME`: Docker Hub 用户名
   - `DOCKERHUB_TOKEN`: Docker Hub 访问令牌

2. 推送代码或创建 tag 以触发构建：

```bash
# 推送到 main 分支
git push origin main

# 创建并推送版本标签
git tag v1.0.0
git push origin v1.0.0
```

## 🛠️ 故障排除

### 容器无法启动

检查日志：
```bash
docker logs copilot-api
```

### 认证失败

1. 确保您的 GitHub 账户已订阅 Copilot
2. 如果使用 GH_TOKEN，确保 token 有效且具有正确的权限
3. 删除持久化数据并重新认证：
```bash
rm -rf ./copilot-data
docker restart copilot-api
```

### 端口冲突

修改映射端口：
```bash
docker run -d -p 8080:4141 xlight/copilot-api:latest
```

### 健康检查失败

检查容器是否正常运行：
```bash
docker inspect copilot-api
curl http://localhost:4141/v1/models
```

## 📝 开发指南

### 本地开发

```bash
# 克隆原始仓库
git clone https://github.com/ericc-ch/copilot-api.git
cd copilot-api

# 安装依赖
bun install

# 开发模式运行
bun run dev

# 构建
bun run build
```

### 贡献

欢迎提交 Pull Request！在提交之前，请确保：

1. 代码通过所有测试
2. 更新相关文档
3. 遵循现有的代码风格

## ⚠️ 注意事项

> **警告**: 这是一个反向工程的 GitHub Copilot API 代理。它不受 GitHub 官方支持，可能会意外中断。使用风险自负。

> **GitHub 安全提示**: 过度的自动化或脚本化使用 Copilot（包括快速或批量请求）可能会触发 GitHub 的滥用检测系统。您可能会收到 GitHub Security 的警告，进一步的异常活动可能导致您的 Copilot 访问被暂时停用。

请负责任地使用此代理，避免账户受限：
- 使用 `--rate-limit` 参数限制请求频率
- 使用 `--manual` 模式手动控制请求
- 避免批量或过度的自动化请求

相关文档：
- [GitHub 可接受使用政策](https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies)
- [GitHub Copilot 条款](https://github.com/features/copilot)

## 📜 许可证

本项目基于 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

原始 copilot-api 项目：[ericc-ch/copilot-api](https://github.com/ericc-ch/copilot-api)

## 🙏 致谢

- 感谢 [ericc-ch](https://github.com/ericc-ch) 创建的优秀的 [copilot-api](https://github.com/ericc-ch/copilot-api) 项目
- 感谢所有为原项目贡献的开发者

## 📞 支持

- 原项目 Issues: https://github.com/ericc-ch/copilot-api/issues
- 本项目 Issues: https://github.com/xlight/docker-copilot-api/issues

## 🔗 相关链接

- [Docker Hub](https://hub.docker.com/r/xlight/copilot-api)
- [GitHub Repository](https://github.com/xlight/docker-copilot-api)
- [原始项目](https://github.com/ericc-ch/copilot-api)
- [使用监控面板](https://ericc-ch.github.io/copilot-api)
