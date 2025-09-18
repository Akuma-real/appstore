# 1Panel 应用商店开发指南

## 应用目录结构

推荐的应用目录结构如下：

```
apps/halo/                    // 应用根目录
├── logo.png                 // 应用 logo，最好是 180 * 180 px，不要超过 10 KB
├── data.yml                 // 应用声明文件
├── README.md                // 应用的中文 README
├── README_en.md             // 应用的英文 README
├── 2.2.0                    // 应用版本，注意不要以 v 开头
│   ├── data.yml             // 应用的参数配置
│   ├── data/                // 挂载出来的目录
│   ├── scripts/             // 脚本目录 存放 init.sh、upgrade.sh、uninstall.sh
│   └── docker-compose.yml   // docker-compose 文件
└── 2.3.2
    ├── data.yml
    ├── data/
    ├── scripts/
    └── docker-compose.yml
```

## 应用分类字典（原 data.yaml 摘要）

以下标签键值用于 `additionalProperties.tags`，需保持与 1Panel 官方分类一致（当前版本 v1.6.0）：

- `Website`：建站 / Website
- `Database`：数据库 / Database
- `Server`：Web 服务器 / Web Server
- `Runtime`：运行环境 / Runtime
- `Tool`：实用工具 / Tool
- `Storage`：云存储 / Cloud Storage
- `AI`：AI / AI
- `BI`：BI / BI
- `CRM`：CRM / CRM
- `Security`：安全 / Security
- `DevTool`：开发工具 / Development Tool
- `DevOps`：DevOps / DevOps
- `Middleware`：中间件 / Middleware
- `Media`：多媒体 / Media
- `Email`：邮件服务 / Email Service
- `Game`：休闲游戏 / Casual Games
- `Local`：本地 / Local

## 应用声明文件 data.yml 配置说明

### 官方应用商店兼容字段

```yaml
# === 官方应用商店兼容字段（上架官方商店必填，其余情况选填） ===
# https://apps.fit2cloud.com/1panel
name: Halo                    # 显示名称
tags: 建站                    # 分类标签
title: 强大易用的开源建站工具    # 副标题
description: 强大易用的开源建站工具  # 应用描述
```

### 1Panel 应用商店数据结构

```yaml
# === 以下为 1Panel 应用商店解析的数据结构，必须保留此结构 ===
additionalProperties:
  # === 必填字段 ===
  key: halo                   # 应用程序唯一标识符（必填）
  name: Halo                  # 应用程序显示名称（必填）
  type: website               # 应用程序类型（必填）
                             # 可选值: website(网站应用)、runtime(运行时环境)、tool(工具类应用)
  tags:                      # 分类标签数组（必填）
    - Website

  # === 描述信息字段（选填，建议填写）===
  shortDescZh: 强大易用的开源建站工具      # 中文描述（v1版本格式）
  shortDescEn: Powerful and easy-to-use open source website builder  # 英文描述（v1版本格式）

  # 多语言描述（v2版本格式）
  description:
    zh: 强大易用的开源建站工具
    zh-hant: 強大易用的開源建站工具
    en: A powerful and easy-to-use open source website building tool
    ja: 強力で使いやすいオープンソースのウェブサイト構築ツール
    # 更多语言支持...

  # === 功能配置字段（选填）===
  crossVersionUpdate: true    # 是否支持跨版本更新，默认false
  limit: 0                   # 安装数量限制，0表示无限制，默认0
  recommend: 5               # 推荐级别，数值越小越推荐，默认9999

  # === 链接信息字段（选填）===
  website: https://halo.run/              # 官方网站链接
  github: https://github.com/halo-dev/halo  # GitHub 仓库链接
  document: https://docs.halo.run/        # 文档链接

  # === 系统要求字段（选填）===
  memoryRequired: 1024       # 内存要求（MB）
  architectures:             # 支持的 CPU 架构
    - amd64
    - arm64
    - ppc64le
    - s390x
  gpuSupport: false         # 是否需要GPU支持，默认false

  # === 版本兼容性字段（选填）===
  version: 0                # 最低面板版本要求，0表示无要求
  deprecated: 0             # 废弃版本标记，0表示未废弃
```

## 版本目录下的 data.yml 配置说明

### 基本配置结构

```yaml
formFields:
  # 1. 应用端口配置
  - type: "number"
    labelZh: "端口"
    labelEn: "Port"
    label:
      zh: "端口"
      zh-hant: "連接埠"
      en: "Port"
      # 更多语言...
    description:
      zh: "设置应用程序的 HTTP 访问端口，有效范围：1-65535"
      # 更多语言...
    required: true
    envKey: "PANEL_APP_PORT_HTTP"  # 环境变量键名（必填）
    default: 8090                  # 默认值
    rule: "ports"                  # 验证规则

  # 2. 应用名称配置
  - type: "text"
    labelZh: "应用名称"
    labelEn: "App Name"
    label:
      zh: "应用名称"
      # 更多语言...
    required: true
    envKey: "APP_NAME"
    default: "demo"
    edit: false                    # 是否允许用户修改
    random: false                  # 是否在默认值后追加随机字符串

  # 3. 数据库服务配置（服务依赖）
  - type: "apps"
    labelZh: "数据库服务"
    labelEn: "Database Service"
    label:
      zh: "数据库服务"
      # 更多语言...
    description:
      zh: "选择应用依赖的数据库类型及对应服务实例"
      # 更多语言...
    required: true
    envKey: "PANEL_DB_TYPE"        # 数据库类型环境变量
    default: "postgresql"          # 默认数据库类型

  # 4. 数据库名称配置
  - type: "text"
    labelZh: "数据库名"
    labelEn: "Database Name"
    description:
      zh: "应用所需的数据库名称，将自动创建"
      # 更多语言...
    required: true
    envKey: "PANEL_DB_NAME"
    default: ""
    edit: false
    random: true
    rule: "paramCommon"

  # 5. 数据库密码配置（敏感信息）
  - type: "password"
    labelZh: "数据库密码"
    labelEn: "Database Password"
    description:
      zh: "数据库用户的访问密码，自动生成高强度随机密码"
      # 更多语言...
    required: true
    envKey: "PANEL_DB_USER_PASSWORD"
    default: ""
    edit: false
    random: true                   # 强制生成随机密码

  # 6. Redis 缓存服务配置
  - type: "service"
    key: "redis"                   # 依赖服务标识
    labelZh: "Redis 服务"
    labelEn: "Redis Service"
    description:
      zh: "选择应用依赖的 Redis 服务实例"
      # 更多语言...
    required: true
    envKey: "PANEL_REDIS_HOST"
```

