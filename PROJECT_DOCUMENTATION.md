# ROCK 项目技术文档

## 项目概述

ROCK (Reinforcement Open Construction Kit) 是一个专为强化学习设计的大规模可扩展环境管理框架。项目采用客户端-服务器架构，支持不同级别的隔离机制，确保环境稳定运行，同时通过SDK支持与各种强化学习训练框架的集成。

### 核心特性

- **多协议动作支持**: 支持GEM、Bash和Chat等多种协议
- **沙箱运行时**: 有状态的运行时环境，提供多级隔离机制
- **灵活部署**: 支持多种部署方式，适配不同环境需求
- **统一SDK接口**: 提供清洁的Python SDK用于环境和沙箱交互
- **分层服务架构**: Admin、Worker和Rocklet分布式架构实现可扩展的资源管理
- **高效资源管理**: 自动沙箱生命周期管理和可配置的资源分配

## 系统架构

### 服务架构层次

```
┌─────────────────────────────────────────────────────────────────┐
│                        ROCK SDK (客户端)                           │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                GEM Protocol 接口                              │ │
│  │  make(env_id), reset(seed), step(action)                     │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                 │
┌─────────────────────────────────────────────────────────────────┐
│                      ROCK CLI 工具                              │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────┐ │
│  │  SandBox命令    │ │  Admin命令      │ │   Config命令        │ │
│  │  build/push/run │ │  start/stop     │ │   管理配置           │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                 │
┌─────────────────────────────────────────────────────────────────┐
│                    ROCK Admin (调度节点)                          │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────┐ │
│  │  沙箱管理       │ │  资源调度       │ │   环境管理           │ │
│  │  SandboxManager│ │  RayService     │ │   EnvService         │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                 │
┌─────────────────────────────────────────────────────────────────┐
│                    ROCK Worker (工作节点)                        │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────┐ │
│  │  资源分配       │ │  沙箱运行       │ │   容器编排           │ │
│  │  ResourceAlloc  │ │  SandboxExec    │ │   DockerManage      │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                 │
┌─────────────────────────────────────────────────────────────────┐
│                 ROCK Rocklet (代理服务)                           │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────┐ │
│  │  SDK通信代理    │ │  网络访问代理   │ │   服务发现           │ │
│  │  SDK-Proxy      │ │  InternetProxy  │ │   ServiceDiscovery  │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                 │
┌─────────────────────────────────────────────────────────────────┐
│                 ROCK Envhub (环境仓库)                           │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────┐ │
│  │  环境注册       │ │  环境存储       │ │   环境索引           │ │
│  │  EnvRegister    │ │  EnvStorage     │ │   EnvIndex          │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 核心组件详解

#### 1. Admin 组件 (`rock/admin/`)

**功能**: 负责沙箱的部署和资源调度管理
- **SandboxManager**: 沙箱生命周期管理
- **RayService**: Ray分布式计算服务
- **GemManager**: GEM协议环境管理器
- **API服务端点**: 提供RESTful API接口

**核心文件**:
- `main.py`: Admin服务入口点
- `core/ray_service.py`: Ray服务管理
- `entrypoints/sandbox_api.py`: 沙箱API接口
- `gem/api.py`: GEM协议API

#### 2. CLI 组件 (`rock/cli/`)

**功能**: 提供命令行工具管理ROCK服务
- **Sandbox命令**: 构建、推送、运行沙箱
- **Admin命令**: 启动/停止管理服务
- **Config命令**: 配置管理

**核心文件**:
- `main.py`: CLI入口点
- `command/command.py`: 命令抽象基类
- `loader.py`: 动态命令加载器

#### 3. Sandbox 组件 (`rock/sandbox/`)

**功能**: 沙箱执行环境管理
- **SandboxActor**: 沙箱执行单元
- **SandboxManager**: 沙箱管理器
- **GemManager**: GEM环境管理器
- **服务组件**: 代理服务、预热服务

#### 4. Deployment 组件 (`rock/deployments/`)

**功能**: 支持多种部署方式
- **DockerDeployment**: Docker容器部署
- **RayDeployment**: Ray分布式部署
- **LocalDeployment**: 本地部署
- **RemoteDeployment**: 远程部署

#### 5. SDK 组件 (`rock/sdk/`)

**功能**: 提供Python SDK用于环境交互
- **Env接口**: GEM标准环境接口
- **Sandbox接口**: 沙箱操作接口
- **Builder接口**: 环境构建工具

#### 6. Utils 组件 (`rock/utils/`)

**功能**: 提供通用工具和功能
- **Provider**: 配置和资源提供者
- **Docker**: Docker容器操作
- **HTTP**: HTTP客户端工具
- **Database**: 数据库操作

## 核心配置

### 环境变量配置

```python
# 核心配置环境变量
ROCK_PROJECT_ROOT          # 项目根目录
ROCK_PYTHON_ENV_PATH       # Python环境路径
ROCK_ENVHUB_DB_URL         # EnvHub数据库URL
ROCK_CONFIG                # 配置文件路径
ROCK_RAY_NAMESPACE         # Ray命名空间
ROCK_ADMIN_ENV             # 管理环境(local/prod)
ROCK_ADMIN_ROLE            # 管理角色(admin/proxy)
```

### 配置文件结构

```yaml
# rock-config.yml
ray:
  address: "ray://ray-cluster:10001"
  namespace: "xrl-sandbox"

