# Xboard

<div align="center">

[![Telegram](https://img.shields.io/badge/Telegram-Channel-blue)](https://t.me/XboardOfficial)
![PHP](https://img.shields.io/badge/PHP-8.2+-green.svg)
![MySQL](https://img.shields.io/badge/MySQL-5.7+-blue.svg)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

## 📖 项目简介

Xboard 是一个基于 Laravel 11 构建的现代化面板系统，专注于提供简洁、高效的使用体验。

## ✨ 特性

- 🚀 基于 Laravel 12 + Octane，性能显著提升
- 🎨 全新管理后台界面（React + Shadcn UI）
- 📱 现代化用户前端（Vue3 + TypeScript）
- 🐳 开箱即用的 Docker 部署方案
- 🎯 更清晰的系统架构，维护性更好

## 🚀 快速开始

```bash
git clone -b compose --depth 1 https://github.com/cedar2025/Xboard && \
cd Xboard && \
docker compose run -it --rm \
    -e ENABLE_SQLITE=true \
    -e ENABLE_REDIS=true \
    -e ADMIN_ACCOUNT=admin@demo.com \
    web php artisan xboard:install && \
docker compose up -d
```

> 安装完成后访问：`http://服务器IP:7001`  
> ⚠️ 请务必保存安装过程中显示的管理员账号信息

## 📖 文档

### 🔄 升级说明
> 🚨 **重要：** 当前版本变更较大，升级前请严格按照升级文档操作，并提前备份数据库。请注意“升级”和“迁移”是两件事，不要混淆。

### 开发指南
- [插件开发指南](./docs/en/development/plugin-development-guide.md) - XBoard 插件开发完整说明

### 部署指南
- [1Panel 部署](./docs/en/installation/1panel.md)
- [Docker Compose 部署](./docs/en/installation/docker-compose.md)
- [aaPanel 部署](./docs/en/installation/aapanel.md)
- [aaPanel + Docker 部署](./docs/en/installation/aapanel-docker.md)（推荐）

### 迁移指南
- [从 v2board dev 迁移](./docs/en/migration/v2board-dev.md)
- [从 v2board 1.7.4 迁移](./docs/en/migration/v2board-1.7.4.md)
- [从 v2board 1.7.3 迁移](./docs/en/migration/v2board-1.7.3.md)

## 🛠️ 技术栈

- 后端：Laravel 11 + Octane
- 管理后台：React + Shadcn UI + TailwindCSS
- 用户前端：Vue3 + TypeScript + NaiveUI
- 部署：Docker + Docker Compose
- 缓存：Redis + Octane Cache

## 📷 页面预览
![后台预览](./docs/images/admin.png)

![前台预览](./docs/images/user.png)

## ⚠️ 免责声明

本项目仅用于学习与交流，使用本项目产生的任何后果由使用者自行承担。

## 🌟 维护状态说明

当前项目处于轻维护状态，我们将会：
- 修复关键 Bug 与安全问题
- 审核并合并重要 PR
- 提供必要的兼容性更新

但新功能开发节奏可能较慢。

## 🔔 重要提示

1. 修改后台路径后需要重启：
```bash
docker compose restart
```

2. aaPanel 安装方式下，请重启 Octane 守护进程

## 🤝 贡献

欢迎提交 Issue 与 Pull Request 一起完善项目。

## 📈 Star 历史

[![Stargazers over time](https://starchart.cc/cedar2025/Xboard.svg)](https://starchart.cc/cedar2025/Xboard)
