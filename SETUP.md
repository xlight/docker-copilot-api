# 详细设置指南

本指南将帮助您从零开始设置和部署 Docker Copilot API 项目。

## 📋 目录

- [准备工作](#准备工作)
- [GitHub Actions 设置](#github-actions-设置)
- [本地开发设置](#本地开发设置)
- [生产环境部署](#生产环境部署)
- [常见问题](#常见问题)

---

## 准备工作

### 1. 必需的账户和工具

- **GitHub 账户**：需要有 Copilot 订阅（个人版、商业版或企业版）
- **Docker Hub 账户**：用于存储构建的镜像
- **Docker**：本地开发需要（可选）
- **Git**：用于克隆和管理代码

### 2. 验证 GitHub Copilot 订阅

访问 https://github.com/settings/copilot 确认您的订阅状态。

---

## GitHub Actions 设置

### 步骤 1：Fork 或创建仓库

1. 在 GitHub 上创建新仓库 `docker-copilot-api`
2. 将本地代码推送到 GitHub：

```bash
cd docker-copilot-api
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/xlight/docker-copilot-api.git
git push -u origin main
```

### 步骤 2：获取 Docker Hub 访问令牌

1. 登录 [Docker Hub](https://hub.docker.com/)
2. 点击右上角的用户名 → Account Settings
3. 选择 **Security** → **New Access Token**
4. 创建令牌（建议权限：Read, Write, Delete）
5. **立即复制令牌**（只会显示一次）

### 步骤 3：配置 GitHub Secrets

1. 打开您的 GitHub 仓库
2. 进入 **Settings** → **Secrets and variables** → **Actions**
3. 点击 **New repository secret** 添加以下 secrets：

| Secret 名称 | 值 | 说明 |
|------------|-----|------|
| `DOCKERHUB_USERNAME` | 您的 Docker Hub 用户名 | 例如：`johndoe` |
| `DOCKERHUB_TOKEN` | 刚才创建的访问令牌 | 以 `dckr_pat_` 开头 |

### 步骤 4：更新配置文件

在 `README.md` 和 `docker-compose.yml` 中，将所有 `xlight` 替换为您的实际 Docker Hub 用户名。

**批量替换命令（macOS/Linux）：**

```bash
# 替换 README.md
sed -i 's/xlight/your-dockerhub-username/g' README.md

# 替换 docker-compose.yml
sed -i 's/xlight/your-dockerhub-username/g' docker-compose.yml
```

**Windows PowerShell：**

```powershell
(Get-Content README.md) -replace 'xlight', 'your-dockerhub-username' | Set-Content README.md
(Get-Content docker-compose.yml) -replace 'xlight', 'your-dockerhub-username' | Set-Content docker-compose.yml
```

### 步骤 5：触发构建

推送更改到 GitHub 将自动触发构建：

```bash
git add .
git commit -m "Update Docker Hub username"
git push origin main
```

查看构建状态：
- 访问仓库的 **Actions** 标签页
- 点击最新的 workflow 运行查看详情

### 步骤 6：验证镜像

构建成功后，访问 Docker Hub 确认镜像已推送：
```
https://hub.docker.com/r/xlight/copilot-api
```

---

## 本地开发设置

### 方法 1：使用 Docker Hub 镜像（推荐）

等待 GitHub Actions 构建完成后：

```bash
# 创建数据目录
mkdir -p ./copilot-data

# 拉取并运行镜像
docker pull xlight/copilot-api:latest
docker run -it -p 4141:4141 -v $(pwd)/copilot-data:/root/.local/share/copilot-api xlight/copilot-api:latest
```

### 方法 2：本地构建镜像

如果您需要自定义构建：

```bash
# 克隆 copilot-api 源代码
git clone https://github.com/ericc-ch/copilot-api.git

# 进入目录
cd copilot-api

# 构建镜像
docker build -t copilot-api:local .

# 运行容器
cd ..
mkdir -p ./copilot-data
docker run -it -p 4141:4141 -v $(pwd)/copilot-data:/root/.local/share/copilot-api copilot-api:local
```

### 方法 3：使用 Docker Compose

```bash
# 确保 docker-compose.yml 中的用户名已更新
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

### 首次认证流程

容器启动后，会提示您进行 GitHub 认证：

1. 访问显示的 URL（通常是 `https://github.com/login/device`）
2. 输入显示的设备代码
3. 授权应用访问您的 GitHub 账户
4. 返回终端，等待认证完成

认证信息将保存在 `copilot-data` 目录中，下次启动会自动使用。

### 测试 API

```bash
# 测试健康检查
curl http://localhost:4141/

# 列出可用模型
curl http://localhost:4141/v1/models

# 发送聊天请求
curl -X POST http://localhost:4141/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy" \
  -d '{
    "model": "gpt-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'

# 查看使用情况
curl http://localhost:4141/usage
```

---

## 生产环境部署

### 选项 1：使用环境变量（推荐）

适合自动化部署，无需交互式认证。

#### 步骤 1：获取 GitHub Token

在本地运行以获取 token：

```bash
docker run -it --rm xlight/copilot-api:latest auth --show-token
```

按提示完成认证，然后复制显示的 token。

#### 步骤 2：使用 Token 部署

```bash
docker run -d \
  --name copilot-api \
  -p 4141:4141 \
  -e GH_TOKEN=your_github_token_here \
  --restart unless-stopped \
  xlight/copilot-api:latest
```

或使用 Docker Compose：

```yaml
# docker-compose.prod.yml
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
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:4141/"]
      interval: 30s
      timeout: 5s
      retries: 3
```

创建 `.env` 文件：

```bash
echo "GH_TOKEN=your_github_token_here" > .env
```

启动服务：

```bash
docker-compose -f docker-compose.prod.yml up -d
```

### 选项 2：使用数据卷持久化

适合有持久化存储的服务器环境。

```bash
# 首次交互式认证
docker run -it \
  -p 4141:4141 \
  -v /data/copilot-api:/root/.local/share/copilot-api \
  xlight/copilot-api:latest

# 后续自动启动
docker run -d \
  --name copilot-api \
  -p 4141:4141 \
  -v /data/copilot-api:/root/.local/share/copilot-api \
  --restart unless-stopped \
  xlight/copilot-api:latest
```

### 选项 3：Kubernetes 部署

创建 `k8s-deployment.yaml`：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: copilot-api-secret
type: Opaque
stringData:
  gh-token: your_github_token_here
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: copilot-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: copilot-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: copilot-api
  template:
    metadata:
      labels:
        app: copilot-api
    spec:
      containers:
      - name: copilot-api
        image: xlight/copilot-api:latest
        ports:
        - containerPort: 4141
        env:
        - name: GH_TOKEN
          valueFrom:
            secretKeyRef:
              name: copilot-api-secret
              key: gh-token
        volumeMounts:
        - name: data
          mountPath: /root/.local/share/copilot-api
        livenessProbe:
          httpGet:
            path: /v1/models
            port: 4141
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /v1/models
            port: 4141
          initialDelaySeconds: 10
          periodSeconds: 10
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: copilot-data-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: copilot-api
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 4141
  selector:
    app: copilot-api
```

部署：

```bash
kubectl apply -f k8s-deployment.yaml
```

### 生产环境最佳实践

1. **反向代理**：使用 Nginx 或 Traefik 添加 HTTPS
2. **速率限制**：启动时添加 `--rate-limit 30 --wait` 参数
3. **监控**：定期检查 `/usage` 端点
4. **日志**：配置日志聚合系统
5. **备份**：定期备份 `copilot-data` 目录
6. **更新**：定期拉取最新镜像

```bash
# 更新到最新版本
docker pull xlight/copilot-api:latest
docker-compose up -d
```

---

## 常见问题

### 1. 如何查看容器日志？

```bash
# Docker
docker logs copilot-api

# Docker Compose
docker-compose logs -f copilot-api

# 查看最近 100 行
docker logs --tail 100 copilot-api
```

### 2. 认证失败怎么办？

```bash
# 清除认证数据
rm -rf ./copilot-data

# 重新启动容器进行认证
docker restart copilot-api
docker logs -f copilot-api
```

### 3. 端口被占用怎么办？

```bash
# 查看端口占用
lsof -i :4141  # macOS/Linux
netstat -ano | findstr :4141  # Windows

# 使用不同端口
docker run -d -p 8080:4141 xlight/copilot-api:latest
```

### 4. 如何更新镜像？

```bash
# 拉取最新镜像
docker pull xlight/copilot-api:latest

# 停止并删除旧容器
docker stop copilot-api
docker rm copilot-api

# 启动新容器
docker run -d --name copilot-api -p 4141:4141 -v $(pwd)/copilot-data:/root/.local/share/copilot-api xlight/copilot-api:latest
```

### 5. GitHub Actions 构建失败？

检查以下几点：
- Docker Hub secrets 是否正确设置
- 用户名和 token 是否有效
- GitHub Actions 是否有足够的权限
- 查看 Actions 日志获取详细错误信息

### 6. 如何启用详细日志？

```bash
docker run -d -p 4141:4141 xlight/copilot-api:latest start --verbose
```

### 7. 商业版或企业版账户如何配置？

```bash
docker run -d -p 4141:4141 xlight/copilot-api:latest start --account-type business
# 或
docker run -d -p 4141:4141 xlight/copilot-api:latest start --account-type enterprise
```

### 8. 如何限制请求速率？

```bash
# 设置 30 秒的请求间隔
docker run -d -p 4141:4141 xlight/copilot-api:latest start --rate-limit 30 --wait
```

### 9. Token 过期怎么办？

Token 会自动刷新。如果遇到问题：

```bash
# 删除旧的认证数据
rm -rf ./copilot-data

# 重新认证
docker run -it -p 4141:4141 -v $(pwd)/copilot-data:/root/.local/share/copilot-api xlight/copilot-api:latest
```

### 10. 如何配置反向代理（Nginx）？

创建 `/etc/nginx/sites-available/copilot-api`：

```nginx
server {
    listen 80;
    server_name copilot-api.yourdomain.com;

    location / {
        proxy_pass http://localhost:4141;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

启用配置：

```bash
sudo ln -s /etc/nginx/sites-available/copilot-api /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

添加 HTTPS（使用 Let's Encrypt）：

```bash
sudo certbot --nginx -d copilot-api.yourdomain.com
```

---

## 下一步

- 阅读 [README.md](README.md) 了解更多功能
- 查看 [原始项目文档](https://github.com/ericc-ch/copilot-api) 获取详细 API 说明
- 访问 [使用监控面板](https://ericc-ch.github.io/copilot-api) 查看配额

---

## 需要帮助？

- 提交 Issue：https://github.com/xlight/docker-copilot-api/issues
- 原项目 Issue：https://github.com/ericc-ch/copilot-api/issues
