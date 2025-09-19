# 1Panel 应用商店应用开发指南（修正版，AI 友好）

本指南系统梳理 1Panel 应用商店应用的开发、打包、校验与上架流程，修复官方文档长期未更新的问题；亦适合作为提示词素材喂给 AI 协作。

适用读者：应用开发者、DevOps、私有化交付工程师。

兼容性：面向 1Panel v1/v2 应用商店结构。

建议阅读路径：准备工作 → 目录结构 → 应用声明 data.yml → 版本表单 data.yml → docker-compose → 脚本 → 数据卷。

注意：文中所有变量名需与版本目录下 data.yml 的 formFields.envKey 一致。

## 准备工作

- 确认满足上架前提（账号、合规、版权、镜像可访问）。
- 准备应用基础信息：名称、简述、描述、图标（180x180，≤10KB PNG）。
- 在本地或 1Panel 环境完成可运行性验证。
- 统一术语：根目录“应用声明 data.yml”，版本目录“安装表单 data.yml”。

## 目录结构（示例）

```text
├── halo
│   ├── logo.png
│   ├── data.yml
│   ├── README.md
│   ├── 2.2.0
│   │   ├── data.yml
│   │   ├── data
```│   │   ├── scripts
│   │   └── docker-compose.yml
│   └── 2.3.2
│       ├── data.yml
│       ├── data
│       └── docker-compose.yml
```

要点：
- 版本目录名不以 v 开头（如 2.3.2）。
- 空 data 目录可放置 .gitkeep 以便纳入版本控制。
- 多版本并存，每个版本目录独立可部署。

## 应用声明文件 data.yml（根目录）

该文件用于应用在商店中的基础元信息展示与分类；同时必须保留 additionalProperties 结构以兼容 1Panel 解析。

最小可用示例：

```yaml
# === 官方应用商店兼容字段（上架官方商店必填）===
name: Halo
tags: 建站
title: 强大易用的开源建站工具
description: 强大易用的开源建站工具

# === 1Panel 解析结构（必须保留）===
additionalProperties:
  key: halo              # 英文唯一键，仅用于目录与标识
  name: Halo             # 展示名
  type: website          # website/runtime/tool
  tags:
    - Website
  description:
    zh: 强大易用的开源建站工具
  crossVersionUpdate: true
  limit: 0
  recommend: 5
  website: https://halo.run/
  github: https://github.com/halo-dev/halo
  document: https://docs.halo.run/
```yaml
additionalProperties:
  memoryRequired: 1024
  architectures: [amd64, arm64, ppc64le, s390x]
  gpuSupport: false
  version: 0
  deprecated: 0
```

校验清单：
- key 为英文、短横线、下划线与数字组合（建议 2–30 位）。
- type 合法：website/runtime/tool。
- tags 与官方分类兼容；必要时同时提供中文 tags 供展示。
- 描述建议仅提供简体中文 zh。
- 链接均为有效 HTTP/HTTPS。
- architectures 与镜像实际支持保持一致。
- 若应用已废弃，设置 deprecated 大于等于当前系统版本。

## README.md 编写规范（应用根目录）

建议仅提供中文 `README.md`。

推荐结构：

1) 产品介绍
2) 默认账户信息（如有，置顶醒目标注）
3) 主要功能（或关键特性）
4) 配置与使用说明
5) 注意事项
6) 更新日志 / 兼容性

中文模板：

```markdown
# 产品名称

> 一句话/一段话介绍应用用途与优势。

## 默认账户信息
- 用户名：admin
- 密码：安装时生成或见环境变量

## 主要功能
- 功能一……
- 功能二……
```

注意：命令、配置示例使用代码块并标注语言类型（如 `yaml`、`bash`、`docker`）。

## 版本目录 data.yml（安装表单）

路径：`<应用根>/<版本>/data.yml`；作用：定义安装参数与表单字段，驱动安装流程。

核心结构：

```yaml
additionalProperties:
  formFields:
    - type: text
      labelZh: "演示标签"

      required: true
      envKey: "APP_NAME"
      default: "demo"
      edit: false
      random: false
      rule: "paramCommon"
```

常用字段类型：text / password / number / select / service / apps。
规则校验：paramPort、paramExtUrl、paramCommon、paramComplexity。
示例：应用 HTTP 端口（强校验端口占用）

```yaml
- type: number
  labelZh: "HTTP 端口"

  required: true
  envKey: "PANEL_APP_PORT_HTTP"
  default: 8080
  edit: true
  rule: "paramPort"
```

说明：包含 `PANEL_APP_PORT` 前缀的 envKey 会触发 1Panel 的端口占用检查。

示例：数据库服务选择（主类型+实例）

```yaml
- type: apps
  labelZh: "数据库服务"

  required: true
  envKey: "PANEL_DB_TYPE"
  default: "postgresql"
  edit: false
  values:
    - {label: "PostgreSQL", value: "postgresql"}
    - {label: "MySQL",     value: "mysql"}
    - {label: "MariaDB",   value: "mariadb"}
  child:
    type: service
    envKey: "PANEL_DB_HOST"
    required: true
    default: ""
```
示例：数据库名称与用户（自动生成 + 校验）

```yaml
- type: text
  labelZh: "数据库名称"

  required: true
  envKey: "PANEL_DB_NAME"
  default: ""
  edit: false
  random: true
  rule: "paramCommon"

- type: text
  labelZh: "数据库用户"

  required: true
  envKey: "PANEL_DB_USER"
  default: ""
  edit: false
  random: true
  rule: "paramCommon"
```

