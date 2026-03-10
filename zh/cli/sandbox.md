

  CLI 命令

  
# 沙盒（sandbox）CLI

管理基于 Docker 的沙盒容器，为智能体（agent）提供隔离的执行环境。

## 概述

OpenClaw 支持在隔离的 Docker 容器中运行智能体，以增强安全性。`sandbox` 命令帮助你管理这些容器——尤其是在更新镜像或修改配置后，让变更及时生效。

## 命令

### openclaw sandbox explain

查看当前**实际生效的**沙盒配置，包括模式、范围、工作空间访问权限、工具策略，以及权限提升门控（附带对应的配置项路径）。

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### openclaw sandbox list

列出所有沙盒容器，显示其状态和配置信息。

```bash
openclaw sandbox list
openclaw sandbox list --browser  # 仅列出浏览器容器
openclaw sandbox list --json     # 以 JSON 格式输出
```

**输出内容包括：**

-   容器名称和状态（运行中/已停止）
-   Docker 镜像及是否与配置匹配
-   存在时长（创建以来的时间）
-   空闲时长（上次使用以来的时间）
-   关联的会话/智能体

### openclaw sandbox recreate

删除沙盒容器，使其在下次使用时用更新的镜像或配置重新创建。

```bash
openclaw sandbox recreate --all                # 重建所有容器
openclaw sandbox recreate --session main       # 指定会话
openclaw sandbox recreate --agent mybot        # 指定智能体
openclaw sandbox recreate --browser            # 仅浏览器容器
openclaw sandbox recreate --all --force        # 跳过确认提示
```

**选项：**

-   `--all`: 重建所有沙盒容器
-   `--session `: 重建指定会话的容器
-   `--agent `: 重建指定智能体的容器
-   `--browser`: 仅重建浏览器容器
-   `--force`: 跳过确认提示

**注意：** 容器会在智能体下次使用时自动重建。

## 典型使用场景

### 更新 Docker 镜像后

```bash
# 拉取新镜像
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# 更新配置以使用新镜像
# 编辑 agents.defaults.sandbox.docker.image（或 agents.list[].sandbox.docker.image）

# 重建容器让变更生效
openclaw sandbox recreate --all
```

### 修改沙盒配置后

```bash
# 编辑 agents.defaults.sandbox.*（或 agents.list[].sandbox.*）

# 重建以应用新配置
openclaw sandbox recreate --all
```

### 修改 setupCommand 后

```bash
openclaw sandbox recreate --all
# 或只针对某个智能体：
openclaw sandbox recreate --agent family
```

### 只更新某个智能体的容器

```bash
openclaw sandbox recreate --agent alfred
```

## 为什么需要手动重建？

**问题所在：** 当你更新沙盒 Docker 镜像或修改配置后：

-   现有容器仍会继续使用旧设置运行
-   容器默认在闲置 24 小时后才会被自动清理
-   频繁使用的智能体会让旧容器无限期运行下去

**解决方案：** 使用 `openclaw sandbox recreate` 强制删除旧容器。下次需要时，它们会自动用当前配置重新创建。

**小贴士：** 优先使用 `openclaw sandbox recreate` 而非手动执行 `docker rm`。它能正确处理网关的容器命名规则，避免在范围或会话键变更时出现不匹配问题。

## 配置参考

沙盒设置位于 `~/.openclaw/openclaw.json` 的 `agents.defaults.sandbox` 下（针对单个智能体的覆盖配置在 `agents.list[].sandbox` 中）：

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all", // off, non-main, all
        "scope": "agent", // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-",
          // ... 更多 Docker 选项
        },
        "prune": {
          "idleHours": 24, // 闲置 24 小时后自动清理
          "maxAgeDays": 7, // 存在超过 7 天自动清理
        },
      },
    },
  },
}
```

## 相关文档

-   [沙盒文档](../gateway/sandboxing.md)
-   [智能体配置](../concepts/agent-workspace.md)
-   [Doctor 命令](../gateway/doctor.md) - 检查沙盒配置是否正确

[reset](./reset.md)[secrets](./secrets.md)