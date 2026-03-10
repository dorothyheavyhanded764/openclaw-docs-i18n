

  浏览器

  
# Chrome 扩展

想让智能体（agent）直接控制你**正在使用的 Chrome 标签页**，而不是另起一个独立的浏览器窗口吗？OpenClaw Chrome 扩展正是为此设计的——只需点击工具栏上的一个按钮，就能轻松附加或分离标签页的控制权。

## 工作原理

整个系统由三个部分协同工作：

- **浏览器控制服务**（Gateway 或 node）：智能体（agent）和工具调用的 API 入口
- **本地中继服务器**（loopback CDP）：在控制服务和扩展之间搭建桥梁，默认地址为 `http://127.0.0.1:18792`
- **Chrome MV3 扩展**：通过 `chrome.debugger` API 附加到当前标签页，将 CDP 消息转发到中继服务器

附加成功后，OpenClaw 就能通过常规的 `browser` 工具来操控这个标签页了。

## 安装扩展

按以下步骤加载解压版扩展：

1. 将扩展安装到本地稳定路径：

```bash
openclaw browser extension install
```

2. 查看扩展的安装路径：

```bash
openclaw browser extension path
```

3. 打开 Chrome，访问 `chrome://extensions`
   - 开启"开发者模式"
   - 点击"加载已解压的扩展程序"，选择上一步打印的目录

4. 固定扩展图标到工具栏，方便后续操作。

## 更新扩展

扩展文件随 OpenClaw 发布包（npm 包）一起分发，无需单独构建。升级 OpenClaw 后：

- 重新运行 `openclaw browser extension install`，刷新 OpenClaw 状态目录下的扩展文件
- Chrome 访问 `chrome://extensions`，点击扩展上的"重新加载"

## 首次使用：配置网关令牌

OpenClaw 内置了一个名为 `chrome` 的浏览器配置文件，默认指向扩展中继端口。首次使用前，需要打开扩展的选项页面，配置以下内容：

- `Port`（端口）：默认 `18792`
- `Gateway token`（网关令牌）：必须与 `gateway.auth.token` 或 `OPENCLAW_GATEWAY_TOKEN` 环境变量一致

配置完成后，可以这样使用：

- 命令行：`openclaw browser --browser-profile chrome tabs`
- 智能体（agent）工具：调用 `browser` 时指定 `profile="chrome"`

如果需要使用不同的名称或端口，可以创建自定义配置文件：

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

### 自定义网关端口

如果你修改了网关端口，扩展中继端口会自动按规则派生：**扩展中继端口 = 网关端口 + 3**

例如，如果 `gateway.port: 19001`，则扩展中继端口为 `19004`（19001 + 3）。记得在扩展选项页面中更新端口设置。

## 附加与分离标签页

操作非常简单：

- 打开你想让 OpenClaw 控制的标签页
- 点击扩展图标，徽章显示 `ON` 表示已附加成功
- 再次点击即可分离，交还控制权

## 控制范围说明

请记住：

- 扩展**不会**自动控制"你当前正在看的标签页"
- 它只控制**你明确点击扩展图标附加过的**标签页
- 想切换控制目标？打开目标标签页，再点一次扩展图标即可

## 状态徽章与常见问题

根据徽章状态判断当前情况：

- `ON`：已附加，OpenClaw 可以控制该标签页
- `…`：正在连接本地中继服务器
- `!`：中继服务器无法连接或认证失败（最常见的原因：中继服务未启动，或网关令牌缺失/错误）

遇到 `!` 时：

- 确保本机正在运行 Gateway（默认配置），或者当 Gateway 在远程时，在本机运行 node host
- 打开扩展选项页面，它会自动检测中继可达性和网关令牌认证状态

## 远程网关场景

### 本地网关（Gateway 和 Chrome 在同一台机器）

这是最常见的默认配置。Gateway 在本地启动浏览器控制服务和中继服务器，扩展与本机中继通信，CLI 和工具调用直接发往 Gateway，无需额外配置。

### 远程网关（Gateway 在其他机器）

当 Gateway 运行在远程机器时，你需要在运行 Chrome 的机器上启动一个 node host。Gateway 会将浏览器操作代理到这个 node，而扩展和中继仍然保留在浏览器所在的机器上。

如果连接了多个 node，可以通过 `gateway.nodes.browser.node` 指定目标 node，或设置 `gateway.nodes.browser.mode` 来控制路由策略。

## 沙盒环境注意事项

如果你的智能体（agent）会话启用了沙盒（`agents.defaults.sandbox.mode != "off"`），`browser` 工具的使用会受到限制：

- 默认情况下，沙盒会话通常访问**沙盒浏览器**（`target="sandbox"`），而非你本机的 Chrome
- Chrome 扩展的中继接管需要控制**主机**上的浏览器控制服务

解决方案：

- 最简单：从**非沙盒**的会话或智能体（agent）中使用扩展
- 或者，允许沙盒会话控制主机浏览器：

```json
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

配置后，确保工具策略没有拒绝 `browser` 工具，并在调用时指定 `target="host"`。调试时可以运行 `openclaw sandbox explain` 查看详细信息。

## 远程访问安全建议

- 让 Gateway 和 node host 保持同一个 tailnet 网络中，避免将中继端口暴露到局域网或公网
- 谨慎配对 node，如果不需要远程控制，可设置 `gateway.nodes.browser.mode="off"` 禁用浏览器代理路由

## 扩展路径的工作机制

`openclaw browser extension path` 输出的是扩展文件的**已安装**目录路径。CLI 不会直接输出 `node_modules` 中的路径，因为那个位置不够稳定。

请务必先运行 `openclaw browser extension install`，将扩展复制到 OpenClaw 状态目录下的固定位置。如果移动或删除了该目录，Chrome 会把扩展标记为损坏，直到你从有效路径重新加载。

## 安全提示（请务必阅读）

这项功能很强大，但也伴随风险——相当于把浏览器的"操作权"交给了模型。

扩展使用 Chrome 的 `chrome.debugger` API。附加后，模型可以：

- 在标签页中点击、输入、导航
- 读取页面内容
- 访问该标签页登录会话能访问的所有资源

这与 OpenClaw 管理的独立浏览器配置文件**不同，没有隔离保护**。如果你附加了日常使用的浏览器配置文件或标签页，就等于授权访问你在这个账户下的所有状态。

**建议：**

- 为扩展中继专门创建一个独立的 Chrome 配置文件，与个人浏览分开
- Gateway 和 node host 仅限 tailnet 内网访问，依赖 Gateway 认证和 node 配对机制
- 不要将中继端口绑定到 `0.0.0.0` 暴露到局域网，也不要使用 Funnel 公开暴露
- 中继服务器会拦截非扩展来源的请求，并要求 `/cdp` 和 `/extension` 接口都进行网关令牌认证

**相关内容：**

- 浏览器工具概述：[浏览器](./browser.md)
- 安全审计详情：[安全](../gateway/security.md)
- Tailscale 配置：[Tailscale](../gateway/tailscale.md)

[浏览器登录](./browser-login.md)[浏览器故障排除](./browser-linux-troubleshooting.md)