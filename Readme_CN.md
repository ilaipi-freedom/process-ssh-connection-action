# SSH 连接处理 Action

这个 Action 可以根据标签或分支名称处理 SSH 连接详情。它帮助标准化 SSH 连接的环境变量名称。

## 功能特点

- 从 ref 中提取环境后缀或使用直接输入
- 将环境特定的变量名转换为标准输出
- 自动处理 SSH 密钥的换行符
- 支持多环境（prod、staging 等）

## 输入参数

| 参数 | 描述 | 必需 | 默认值 |
|-----|------|------|--------|
| `suffix` | 直接使用的后缀值（优先于 ref） | 否 | `''` |
| `ref` | 要处理的引用字符串 | 否 | `${{ github.ref }}` |
| `suffix_regex` | 从 ref 中提取后缀的正则表达式 | 否 | `'refs/(tags\|heads)/.*-(.+)'` |
| `suffix_group` | 正则表达式中包含后缀的组号 | 否 | `'2'` |

## 输出参数

| 输出 | 描述 |
|------|------|
| `host` | 选定的 SSH 主机变量名 |
| `username` | 选定的 SSH 用户名变量名 |
| `port` | 选定的 SSH 端口变量名 |
| `key` | 处理过换行符的 SSH 密钥 |
| `environment` | 选定的环境（小写后缀） |
| `environment_upper` | 选定的环境（大写后缀） |

## 所需的环境变量

对于每个环境（后缀），你需要设置以下环境变量：
- `SSH_HOST_{SUFFIX}`
- `SSH_USERNAME_{SUFFIX}`
- `SSH_PORT_{SUFFIX}`
- `SSH_KEY_{SUFFIX}`

其中 `{SUFFIX}` 是你的环境后缀的大写版本。

## 使用示例

### 1. 使用直接后缀

```yaml
- uses: ilaipi-freedom/process-ssh-connection-action@v1.0.5
  with:
    suffix: 'prod'
```

### 2. 使用正则表达式提取后缀

```yaml
- uses: ilaipi-freedom/process-ssh-connection-action@v1.0.5
  with:
    ref: 'refs/tags/release-prod'
    suffix_regex: 'refs/(tags|heads)/.*-(.+)'
    suffix_group: '2'
```

## 完整工作流示例

```yaml
name: 部署
on:
  push:
    tags:
      - '*-prod'
      - '*-staging'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # 从 .env 文件加载环境变量
      - name: 加载环境变量
        uses: ilaipi-freedom/load-env-action@v1.0.2

      - name: 处理 SSH 连接
        id: ssh
        uses: ilaipi-freedom/process-ssh-connection-action@v1.0.5
      
      - name: 使用 SSH 连接
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ env[steps.ssh.outputs.host] }}
          username: ${{ env[steps.ssh.outputs.username] }}
          port: ${{ env[steps.ssh.outputs.port] }}
          key: ${{ steps.ssh.outputs.key }}
          script: |
            echo "已连接到 ${{ steps.ssh.outputs.environment }} 环境！"
```

## 注意事项

1. Action 会自动将后缀转换为大写形式来构建环境变量名。
2. SSH 密钥的换行符（`\n`）会被自动处理。
3. 使用此 Action 前请确保所有必需的环境变量都已设置。
4. Action 同时输出环境名称的小写版本（`environment`）和大写版本（`environment_upper`）以提供更大的灵活性。