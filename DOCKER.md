# Docker 部署指南

## 快速开始

### 使用预构建镜像（推荐）

从 GitHub Container Registry 拉取已构建的镜像：

```bash
# 拉取最新镜像
docker pull ghcr.io/jnuse/resume:latest

# 使用 docker-compose 启动
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止
docker-compose down
```

访问 http://localhost:3000

### 本地构建

如果需要修改代码并本地构建：

```bash
# 使用开发配置构建
docker-compose -f docker-compose.dev.yml up -d --build

# 或手动构建
docker build -t resume:local .
docker run -p 3000:3000 resume:local
```

## 环境变量

在 `docker-compose.yml` 中配置环境变量：

```yaml
environment:
  # 启用站点密码保护（可选）
  - SITE_PASSWORD=your_secure_password_here
```

## 配置说明

### docker-compose.yml
- **用途**: 生产环境，使用预构建镜像
- **镜像**: `ghcr.io/jnuse/resume:latest`
- **端口**: 3000

### docker-compose.dev.yml
- **用途**: 开发/测试环境，本地构建
- **构建**: 从源代码构建
- **端口**: 3000

## 镜像构建

GitHub Actions 会在以下情况自动构建并推送镜像：

1. **推送到 main 分支**
   - 构建 `ghcr.io/jnuse/resume:latest`
   - 构建 `ghcr.io/jnuse/resume:main`

2. **创建版本标签**（如 v1.0.0）
   - 构建 `ghcr.io/jnuse/resume:1.0.0`
   - 构建 `ghcr.io/jnuse/resume:1.0`
   - 构建 `ghcr.io/jnuse/resume:1`

3. **支持架构**
   - linux/amd64
   - linux/arm64

## 健康检查

容器启动后会自动进行健康检查：

```bash
# 手动检查
curl http://localhost:3000/api/pdf/health
```

## 资源限制

如需限制容器资源使用，取消 `docker-compose.yml` 中的注释：

```yaml
deploy:
  resources:
    limits:
      cpus: '1.0'
      memory: 1G
    reservations:
      cpus: '0.5'
      memory: 512M
```

## 镜像大小

优化后的镜像大小约 **200MB**：
- 基于 Alpine Linux
- 使用 Next.js standalone 输出
- 系统 Chromium 替代打包版本

## 故障排查

### 容器无法启动

```bash
# 查看容器日志
docker-compose logs resume

# 查看容器状态
docker-compose ps
```

### PDF 生成失败

确保 Chromium 已正确安装：

```bash
docker-compose exec resume chromium-browser --version
```

### 权限问题

容器以非 root 用户（nextjs:nodejs, UID 1001）运行，确保卷挂载权限正确。

## GitHub Actions 配置

workflow 文件位于 `.github/workflows/docker-build.yml`

**重要**: 需要在 GitHub 仓库设置中启用必要的权限：

1. 进入 **Settings** → **Actions** → **General**
2. 找到 **Workflow permissions**
3. 选择 **Read and write permissions**
4. 保存

这样 workflow 才能推送镜像到 GitHub Container Registry。