示例：数据库密码（敏感信息，强制随机）

```yaml
- type: password
  labelZh: "数据库密码"

  required: true
  envKey: "PANEL_DB_USER_PASSWORD"
  default: ""
  edit: false
  random: true
```

示例：Redis 缓存服务（服务实例选择）

```yaml
- type: service
  key: "redis"
  labelZh: "Redis 缓存服务"

  required: true
  envKey: "REDIS_HOST"
  default: ""
  edit: false
```
## docker-compose.yml（容器编排）

放置于版本目录；建议遵循以下最小示例，并与表单 envKey 对齐：

```yaml
networks:
  1panel-network:
    external: true

services:
  halo:
    container_name: ${CONTAINER_NAME}
    image: halohub/halo-pro:2.21.8
    restart: always
    networks: [1panel-network]
    volumes:
      - ./data:/root/.halo2
    ports:
      - ${PANEL_APP_PORT_HTTP}:8090
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8090/actuator/health/readiness"]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 30s
    command:
      - --spring.r2dbc.url=r2dbc:pool:${PANEL_DB_TYPE}://${PANEL_DB_HOST}:${PANEL_DB_PORT}/${PANEL_DB_NAME}
      - --spring.r2dbc.username=${PANEL_DB_USER}
      - --spring.r2dbc.password=${PANEL_DB_USER_PASSWORD}
```
```yaml
    command:
      - --spring.sql.init.platform=${PANEL_DB_TYPE}
      - --halo.external-url=${HALO_EXTERNAL_URL}
    environment:
      - JVM_OPTS=
    labels:
      createdBy: "Apps"
```

检查清单：
- networks 引用外部 `1panel-network`，确保与数据库/缓存互通。
- 容器名使用 `${CONTAINER_NAME}`，多容器可追加后缀。
- 所有 `${...}` 变量需与表单 envKey 一致。
- 核心数据目录使用 volumes 挂载以持久化。
- 必须保留 `labels.createdBy: "Apps"` 以便平台识别管理。

## 脚本目录 scripts（可选）

支持的生命周期脚本：
- `init.sh`：首次安装时执行
- `upgrade.sh`：版本升级任务阶段执行
- `uninstall.sh`：卸载时执行

最佳实践：
- 幂等：重复执行不产生副作用；提前判断资源/文件是否存在。
- 失败即退出：非零即失败，`set -euo pipefail`。
- 清晰日志：向 stdout 输出关键步骤与错误信息。
- 参数输入：通过环境变量接入安装表单参数。

示例骨架：

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "[init] starting..."
# 读取参数：如 $PANEL_DB_HOST / $PANEL_DB_NAME 等
echo "[init] done"
```
## data 目录（预置数据）

- 当应用需要初始配置/模板/示例数据时，将文件放入版本目录下的 `data/`，再通过 volumes 挂载到容器路径。
- 若目录为空，为便于纳入版本控制请添加 `.gitkeep`。

示例挂载：

```yaml
volumes:
  - ./data:/root/.halo2
```

## 常见问题与排错

- 端口冲突：确保使用 `PANEL_APP_PORT_*` 前缀并通过面板占用检查。
- 变量未生效：核对版本 data.yml 的 `envKey` 与 compose 中 `${...}` 完全一致。
- 容器互通失败：缺失 `1panel-network` 外部网络或服务未加入该网络。
- 数据丢失：未挂载数据卷或挂载路径错误。
- 升级失败：`upgrade.sh` 未考虑旧版本残留或不可逆变更。

## 安全清单（强烈建议）
- 所有默认密码必须随机生成并仅在安装阶段展示。
- 严禁硬编码凭据；敏感信息通过环境变量传递。
- 镜像来源可信且版本可追溯；固定 tag 或 digest。
- 容器最小权限运行；避免不必要的特权与挂载。
- 健康检查覆盖关键依赖（数据库就绪、应用就绪）。
## 交付与上架验证

- 本地使用 `docker compose config` 预检语法与变量引用。
- 在 1Panel 测试环境完成一次完整安装、访问与卸载。
- 导出安装日志与容器日志，确认无严重告警。
- 编写/更新根目录 README 与版本目录 data.yml 说明。

## AI 协作提示片段

将以下关键信息喂给 AI，便于其生成一致的应用：
- 应用根目录结构与版本目录结构
- 根 data.yml 字段约定与最小示例
- 版本 data.yml 表单字段定义与 envKey 列表
- docker-compose.yml 变量引用约定与最小示例
- 生命周期脚本约束与幂等要求

## 附录：快速质检脚本（人工检查清单）

- [ ] `logo.png` 为 180x180 且 ≤10KB
- [ ] 根 `data.yml` 含 `additionalProperties.key/name/type/tags` 等必需字段
- [ ] 版本 `data.yml` 含所需 `formFields` 且规则与 compose 变量一致
- [ ] `docker-compose.yml` 包含 `1panel-network`、`${CONTAINER_NAME}`、`labels.createdBy: "Apps"`
- [ ] `data/` 挂载路径正确且可写
- [ ] 如有脚本，具备幂等与日志输出
- [ ] README（中文）结构完整、含默认账户信息（如适用）

—— 祝交付顺利。
