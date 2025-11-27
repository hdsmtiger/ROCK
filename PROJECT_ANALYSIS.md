# ROCK 项目技术文档

## 项目概述

**ROCK (Reinforcement Open Construction Kit)** 是一个易于使用、可扩展的沙盒环境管理框架，主要用于智能体强化学习环境。它提供了构建、管理和调度强化学习环境的工具，适用于开发、测试和研究场景。

### 核心特性
- **多协议动作支持**：支持GEM、Bash、Chat等多种动作协议
- **沙盒运行时**：具有状态的运行时环境，支持多种隔离机制，确保一致性和安全性
- **灵活部署**：支持不同部署方式，满足不同环境需求和操作系统
- **统一SDK接口**：为Env和Sandbox交互提供清晰的Python SDK
- **分层服务架构**：分布式Admin、Worker和Rocklet架构，可扩展的资源管理
- **高效资源管理**：自动化的沙盒生命周期管理，可配置的资源分配

## 系统架构

### 架构组件

ROCK采用客户端-服务器架构，主要包含以下组件：

#### 1. ROCK SDK
- **位置**: `rock/sdk/`
- **功能**: 环境开发工具包，提供开发工具帮助开发者构建、注册、部署和访问环境
- **关键模块**:
  - `rocklet/` - 轻量级代理服务组件
  - `envs/` - GEM协议兼容环境接口
  - `sandbox/` - 沙盒SDK

#### 2. ROCK CLI
- **位置**: `rock/cli/`
- **功能**: 命令行界面工具，帮助用户管理和操作环境和服务
- **入口点**: `rock/cli/main.py`
- **核心功能**: 
  - 沙盒构建、推送和运行命令
  - 配置文件管理
  - 远程服务连接

#### 3. ROCK Admin
- **位置**: `rock/admin/`
- **功能**: 调度节点，负责部署环境为沙盒，管理沙盒资源调度和分配
- **端口**: 默认8080
- **核心服务**:
  - `sandbox_api.py` - 沙盒管理API
  - `sandbox_proxy_api.py` - 沙盒代理API
  - `warmup_api.py` - 环境预热API
  - `gem/api.py` - GEM协议API

#### 4. ROCK Worker
- **位置**: `rock/deployments/`
- **功能**: 工作节点，分配机器物理资源给沙盒，执行具体沙盒运行时
- **部署方式**:
  - Docker部署 (`deployments/docker.py`)
  - Ray部署 (`deployments/ray.py`)
  - 本地部署 (`deployments/local.py`)
  - 远程部署 (`deployments/remote.py`)

#### 5. ROCK Rocklet
- **位置**: `rock/rocklet/`
- **功能**: 轻量级代理服务组件，处理SDK到沙盒的动作通信，支持外部互联网服务访问
- **服务**: `server.py` - FastAPI服务器，默认端口8000

#### 6. ROCK Envhub
- **位置**: `rock/envhub/`
- **功能**: 环境仓库，提供环境数据注册和存储
- **组件**: `core/envhub.py` - 核心环境管理

### 沙盒生命周期管理

#### 核心管理器
- **GemManager** (`rock/sandbox/gem_manager.py`): 继承自SandboxManager，专门处理GEM协议环境
- **BaseManager** (`rock/sandbox/base_manager.py`): 基础管理器，提供通用的沙盒管理功能
- **SandboxManager** (`rock/sandbox/sandbox_manager.py`): 具体的沙盒管理器实现

#### 部署管理器
- **DeploymentManager** (`rock/deployments/manager.py`): 管理不同类型的部署配置
- **支持部署类型**:
  - Docker部署 (最常用)
  - Ray部署 (分布式)
  - 本地部署
  - 远程部署

### 协议支持

#### GEM协议兼容性
ROCK完全兼容GEM接口，保持与强化学习环境的标准化接口：
- `make(env_id)`: 创建环境实例
- `reset(seed)`: 重置环境状态
- `step(action)`: 执行动作并返回结果

## 核心功能模块

### 1. 配置管理
**位置**: `rock/config.py`
- 集中化配置管理
- 支持环境变量和配置文件
- 动态配置更新（Nacos支持）
- 嵌套配置对象（Ray、Redis、Sandbox等）

### 2. 环境变量管理
**位置**: `rock/env_vars.py`
- 统一的环境变量管理
- 类型检查和验证
- 支持默认值和必需变量

### 3. 日志系统
**位置**: `rock/logger.py`
- 统一的日志配置
- 支持不同级别日志
- 结构化日志输出

### 4. 工具类库
**位置**: `rock/utils/`
- 数据库工具 (`database.py`)
- HTTP工具 (`http.py`)
- 系统工具 (`system.py`)
- 重试机制 (`retry.py`)
- Docker工具 (`docker.py`)
- 提供者模式 (`providers/`)

