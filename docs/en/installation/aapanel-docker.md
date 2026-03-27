# Xboard 在 aaPanel + Docker 环境部署指南

## 目录
1. [环境要求](#环境要求)
2. [快速部署](#快速部署)
3. [详细配置](#详细配置)
4. [维护指南](#维护指南)
5. [故障排查](#故障排查)

## 环境要求

### 硬件要求
- CPU：1 核及以上
- 内存：2GB 及以上
- 磁盘：可用空间 10GB+

### 软件要求
- 操作系统：Ubuntu 20.04+ / CentOS 7+ / Debian 10+
- aaPanel 最新版
- Docker 与 Docker Compose
- Nginx（任意版本）
- MySQL 5.7+

## 快速部署

### 1. 安装 aaPanel
```bash
curl -sSL https://www.aapanel.com/script/install_6.0_en.sh -o install_6.0_en.sh && \
bash install_6.0_en.sh aapanel
```

### 2. 基础环境配置

#### 2.1 安装 Docker
```bash
# 安装 Docker
curl -sSL https://get.docker.com | bash

# CentOS 额外执行：
systemctl enable docker
systemctl start docker
```

#### 2.2 安装必要组件
在 aaPanel 后台安装：
- Nginx（任意版本）
- MySQL 5.7
- ⚠️ 不需要安装 PHP 和 Redis

### 3. 站点配置

#### 3.1 创建站点
1. 进入：aaPanel > 网站 > 添加站点
2. 填写：
   - 域名：你的站点域名
   - 数据库：选择 MySQL
   - PHP 版本：选择纯静态

#### 3.2 部署 Xboard
```bash
# 进入站点目录
cd /www/wwwroot/your-domain

# 清空目录
chattr -i .user.ini
rm -rf .htaccess 404.html 502.html index.html .user.ini

# 克隆仓库
git clone https://github.com/cedar2025/Xboard.git ./

# 准备配置文件
cp compose.sample.yaml compose.yaml

# 安装依赖并初始化
docker compose run -it --rm web sh init.sh
```
> ⚠️ 安装完成后请保存后台地址、用户名和密码

#### 3.3 启动服务
```bash
docker compose up -d
```

#### 3.4 配置反向代理
将以下内容添加到站点配置：
```nginx
location /ws/ {
    proxy_pass http://127.0.0.1:8076;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_read_timeout 60s;
}

location ^~ / {
    proxy_pass http://127.0.0.1:7001;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Real-PORT $remote_port;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header Scheme $scheme;
    proxy_set_header Server-Protocol $server_protocol;
    proxy_set_header Server-Name $server_name;
    proxy_set_header Server-Addr $server_addr;
    proxy_set_header Server-Port $server_port;
    proxy_cache off;
}
```
> `/ws/` 用于通过 `ws-server` 实现节点实时同步。该服务默认开启，也可在「后台 > 系统设置 > 服务器」中开关。

## 维护指南

### 版本更新

> 💡 重要：更新命令取决于你的安装版本：
> - 较新安装（新版本）使用：
```bash
docker compose pull && \
docker compose run -it --rm web sh update.sh && \
docker compose up -d
```
> - 较早安装（旧版本）将 `web` 改为 `xboard`：
```bash
git config --global --add safe.directory $(pwd)
git fetch --all && git reset --hard origin/master && git pull origin master
docker compose pull && \
docker compose run -it --rm xboard sh update.sh && \
docker compose up -d
```
> 🤔 不确定用哪个？先试新版本命令，失败再用旧版命令。

### 日常维护
- 定期查看日志：`docker compose logs`
- 监控系统资源占用
- 定期备份数据库和配置文件

## 故障排查

如果安装或运行异常，请重点检查：
1. **后台空白**：若后台页面空白，执行 `git submodule update --init --recursive --force` 恢复主题文件
2. 系统环境是否满足要求
3. 所需端口是否可用
3. Docker 服务是否正常运行
4. Nginx 配置是否正确
5. 查看日志定位详细错误

> 节点侧会在握手时自动探测 WebSocket 可用性，无需额外配置。
