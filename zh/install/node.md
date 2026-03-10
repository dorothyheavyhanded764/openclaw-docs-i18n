

  Node 运行时

  
# Node.js

OpenClaw 需要 **Node 22 或更高版本**。[安装脚本](../install.md#install-methods) 会自动检测并安装 Node——本页面适用于你希望自行设置 Node 并确保所有配置正确的情况（版本、PATH、全局安装）。

## 检查版本

```bash
node -v
```

如果输出 `v22.x.x` 或更高版本，说明一切就绪。如果未安装 Node 或版本过低，请选择下面的安装方式。

## 安装 Node

 

版本管理器让你可以轻松切换 Node 版本。常用选项：

- [**fnm**](https://github.com/Schniz/fnm)——快速、跨平台
- [**nvm**](https://github.com/nvm-sh/nvm)——在 macOS/Linux 上广泛使用
- [**mise**](https://mise.jdx.dev/)——多语言支持（Node、Python、Ruby 等）

以 fnm 为例：

```bash
fnm install 22
fnm use 22
```

请确保你的版本管理器已在 shell 启动文件（`~/.zshrc` 或 `~/.bashrc`）中初始化。如果未初始化，新终端会话中可能找不到 `openclaw` 命令，因为 PATH 中不包含 Node 的 bin 目录。

## 故障排除

### openclaw: command not found

这几乎总是意味着 npm 的全局 bin 目录不在你的 PATH 中。

### 步骤 1：查找全局 npm 前缀

```bash
npm prefix -g
```

### 步骤 2：检查是否在 PATH 中

```bash
echo "$PATH"
```

在输出中查找 `<npm-prefix>/bin`（macOS/Linux）或 `<npm-prefix>`（Windows）。

### 步骤 3：添加到 shell 启动文件

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

通过"设置" → "系统" → "环境变量"，将 `npm prefix -g` 的输出添加到系统 PATH。

### npm install -g 权限错误（Linux）

如果遇到 `EACCES` 错误，请将 npm 全局前缀切换到用户可写的目录：

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

将 `export PATH=...` 这行添加到 `~/.bashrc` 或 `~/.zshrc` 使其永久生效。

[诊断标志](../diagnostics/flags.md)[会话管理深度解析](../reference/session-management-compaction.md)