runtime:
  enable_auto_clear: true
  project_root: "/app/rock"
  python_env_path: "/opt/uv/venv/bin/python"
  envhub_db_url: "sqlite:///app/data/envhub.db"

redis:
  host: "redis-cluster"
  port: 6379
  password: "your-password"

sandbox_config:
  actor_resource: "CPU"
  actor_resource_num: 2.0
  gateway_num: 1
```

## 核心API接口

### 沙箱管理API

```python
# 启动沙箱
POST /apis/envs/sandbox/v1/start
{
  "image": "python:3.11",
  "command": "python script.py",
  "resources": {"memory": "4g", "cpus": 2}
}

# 获取沙箱状态
GET /apis/envs/sandbox/v1/status/{sandbox_id}

# 停止沙箱
POST /apis/envs/sandbox/v1/stop/{sandbox_id}
```

### GEM环境API

```python
# 创建环境
POST /apis/v1/envs/gem/make
{
  "env_id": "game:Sokoban-v0-easy"
}

# 重置环境
POST /apis/v1/envs/gem/reset
{
  "sandbox_id": "sandbox_123",
  "seed": 42
}

# 执行动作
POST /apis/v1/envs/gem/step
{
  "sandbox_id": "sandbox_123",
  "action": "\\boxed{up}"
}
```

## 使用示例

### 1. GEM协议环境使用

```python
import rock

# 创建环境
env_id = "game:Sokoban-v0-easy"
env = rock.make(env_id)

# 重置环境
observation, info = env.reset(seed=42)

# 交互式环境操作
while True:
    action = "\\boxed{up}"  # 动作格式
    observation, reward, terminated, truncated, info = env.step(action)
    
    if terminated or truncated:
        break

# 关闭环境
env.close()
```

### 2. 沙箱SDK使用

```python
import asyncio
from rock.actions import CreateBashSessionRequest
from rock.sdk.sandbox.client import Sandbox
from rock.sdk.sandbox.config import SandboxConfig

async def run_sandbox():
    config = SandboxConfig(
        image="python:3.11", 
        memory="8g", 
        cpus=2.0
    )
    sandbox = Sandbox(config)

    await sandbox.start()
    await sandbox.create_session(CreateBashSessionRequest(session="bash-1"))
    result = await sandbox.run(cmd="echo Hello Rock", session="bash-1")
    await sandbox.stop()

asyncio.run(run_sandbox())
```

### 3. CLI命令使用

```bash
# 启动Admin服务
rock admin start --env local --port 8080

# 构建沙箱镜像
rock sandbox build --image python:3.11 --build-command "pip install -r requirements.txt"

# 运行沙箱
rock sandbox run --image python:3.11 --command "python app.py" --interactive

