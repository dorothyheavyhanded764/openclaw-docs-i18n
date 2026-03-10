

  浏览器

  
# 浏览器登录

## 手动登录（推荐）

当网站需要登录时，请在**宿主**浏览器配置文件（即 openclaw 浏览器）中**手动登录**。**切勿**将您的账号密码交给模型。自动登录很容易触发网站的反机器人机制，甚至可能导致账户被锁定。返回主浏览器文档：[浏览器](./browser.md)。

## 使用哪个 Chrome 配置文件？

OpenClaw 使用一个**专用的 Chrome 配置文件**（名称为 `openclaw`，界面呈橙色色调），与您日常使用的浏览器配置完全隔离。有两种方式可以访问：

1.  **让智能体（agent）打开浏览器**，然后您自己登录。
2.  **通过 CLI 打开**：

```bash
openclaw browser start
openclaw browser open https://x.com
```

如果您有多个配置文件，可以通过 `--browser-profile ` 指定（默认为 `openclaw`）。

## X/Twitter：推荐流程

-   **阅读/搜索/查看帖子串：** 使用**宿主**浏览器（手动登录）。
-   **发布更新：** 使用**宿主**浏览器（手动登录）。

## 沙盒隔离 + 宿主浏览器访问

沙盒化的浏览器会话**更容易**触发机器人检测。对于 X/Twitter 等审核严格的网站，建议使用**宿主**浏览器。如果智能体（agent）运行在沙盒中，浏览器工具默认会使用沙盒环境。要允许控制宿主浏览器，请添加以下配置：

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

然后指定目标为宿主浏览器：

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

或者，直接为需要发布更新的智能体（agent）禁用沙盒隔离。

[浏览器（OpenClaw 托管）](./browser.md)[Chrome 扩展](./chrome-extension.md)