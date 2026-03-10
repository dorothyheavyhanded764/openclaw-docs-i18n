

  配置与操作

  
# 认证凭据语义

本文档定义了以下场景中使用的凭据资格与解析语义规范：

- `resolveAuthProfileOrder`
- `resolveApiKeyForProfile`
- `models status --probe`
- `doctor-auth`

目标是确保选择时与运行时的行为保持一致。

## 稳定的原因代码

- `ok`
- `missing_credential`
- `invalid_expires`
- `expired`
- `unresolved_ref`

## 令牌凭据

令牌凭据（`type: "token"`）支持内联 `token` 和/或 `tokenRef`。

### 资格规则

1. 当 `token` 和 `tokenRef` 均不存在时，该令牌配置文件不具备资格。
2. `expires` 是可选字段。
3. 如果存在 `expires`，它必须是一个大于 `0` 的有限数值。
4. 如果 `expires` 无效（`NaN`、`0`、负数、非有限数或类型错误），则配置文件不具备资格，原因为 `invalid_expires`。
5. 如果 `expires` 已过期，则配置文件不具备资格，原因为 `expired`。
6. `tokenRef` 不会绕过 `expires` 验证。

### 解析规则

1. 解析器在 `expires` 方面的语义与资格语义保持一致。
2. 对于具备资格的配置文件，令牌材料可从内联值或 `tokenRef` 解析获得。
3. 无法解析的引用会在 `models status --probe` 输出中产生 `unresolved_ref` 错误。

## 向后兼容的消息传递

为了脚本兼容性，探测错误的第一行保持不变：`Auth profile credentials are missing or expired.`。人性化的详细信息和稳定的原因代码会添加在后续行中。

[认证](./gateway/authentication.md)[密钥管理](./gateway/secrets.md)