# 推送沙箱镜像到仓库
rock sandbox push --image my-app:latest --registry hub.alibaba-inc.com
```

## 部署架构

### 本地开发环境

```bash
# 1. 克隆仓库
git clone https://github.com/alibaba/ROCK.git
cd ROCK

# 2. 创建虚拟环境
uv venv --python 3.11 --python-preference only-managed

# 3. 安装依赖
uv sync --all-extras

# 4. 启动服务
source .venv/bin/activate
rock admin start
```

### 分布式部署

```yaml
# docker-compose.yml
version: '3.8'
services:
  rock-admin:
    image: rl-rock:0.2.1
    ports:
      - "8080:8080"
    environment:
      - ROCK_ADMIN_ENV=prod
      - ROCK_RAY_NAMESPACE=rock-prod
    volumes:
      - ./config:/app/config
      - /var/run/docker.sock:/var/run/docker.sock

  rock-worker:
    image: rl-rock:0.2.1
    environment:
      - ROCK_ADMIN_ENV=prod
    command: rock worker start
    depends_on:
      - rock-admin

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

## 性能特性

### 资源管理
- **动态资源分配**: 支持CPU、内存等资源的动态分配
- **容器隔离**: 基于Docker的进程和文件系统隔离
- **自动清理**: 支持运行时自动清理过期沙箱

### 扩展性
- **水平扩展**: 支持多Worker节点的横向扩展
- **负载均衡**: 内置资源调度和负载均衡
- **故障恢复**: 自动故障检测和恢复机制

### 监控与可观测性
- **指标收集**: 内置Prometheus指标导出
- **日志聚合**: 结构化日志和分布式追踪
- **健康检查**: 服务健康状态监控

## 安全特性

### 隔离机制
- **容器隔离**: 每个沙箱运行在独立的Docker容器中
- **网络隔离**: 可配置的网络策略
- **资源限制**: CPU、内存、磁盘等资源限制

### 访问控制
- **身份认证**: 支持令牌认证
- **权限管理**: 基于角色的访问控制
- **审计日志**: 操作审计和访问日志

## 技术栈

### 核心依赖
- **Python 3.10+**: 主要开发语言
- **FastAPI**: Web服务框架
- **Ray**: 分布式计算框架
- **Docker**: 容器运行时
- **Redis**: 缓存和状态管理
- **SQLite/PostgreSQL**: 数据存储

### 开发工具
- **uv**: Python包管理工具
- **pytest**: 测试框架
- **ruff**: 代码格式化和检查
- **pre-commit**: Git钩子管理

### 可选依赖
- **Nacos**: 配置中心
- **OSS**: 对象存储
- **APScheduler**: 任务调度
- **OTel**: 可观测性

## 扩展性设计

### 插件机制
- **Deployment插件**: 支持自定义部署后端
- **Protocol插件**: 支持新的交互协议
- **Provider插件**: 支持新的配置提供者

### 协议兼容性
- **GEM协议**: 完全兼容GEM接口标准
- **OpenAI协议**: 支持ChatGPT API调用
- **自定义协议**: 支持扩展自定义协议

## 故障排查

### 常见问题

1. **服务启动失败**
   - 检查配置文件路径和格式
   - 验证环境变量设置
   - 确认依赖服务可用性

2. **沙箱创建失败**
   - 检查Docker服务状态
   - 验证镜像可用性
   - 确认资源配额设置

3. **性能问题**
   - 检查资源使用情况
   - 验证网络连接质量
   - 优化配置参数

### 日志分析
```bash
# 查看服务日志
rock admin logs --level INFO

# 查看沙箱日志
rock sandbox logs sandbox_id

# 实时监控
tail -f logs/rock-admin.log
```

## 未来规划

### 短期目标
- 支持更多的强化学习框架集成
- 增强监控和可观测性功能
- 优化资源调度算法

### 长期目标
- 支持云原生部署(Kubernetes)
- 集成更多的强化学习算法库
- 提供可视化管理和监控界面

---

## 总结

ROCK是一个功能完善的强化学习环境管理平台，具有良好的架构设计、丰富的功能和强大的扩展性。通过分层服务和插件化设计，为强化学习研究提供了可靠的实验环境支撑。项目文档和API设计清晰，易于使用和扩展。