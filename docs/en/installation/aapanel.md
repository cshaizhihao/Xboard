# Xboard 在 aaPanel 环境部署指南

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
- 操作系统：Ubuntu 20.04+ / Debian 10+（⚠️ 不建议 CentOS 7）
- aaPanel 最新版
- PHP 8.2
- MySQL 5.7+
- Redis
- Nginx（任意版本）

## 快速部署

### 1. 安装 aaPanel
```bash
URL=https://www.aapanel.com/script/install_6.0_en.sh && \
if [ -f /usr/bin/curl ];then curl -ksSO "$URL" ;else wget --no-check-certificate -O install_6.0_en.sh "$URL";fi && \
bash install_6.0_en.sh aapanel
```

### 2. 基础环境配置

#### 2.1 安装 LNMP 环境
在 aaPanel 后台安装：
- Nginx（任意版本）
- MySQL 5.7
- PHP 8.2

#### 2.2 安装 PHP 扩展
必需扩展：
- redis
- fileinfo
- swoole
- readline
- event
- mbstring

#### 2.3 开启 PHP 必需函数
需要放行的函数：
- putenv
- proc_open
- pcntl_alarm
- pcntl_signal

### 3. 站点配置

#### 3.1 创建站点
1. 进入：aaPanel > 网站 > 添加站点
2. 填写：
   - 域名：你的站点域名
   - 数据库：选择 MySQL
   - PHP 版本：选择 8.2

#### 3.2 部署 Xboard
```bash
# 进入站点目录
cd /www/wwwroot/your-domain

# 清空目录
chattr -i .user.ini
rm -rf .htaccess 404.html 502.html index.html .user.ini

# 克隆仓库
git clone https://github.com/cedar2025/Xboard.git ./

# 安装依赖
sh init.sh
```

#### 3.3 配置站点
1. 运行目录设置为 `/public`
2. 添加伪静态规则：
```nginx
location /downloads {
}

location / {  
    try_files $uri $uri/ /index.php$is_args$query_string;  
}

location ~ .*\.(js|css)?$
{
    expires      1h;
    error_log off;
    access_log /dev/null; 
}
```

## 详细配置

### 1. 配置守护进程
1. 安装 Supervisor
2. 添加队列守护进程：
   - 名称：`Xboard`
   - 运行用户：`www`
   - 运行目录：站点目录
   - 启动命令：`php artisan horizon`
   - 进程数：1

### 2. 配置计划任务
- 类型：Shell 脚本
- 任务名称：v2board
- 运行用户：www
- 执行频率：每 1 分钟
- 脚本内容：`php /www/wwwroot/site-directory/artisan schedule:run`

### 3. Octane 配置（可选）
#### 3.1 添加 Octane 守护进程
- 名称：Octane
- 运行用户：www
- 运行目录：站点目录
- 启动命令：`/www/server/php/82/bin/php artisan octane:start --port 7010`
- 进程数：1

#### 3.2 Octane 专用伪静态规则
```nginx
location ~* \.(jpg|jpeg|png|gif|js|css|svg|woff2|woff|ttf|eot|wasm|json|ico)$ {
}

location ~ .* {
    proxy_pass http://127.0.0.1:7010;
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
}
```

## 维护指南

### 版本更新
```bash
# 进入站点目录
cd /www/wwwroot/your-domain

# 执行更新脚本
git fetch --all && git reset --hard origin/master && git pull origin master
sh update.sh

# 如启用了 Octane，更新后重启守护进程
# aaPanel > 软件商店 > 工具 > Supervisor > 重启 Octane
```

### 日常维护
- 定期检查日志
- 监控系统资源占用
- 定期备份数据库与配置文件

## 故障排查

### 常见问题
1. **后台空白**：若后台页面空白，执行 `git submodule update --init --recursive --force` 恢复主题文件
2. 修改后台路径后需要重启服务才会生效
3. 开启 Octane 后，任何代码变更都需要重启才生效
3. PHP 扩展安装失败时，先确认 PHP 版本是否正确
4. 数据库连接失败时，检查数据库配置与权限

## 启用 WebSocket 实时同步（可选）

WebSocket 可将配置和用户变更实时同步到节点。

### 1. 启动 WS Server

在 aaPanel Supervisor 中新增 WebSocket 守护进程：
- 名称：`Xboard-WS`
- 运行用户：`www`
- 运行目录：站点目录
- 启动命令：`php artisan ws-server start`
- 进程数：1

### 2. 配置 Nginx

在站点 Nginx 配置中，把以下 WebSocket 配置放在主 `location ^~ /` 前面：
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
```

### 3. 重启服务

在 Supervisor 中重启 Octane 与 WS Server 进程。

> 节点侧会在握手时自动探测 WebSocket 可用性，无需额外配置。
