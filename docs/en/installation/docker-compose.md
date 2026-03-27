# Docker Compose 快速部署指南

本文说明如何使用 Docker Compose 快速部署 Xboard。默认使用 SQLite 数据库，无需单独安装 MySQL。

### 1. 环境准备

安装 Docker：
```bash
curl -sSL https://get.docker.com | bash

# CentOS 额外执行：
systemctl enable docker
systemctl start docker
```

### 2. 部署步骤

1. 获取项目文件：
```bash
git clone -b compose --depth 1 https://github.com/cedar2025/Xboard
cd Xboard
```

2. 安装并初始化数据库：

- 快速安装（推荐新手）
```bash
docker compose run -it --rm \
    -e ENABLE_SQLITE=true \
    -e ENABLE_REDIS=true \
    -e ADMIN_ACCOUNT=admin@demo.com \
    web php artisan xboard:install
```
- 自定义配置安装（高级用户）
```bash
docker compose run -it --rm web php artisan xboard:install
```
> 安装完成后请保存后台地址、用户名和密码

3. 启动服务：
```bash
docker compose up -d
```

4. 访问站点：
- 默认端口：7001
- 访问地址：`http://你的服务器IP:7001`

### 3. 版本更新

> 💡 重要：更新命令取决于你的安装版本：
> - 较新安装（新版本）使用：
```bash
cd Xboard
docker compose pull && \
docker compose run -it --rm web php artisan xboard:update && \
docker compose up -d
```
> - 较早安装（旧版本）将 `web` 改为 `xboard`：
```bash
cd Xboard
docker compose pull && \
docker compose run -it --rm xboard php artisan xboard:update && \
docker compose up -d
```
> 🤔 不确定用哪个？先试新版本命令，失败再用旧版命令。

### 4. 版本回滚

1. 将 `docker-compose.yaml` 中镜像版本改为需要回滚的版本
2. 执行：`docker compose up -d`

### 注意事项

- 如需 MySQL，请自行安装并按需重新部署
- 代码修改后需重启服务才能生效
- 建议通过 Nginx 反向代理到 80 端口访问
