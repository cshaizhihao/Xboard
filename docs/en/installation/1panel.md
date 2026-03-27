# 1Panel 快速部署指南

本文说明如何使用 1Panel 部署 Xboard。

## 1. 环境准备

安装 1Panel：
```bash
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && \
sudo bash quick_start.sh
```

## 2. 环境配置

1. 在应用商店安装：
   - OpenResty（任意版本）
     - ⚠️ 勾选“外部端口访问”，确保防火墙放行
   - MySQL 5.7（ARM 架构建议使用 MariaDB）

2. 创建数据库：
   - 数据库名：`xboard`
   - 用户名：`xboard`
   - 访问权限：所有主机（%）
   - 请保存数据库密码，安装时会用到

## 3. 部署步骤

1. 新建网站：
   - 进入“网站” > “创建网站” > “反向代理”
   - 域名：填写你的域名
   - 代号：`xboard`
   - 代理地址：`127.0.0.1:7001`

2. 配置反向代理：
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

3. 安装 Xboard：
```bash
# 进入站点目录
cd /opt/1panel/apps/openresty/openresty/www/sites/xboard/index

# 安装 Git（未安装时）
## Ubuntu/Debian
apt update && apt install -y git
## CentOS/RHEL
yum update && yum install -y git

# 克隆仓库
git clone -b compose --depth 1 https://github.com/cedar2025/Xboard ./

# 准备 Docker Compose 配置
```

4. 编辑 compose.yaml：
```yaml
services:
  web:
    image: ghcr.io/cedar2025/xboard:new
    volumes:
      - redis-data:/data
      - ./.env:/www/.env
      - ./.docker/.data/:/www/.docker/.data
      - ./storage/logs:/www/storage/logs
      - ./storage/theme:/www/storage/theme
      - ./plugins:/www/plugins
    environment:
      - docker=true
    depends_on:
      - redis
    command: php artisan octane:start --host=0.0.0.0 --port=7001
    restart: on-failure
    ports:
      - 7001:7001
    networks:
      - 1panel-network

  horizon:
    image: ghcr.io/cedar2025/xboard:new
    volumes:
      - redis-data:/data
      - ./.env:/www/.env
      - ./.docker/.data/:/www/.docker/.data
      - ./storage/logs:/www/storage/logs
      - ./plugins:/www/plugins
    restart: on-failure
    command: php artisan horizon
    networks:
      - 1panel-network
    depends_on:
      - redis
  ws-server:
    image: ghcr.io/cedar2025/xboard:new
    volumes:
      - redis-data:/data
      - ./.env:/www/.env
      - ./.docker/.data/:/www/.docker/.data
      - ./storage/logs:/www/storage/logs
      - ./plugins:/www/plugins
    restart: on-failure
    ports:
      - 8076:8076
    networks:
      - 1panel-network
    command: php artisan ws-server start
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    command: redis-server --unixsocket /data/redis.sock --unixsocketperm 777
    restart: unless-stopped
    networks:
      - 1panel-network
    volumes:
      - redis-data:/data

volumes:
  redis-data:

networks:
  1panel-network:
    external: true
```

5. 初始化安装：
```bash
# 安装依赖并初始化
docker compose run -it --rm web php artisan xboard:install
```

⚠️ 重要配置说明：
1. 数据库配置
   - 数据库主机（Host）：根据部署情况选择
     1. 如果数据库与 Xboard 在同一网络，填 `mysql`
     2. 如果连接失败，去：数据库 -> 目标数据库 -> 连接信息 -> 容器连接，使用其中“Host”值
     3. 如果使用外部数据库，填写真实数据库地址
   - 数据库端口：`3306`（除非你改过）
   - 数据库名：`xboard`
   - 数据库用户：`xboard`
   - 数据库密码：填写前面保存的密码

2. Redis 配置
   - 选择使用内置 Redis
   - 无需额外配置

3. 管理员信息
   - 保存安装完成后显示的管理员账号密码
   - 记录后台访问地址

配置完成后启动服务：
```bash
docker compose up -d
```

6. 启动服务：
```bash
docker compose up -d
```

## 4. 版本更新

> 💡 重要：更新命令取决于你安装的版本形态：
> - 若为较新安装（新版本），使用：
```bash
docker compose pull && \
docker compose run -it --rm web php artisan xboard:update && \
docker compose up -d
```
> - 若为较早安装（旧版本），把 `web` 改成 `xboard`：
```bash
docker compose pull && \
docker compose run -it --rm xboard php artisan xboard:update && \
docker compose up -d
```
> 🤔 不确定用哪个？先试新版本命令，失败再用旧版命令。

## 注意事项

- ⚠️ 建议启用防火墙，避免 7001 端口直接暴露公网
- 修改代码后需要重启服务
- 建议配置 SSL 证书保障访问安全

> 节点侧会在握手时自动探测 WebSocket 可用性，无需额外配置。
