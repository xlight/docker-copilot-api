# 快速开始指南

5 分钟快速部署 Docker Copilot API！

## 🚀 最快速的方式

如果你只是想快速测试，运行以下命令：

```bash
# 拉取镜像
docker pull xlight/copilot-api:latest

# 创建数据目录
mkdir -p copilot-data

# 启动容器（交互式）
docker run -it \
  -p 4141:4141 \
  -v $(pwd)/copilot-data:/root/.local/share/copilot-api \
  xlight/copilot-api:latest
```

按照提示完成 GitHub 认证，然后访问 `http://localhost:4141/v1/models` 测试！

---

## 📝 步骤详解

### 1️⃣ 前提条件

- ✅ 已安装 Docker
- ✅ 有 GitHub Copilot 订阅

验证 Docker 安装：
```bash
docker --version
```

### 2️⃣ 首次启动

```bash
# 创建持久化目录
mkdir -p copilot-data

# 启动容器（首次需要交互式）
docker run -it \
  --name copilot-api \
  -p 4141:4141 \
  -v $(pwd)/copilot-data:/root/.local/share/copilot-api \
  xlight/copilot-api:latest
```

### 3️⃣ GitHub 认证

容器启动后会显示类似信息：

```
Please visit: https://github.com/login/device
Enter code: XXXX-XXXX
```

1. 在浏览器打开显示的 URL
2. 输入显示的代码
3. 授权应用访问
4. 返回终端等待完成

### 4️⃣ 测试 API

打开新终端，运行：

```bash
# 测试连接
curl http://localhost:4141/v1/models

# 发送聊天请求
curl -X POST http://localhost:4141/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy" \
  -d '{
    "model": "gpt-4",
    "messages": [{"role": "user", "content": "写一个 Python 的 Hello World"}]
  }'
```

### 5️⃣ 查看使用情况

访问监控面板：
```
https://ericc-ch.github.io/copilot-api?endpoint=http://localhost:4141/usage
```

或使用命令行：
```bash
curl http://localhost:4141/usage | jq
```

---

## 🔄 后续使用

认证完成后，可以使用后台模式运行：

```bash
# 停止交互式容器
docker stop copilot-api
docker rm copilot-api

# 后台启动
docker run -d \
  --name copilot-api \
  -p 4141:4141 \
  -v $(pwd)/copilot-data:/root/.local/share/copilot-api \
  --restart unless-stopped \
  xlight/copilot-api:latest

# 查看日志
docker logs -f copilot-api
```

---

## 🐳 使用 Docker Compose（推荐）

### 创建 docker-compose.yml

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
    restart: unless-stopped
```

### 启动服务

```bash
# 首次启动（交互式认证）
docker-compose run --rm copilot-api

# 认证完成后，后台启动
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

---

## 🔐 生产环境部署

如果不想每次都交互式认证，可以使用 Token：

### 获取 Token

```bash
# 在本地获取 token
docker run -it --rm xlight/copilot-api:latest auth --show-token
```

按提示完成认证，复制显示的 token。

### 使用 Token 部署

```bash
docker run -d \
  --name copilot-api \
  -p 4141:4141 \
  -e GH_TOKEN=your_github_token_here \
  --restart unless-stopped \
  xlight/copilot-api:latest
```

或在 docker-compose.yml 中：

```yaml
version: "3.8"

services:
  copilot-api:
    image: xlight/copilot-api:latest
    container_name: copilot-api
    ports:
      - "4141:4141"
    environment:
      - GH_TOKEN=${GH_TOKEN}
    restart: unless-stopped
```

创建 `.env` 文件：
```bash
GH_TOKEN=your_github_token_here
```

启动：
```bash
docker-compose up -d
```

---

## 💡 实用示例

### Python 调用

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:4141/v1",
    api_key="dummy"
)

response = client.chat.completions.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "写一个快速排序算法"}
    ]
)

print(response.choices[0].message.content)
```

### Node.js 调用

```javascript
const OpenAI = require('openai');

const openai = new OpenAI({
  baseURL: 'http://localhost:4141/v1',
  apiKey: 'dummy'
});

async function main() {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      { role: 'user', content: '写一个快速排序算法' }
    ]
  });

  console.log(completion.choices[0].message.content);
}

main();
```

### 配置 Claude Code

在项目根目录创建 `.claude/settings.json`：

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

---

## 🛠️ 常用命令

```bash
# 查看容器状态
docker ps

# 查看日志
docker logs copilot-api

# 实时查看日志
docker logs -f copilot-api

# 进入容器
docker exec -it copilot-api sh

# 重启容器
docker restart copilot-api

# 停止容器
docker stop copilot-api

# 删除容器
docker rm copilot-api

# 查看使用情况
curl http://localhost:4141/usage

# 测试健康检查
curl http://localhost:4141/

# 列出模型
curl http://localhost:4141/v1/models
```

---

## ⚠️ 常见问题

### 端口被占用？

```bash
# 使用其他端口
docker run -d -p 8080:4141 xlight/copilot-api:latest

# 或修改 docker-compose.yml 中的端口映射
ports:
  - "8080:4141"
```

### 认证失败？

```bash
# 清除认证数据重试
rm -rf copilot-data
docker restart copilot-api
```

### 容器无法启动？

```bash
# 查看详细日志
docker logs copilot-api

# 检查是否端口冲突
lsof -i :4141  # macOS/Linux
```

---

## 📚 下一步

- 📖 阅读 [完整文档](README.md)
- 🔧 查看 [详细设置指南](SETUP.md)
- 🌐 访问 [原始项目](https://github.com/ericc-ch/copilot-api)

---

## 🎉 完成！

现在你可以：

1. ✅ 使用 OpenAI SDK 连接 Copilot
2. ✅ 在任何支持 OpenAI API 的工具中使用
3. ✅ 配置 Claude Code 使用 Copilot
4. ✅ 监控 API 使用情况

享受你的 Copilot API 服务！🚀
