

  技能（Skills）

  
# 创建技能

想让您的 OpenClaw 助手学会新本领吗？技能（skills）就是答案。OpenClaw 专为易于扩展而设计，而技能正是为助手添加新能力的主要方式。

## 什么是技能？

简单来说，一个技能就是一个目录，里面包含：

- **`SKILL.md` 文件**：告诉大语言模型该做什么、该怎么用工具
- **脚本或资源文件**（可选）：技能需要用到的辅助材料

## 动手实践：创建您的第一个技能

让我们从零开始，创建一个简单的"打招呼"技能。

### 第一步：创建技能目录

技能存放在工作区的 `~/.openclaw/workspace/skills/` 目录下。先给您的技能建个"家"：

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 第二步：编写 SKILL.md

在刚创建的目录里新建 `SKILL.md` 文件。这个文件用 YAML 格式定义元数据，用 Markdown 写指令，非常直观：

```
---
name: hello_world
description: 一个简单的打招呼技能
---

# Hello World 技能

当用户请求问候时，使用 `echo` 工具说"来自您自定义技能的问候！"
```

就这么简单！大语言模型会读懂这些指令，并在合适的时机执行。

### 第三步：添加工具（可选）

如果需要更强大的能力，您可以在 frontmatter 里定义专属工具，也可以直接让智能体调用系统已有的工具（比如 `bash` 或 `browser`）。

### 第四步：让 OpenClaw 识别新技能

告诉您的智能体"刷新技能"，或者重启网关。OpenClaw 会自动发现新目录并索引 `SKILL.md` 文件。

## 写好技能的几个建议

- **言简意赅**：告诉模型要做什么，别教它怎么做 AI
- **安全第一**：如果技能用到了 `bash`，务必防止用户输入被恶意利用
- **先测后用**：本地测试命令 `openclaw agent --message "use my new skill"`，确保一切正常

## 发现更多技能

想看看别人都写了什么好玩的技能？或者想分享您的创意？欢迎访问 [ClawHub](https://clawhub.com)，这里是技能的社区市场。

[多智能体沙盒与工具](./multi-agent-sandbox-tools.md)[斜杠命令](./slash-commands.md)