## Docker Compose 配置规范

### 基本结构要求

```yaml
services:
  halo:
    image: halohub/halo:2.3.2      # 使用官方或长期维护的镜像
    container_name: ${CONTAINER_NAME}
    restart: unless-stopped         # 重启策略
    networks:
      - 1panel-network             # 必须保留此网络配置
    labels:
      createdBy: "Apps"            # 必须保留此标签（固定写法）
    environment:
      # 环境变量配置（对应 data.yml 中的 envKey）
      - SPRING_R2DBC_URL=r2dbc:pool:${PANEL_DB_TYPE}://${PANEL_DB_HOST}:${PANEL_DB_PORT}/${PANEL_DB_NAME}
      - SPRING_R2DBC_USERNAME=${PANEL_DB_USER}
      - SPRING_R2DBC_PASSWORD=${PANEL_DB_USER_PASSWORD}
    volumes:
      # 数据持久化配置（必须挂载核心数据目录）
      - ./data:/root/.halo2
    ports:
      # 端口映射（使用 1Panel 配置的端口变量）
      - ${PANEL_APP_PORT_HTTP}:8090
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8090/actuator/health/readiness"]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 30s

networks:
  1panel-network:                  # 必须保留此网络配置
    external: true
```

### 重要配置说明

1. **环境变量映射**：环境变量名必须与 data.yml 中 formFields 的 envKey 完全一致
2. **数据持久化**：核心数据目录必须通过 volumes 挂载到宿主机
3. **网络配置**：必须使用 `1panel-network` 网络
4. **标签配置**：必须保留 `labels: createdBy: "Apps"` 标签
5. **端口配置**：优先使用 `PANEL_APP_PORT_HTTP` 变量

## README 编写要求

### 基本结构规范

README.md 需包含以下二级标题：

- **产品介绍**：简要介绍应用的主要功能和特点
- **默认账户信息**：提供默认的登录凭据（如有）
- **主要功能**：列出应用的核心功能特性
- **注意事项**：重要的使用注意事项和限制
- **配置步骤**：详细的配置和使用步骤

### 文档规范要求

- 命令示例使用带语言标注的代码块
- 重要警告使用提示块或粗体突出
- 英文版 README_en.md 需同步核心信息

## 开发测试指南

### 验证命令

```bash
# YAML 语法校验
yq eval '.' apps/halo/data.yml
yq eval '.' apps/halo/2.3.2/data.yml

# Docker Compose 配置检查
docker compose -f apps/halo/2.3.2/docker-compose.yml config

# 查看改动统计
git diff --stat
```

### 测试流程

1. **语法验证**：使用 `yq` 校验 YAML 语法
2. **配置检查**：使用 `docker compose config` 检查配置完整性
3. **功能测试**：使用 `docker compose up -d` 进行快速冒烟测试
4. **清理环境**：测试完成后使用 `docker compose down` 清理
5. **真实安装测试**：上传到 `/opt/1panel/resource/apps/local` 并在 1Panel 中导入测试

## 代码风格规范

### YAML 格式要求

- 使用两个空格缩进
- 键名使用小写短横线格式
- 布尔值写成 `true/false`
- 镜像标签、端口与路径保持与上游一致

### 脚本编写规范

- 采用可执行权限的 POSIX Shell
- 首行使用 `#!/bin/bash`
- 在 README 中说明脚本用途

## 提交与 PR 指南

### 提交规范

遵循约定式提交格式：
- `feat`: 新功能
- `fix`: 修复问题
- `chore`: 杂务（如依赖更新）
- `docs`: 文档更新

### PR 要求

PR 描述应包含：
- 改动背景说明
- 涉及的应用列表
- 验证方式（含截图或命令输出）
- 使用 `Fixes #123` 关联相关 Issue
- 涉及安全或兼容性调整需补充风险与回滚建议

## 安全注意事项

1. **镜像安全**：仅使用官方或长期维护的镜像
2. **凭证安全**：环境变量不得硬编码凭证
3. **默认账号**：在 README 中明示默认账号的修改步骤
4. **License 检查**：提交前核查相关 License
5. **平台标识**：必须保留 `networks.1panel-network` 和 `labels.createdBy: "Apps"` 配置

## 本地测试方法

将开发完成的应用上传到目标服务器的 `/opt/1panel/resource/apps/local` 目录，然后在 1Panel 中选择"本地应用"分类即可看到该应用，点击安装进行测试。