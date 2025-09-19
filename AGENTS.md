# 1Panel 应用商店仓库工作规范（AGENTS 版，AI 友好）

本文面向在本仓库协作的开发者与 AI 代理，提炼并结构化 1panel-app-dev-guide.md 的关键约束与最佳实践，用于确保新增/修改应用的一致性、可上架性与可维护性。

适用范围：仓库内所有应用目录（`apps/*`）及其版本子目录。

目标：
- 快速创建/更新 1Panel 应用
- 保持字段/变量一致与可追溯
- 最小化风险，便于回滚
- 通过本地与面板双重校验

## 强制约束清单（务必遵守）
- 版本目录名不得以 `v` 开头（如：`2.3.2`）
- 必须保留 `networks.1panel-network: external: true`
- 必须保留 `labels.createdBy: "Apps"`
- compose 中 `${...}` 变量必须与版本 `data.yml` 的 `envKey` 一致
- 核心数据目录必须通过 `volumes` 持久化
- 端口优先使用 `PANEL_APP_PORT_*` 前缀以触发占用检查
- 默认密码/凭据不得硬编码；安装阶段随机生成并仅展示一次
- 使用可信镜像并固定 `tag` 或 `digest`
- README 为中文，包含默认账户信息（如适用）
- 表单多语言统一为中文：仅保留 `labelZh` 与 `description.zh`，移除 `labelEn`/`description.en`/`label` 多语言映射

## 目录结构（示例）
```text
apps/halo/
├── logo.png              # 180x180，≤10KB PNG
├── data.yml              # 应用声明（根）
├── README.md             # 中文说明
├── 2.2.0                 # 版本目录（禁止以 v 开头）
│   ├── data.yml          # 安装表单（版本）
│   ├── data/             # 数据卷挂载目录（可含 .gitkeep）
│   ├── scripts/          # 生命周期脚本（可选）
│   └── docker-compose.yml
└── 2.3.2
    ├── data.yml
    ├── data/
    └── docker-compose.yml
```
要点：
- 多版本并存，每个版本目录可独立部署
- 空目录建议添加 `.gitkeep` 便于纳管

## 根 data.yml（应用声明）
用于应用在商店中的基础元信息展示与分类；同时必须保留 `additionalProperties` 结构以兼容解析。

最小可用示例：
```yaml
# 官方字段（上架商店必填）
name: Halo
tags: 建站
title: 强大易用的开源建站工具
description: 强大易用的开源建站工具

# 1Panel 解析结构（必须保留）
additionalProperties:
  key: halo              # 英文唯一键（用于目录/标识）
  name: Halo
  type: website          # website/runtime/tool
  tags: [Website]
  description:
    zh: 强大易用的开源建站工具
  crossVersionUpdate: true
  limit: 0
  recommend: 5
  website: https://halo.run/
  github: https://github.com/halo-dev/halo
  document: https://docs.halo.run/
  memoryRequired: 1024
  architectures: [amd64, arm64]
  gpuSupport: false
  version: 0
  deprecated: 0
```
校验清单：
- `key` 为英文/数字/短横线/下划线组合（建议 2–30 位）
- `type` 合法：website/runtime/tool；`tags` 与官方分类兼容
- 链接为有效 `http/https`；`architectures` 与镜像支持一致

## README 编写规范（应用根目录）
推荐仅提供中文 `README.md`，包含：
1) 产品介绍
2) 默认账户信息（如有，置顶醒目标注）
3) 主要功能（或关键特性）
4) 配置与使用说明
5) 注意事项
6) 更新日志 / 兼容性

模板：
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
注意：命令示例使用代码块并标注语言类型（如 `yaml`、`bash`、`docker`）。

## 版本 data.yml（安装表单）
路径：`<应用根>/<版本>/data.yml`；用于定义安装参数与表单字段，驱动安装流程。

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
常用类型：`text` / `password` / `number` / `select` / `service` / `apps`
常用规则：`paramPort`、`paramExtUrl`、`paramCommon`、`paramComplexity`

多语言约定（仓库统一仅中文）：
- 表单字段仅保留 `labelZh` 与 `description.zh`
- 移除 `labelEn`、`description.en` 以及 `label:` 下的多语言映射
- 根 `data.yml` 与版本 `data.yml` 的中文描述均以 `zh` 为准

