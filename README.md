# Test Project

## 安全配置记录

### 问题一：Codex 沙箱模式风险过高

**背景**：Codex CLI 配置文件 `~/.codex/config.toml` 中 `sandbox_mode = "danger-full-access"`，该模式允许 Codex 读写整个系统文件、执行任意命令，风险极高。

**沙箱模式对比**：

| 模式 | 文件系统访问 | 网络访问 | 风险等级 |
|------|-------------|---------|---------|
| `read-only` | 只读当前目录 | 受限 | 低 |
| `workspace-write` | 可写工作目录 | 受限 | 中 |
| `danger-full-access` | 完全读写整个系统 | 完全开放 | 高 |

**解决方案**：将 `sandbox_mode` 修改为 `workspace-write`，限制 Codex 只能读写当前项目目录。

```toml
# ~/.codex/config.toml
sandbox_mode = "workspace-write"

[sandbox]
network_access = true
```

**防护效果**：Codex 无法访问 `~/.ssh`、`.env` 等项目外敏感文件，也无法修改系统文件。

---

### 问题二：npm 恶意包安装防护

**背景**：`network_access = true` 时，`npm install` 可能安装含恶意代码的包，大多数攻击通过依赖包的 `postinstall` 脚本执行。

**解决方案**：配置 npm 全局安全机制。

```bash
npm config set ignore-scripts true   # 禁止依赖包执行脚本
npm config set audit true            # 启用安全审计
npm config set audit-level high      # 只允许 high 级别以下漏洞
```

配置后 `~/.npmrc` 内容：

```ini
ignore-scripts = true
audit = true
audit-level = high
```

**防护效果**：

| 威胁 | 防护状态 |
|------|---------|
| 恶意 npm 脚本执行 | 已阻断（ignore-scripts） |
| 已知漏洞包安装 | 会警告（audit） |

**注意**：`ignore-scripts=true` 可能导致某些需要编译的包（如 `node-sass`）无法正常工作，遇到时可临时执行：

```bash
npm install --ignore-scripts=false
```