### 5. 动作处理系统
**位置**: `rock/actions/`
- 标准化的请求/响应模型
- 支持多种动作类型：
  - 环境动作 (EnvMake, EnvReset, EnvStep等)
  - 沙盒动作 (CreateSession, RunCommand等)
  - 响应处理 (Response处理)

### 6. 指标监控
**位置**: `rock/admin/metrics/`
- 实时指标收集
- 性能监控
- 聚合指标管理

## 数据流架构

### 1. 环境创建流程
```
用户请求 -> CLI/SDK -> Admin API -> GemManager -> DockerDeployment -> SandboxActor
```

### 2. 环境执行流程
```
Env.make() -> Admin API -> Sandbox状态检查 -> Rocklet通信 -> 环境执行
```

### 3. 沙盒管理流程
```
SandboxManager -> DeploymentManager -> AbstractDeployment -> 实际容器/进程管理
```

## 部署架构

### 本地部署
- 使用Docker容器化
- 默认使用python:3.11镜像
- 本地Redis缓存
- 单机模式

### 分布式部署
- Ray集群支持
- 多节点资源分配
- Nacos配置服务
- Redis集群缓存

### 运行时环境
- **uv环境管理**: 推荐使用uv管理的Python环境
- **Docker运行时**: 容器化隔离
- **pip运行时**: 传统Python包管理

## 测试架构

### 测试组织
**位置**: `tests/`
- `unit/` - 单元测试
- `integration/` - 集成测试
- 支持异步测试
- Docker依赖测试标记

### 测试覆盖
- 核心管理器测试
- 部署系统测试
- SDK功能测试
- API接口测试

## 依赖关系

### 核心依赖
- **FastAPI**: Web框架
- **Ray**: 分布式计算
- **Docker**: 容器化
- **uv**: Python包管理
- **Redis**: 缓存和状态管理
- **Pydantic**: 数据验证

### 可选依赖
- **Nacos**: 配置中心
- **Apscheduler**: 任务调度
- **GEM-LLM**: 强化学习环境
- **OpenTelemetry**: 可观测性

### 开发依赖
- **pytest**: 测试框架
- **ruff**: 代码检查
- **pre-commit**: 代码质量

## 性能特性

### 1. 资源管理
- 自动沙盒生命周期管理
- 资源配额和限制
- 动态资源分配
- 内存和CPU监控

### 2. 并发处理
- 异步操作支持
- 连接池管理
- 请求超时控制
- 并发限制

### 3. 监控告警
- 实时指标收集
- 性能监控
- 健康检查
- 自动清理机制

## 扩展性设计

### 1. 插件化架构
- 部署钩子机制
- 可插拔的部署类型
- 自定义验证器
- 灵活的运行时环境

### 2. 协议扩展
- 多协议支持框架
- 协议适配器模式
- 标准化接口设计

### 3. 配置扩展
- 环境特定配置
- 动态配置更新
- 配置分层管理

## 安全性考虑

### 1. 隔离机制
- 容器化隔离
- 进程隔离
- 网络隔离
- 文件系统隔离

### 2. 访问控制
- 认证令牌支持
- 请求头验证
- API权限控制

### 3. 安全配置
- 安全的默认配置
- 环境变量敏感信息
- 运行时安全检查

## 部署和运维

### 启动流程
```bash
# 开发环境初始化
make init

# 启动管理服务
rock admin start

# 启动代理服务
rocklet --port 8000
```

### 配置管理
- YAML配置文件支持
- 环境变量覆盖
- 动态配置更新
- 配置验证

### 日志和监控
- 结构化日志输出
- 性能指标收集
- 访问日志记录
- 错误追踪

## 开发和贡献

### 开发环境设置
1. 安装Docker和uv
2. 运行 `make init` 初始化环境
3. 安装pre-commit钩子
4. 运行测试验证

### 代码质量
- Pre-commit钩子检查
- Ruff代码风格检查
- 类型注解支持
- 文档字符串要求

### 测试要求
- 单元测试覆盖核心功能
- 集成测试验证端到端流程
- 性能测试确保扩展性
- 安全测试验证隔离性

## 总结

ROCK是一个设计良好的强化学习环境管理框架，具有清晰的架构设计、良好的可扩展性和丰富的功能特性。它通过分层架构、标准协议支持和灵活的部署选项，为强化学习研究和应用提供了强大的基础设施支持。

框架的核心优势包括：
- 标准化的GEM协议兼容
- 灵活的部署选项
- 统一的SDK接口
- 强大的资源管理能力
- 完善的监控和指标体系

这使得ROCK不仅适用于研究环境，也能满足生产环境的需求。