端口示例（强校验端口占用）：
```yaml
- type: number
  labelZh: "HTTP 端口"
  required: true
  envKey: "PANEL_APP_PORT_HTTP"
  default: 8080
  edit: true
  rule: "paramPort"
```
数据库主类型 + 实例选择：
```yaml
- type: apps
  labelZh: "数据库服务"
  required: true
  envKey: "PANEL_DB_TYPE"
  default: "postgresql"
  values:
    - { label: "PostgreSQL", value: "postgresql" }
    - { label: "MySQL",     value: "mysql" }
    - { label: "MariaDB",   value: "mariadb" }
  child:
    type: service
    envKey: "PANEL_DB_HOST"
    required: true
    default: ""
```
数据库名称与用户（自动生成 + 校验）：
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
数据库密码（敏感信息，强制随机）：
```yaml
- type: password
  labelZh: "数据库密码"
  required: true
  envKey: "PANEL_DB_USER_PASSWORD"
  default: ""
  edit: false
  random: true
```
Redis 服务实例选择：
```yaml
- type: service
  key: "redis"
  labelZh: "Redis 缓存服务"
  required: true
  envKey: "REDIS_HOST"
  default: ""
  edit: false
```
说明：包含 `PANEL_APP_PORT` 前缀的 `envKey` 会触发 1Panel 的端口占用检查。

## docker-compose 规范（版本目录）
建议遵循以下最小示例，并与表单 `envKey` 对齐：
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
      - --spring.sql.init.platform=${PANEL_DB_TYPE}
      - --halo.external-url=${HALO_EXTERNAL_URL}
    environment:
      - JVM_OPTS=
    labels:
      createdBy: "Apps"
```
检查清单：
- `1panel-network` 为外部网络，确保与数据库/缓存互通
- `${...}` 变量与版本 `data.yml` 的 `envKey` 完全一致
- 核心数据目录通过 `volumes` 持久化
- 保留 `labels.createdBy: "Apps"`

## 脚本目录 scripts（可选）
支持的生命周期脚本：
- `init.sh`：首次安装执行
- `upgrade.sh`：版本升级阶段执行
- `uninstall.sh`：卸载时执行

最佳实践：
- 幂等：重复执行无副作用（先判断再创建/修改）
- 失败即退出：`set -euo pipefail`
- 清晰日志：向 stdout 输出关键步骤与错误
- 参数输入：通过环境变量读取安装表单参数

骨架示例：
```bash
#!/usr/bin/env bash
set -euo pipefail

echo "[init] starting..."
# 读取参数：如 $PANEL_DB_HOST / $PANEL_DB_NAME 等
echo "[init] done"
```

## data 目录（预置数据）
- 应用需要初始配置/模板/示例数据时，将文件放入版本目录的 `data/`，再通过 `volumes` 挂载到容器路径
- 若目录为空，添加 `.gitkeep` 便于纳管

示例挂载：
```yaml
volumes:
  - ./data:/root/.halo2
```

## 常见问题与排错
- 端口冲突：使用 `PANEL_APP_PORT_*` 前缀并通过面板占用检查
- 变量未生效：核对版本 `data.yml` 的 `envKey` 与 compose `${...}` 一致
- 容器互通失败：缺失外部网络 `1panel-network` 或服务未加入该网络
- 数据丢失：未挂载数据卷或挂载路径错误
- 升级失败：`upgrade.sh` 未考虑旧版本残留或不可逆变更

## 安全清单（强烈建议）
- 默认密码随机生成；敏感信息仅通过环境变量传递
- 禁止在仓库中硬编码凭据
- 镜像来源可信且版本可追溯；固定 tag 或 digest
- 以最小权限运行容器，避免不必要的特权/挂载
- 健康检查覆盖关键依赖（数据库就绪、应用就绪）

## 交付与上架验证
- 使用 `docker compose config` 预检语法与变量引用
- 在 1Panel 测试环境完成完整安装、访问与卸载
- 导出安装日志与容器日志，确认无严重告警
- 完成根 `README.md` 与版本 `data.yml` 的说明与同步

## AI 协作提示片段
向 AI 提供以下关键信息以生成一致应用：
- 应用根目录与版本目录结构
- 根 `data.yml` 字段约定与最小示例
- 版本 `data.yml` 表单字段与 `envKey` 列表
- `docker-compose.yml` 变量引用约定与最小示例
- 生命周期脚本约束与幂等要求

## 附录：快速质检清单与命令

质检清单：
- [ ] `logo.png` 为 180x180 且 ≤10KB
- [ ] 根 `data.yml` 含 `additionalProperties.key/name/type/tags` 必需字段
- [ ] 版本 `data.yml` 定义所需 `formFields` 且与 compose 变量一致
- [ ] `docker-compose.yml` 含 `1panel-network`、`${CONTAINER_NAME}`、`labels.createdBy: "Apps"`
- [ ] `data/` 挂载路径正确且可写
- [ ] 如有脚本，具备幂等与日志输出
- [ ] `README.md`（中文）结构完整、含默认账户信息（如适用）

验证命令：
```bash
# YAML 语法校验（示例路径按实际应用修改）
yq eval '.' apps/halo/data.yml
yq eval '.' apps/halo/2.3.2/data.yml

# Docker Compose 配置检查
docker compose -f apps/halo/2.3.2/docker-compose.yml config

# 查看改动统计
git diff --stat
```
—— 祝交付顺利。
