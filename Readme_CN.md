# SSH 连接处理 Action

这个 Action 可以根据标签或分支名称处理 SSH 连接详情，支持环境特定的配置。

## 输入参数

| 输入参数 | 描述 | 必需 | 默认值 |
|---------|------|------|--------|
| `suffix` | 直接使用的后缀值（优先于 ref） | 否 | `''` |
| `ref` | 要处理的引用字符串 | 否 | `${{ github.ref }}` |
| `suffix_regex` | 从 ref 中提取后缀的正则表达式 | 否 | `'refs/(tags\|heads)/.*-(.+)'` |
| `suffix_group` | 正则表达式中包含后缀的组号 | 否 | `'2'` |
| `key_prefix` | 添加到 SSH key 名称前的前缀（优先于 key_prefix_environment） | 否 | `''` |
| `key_prefix_environment` | 是否使用环境名作为 SSH key 名称的前缀（仅在 key_prefix 为空时使用） | 否 | `'false'` |

## 输出参数

| 输出参数 | 描述 |
|---------|------|
| `ssh_host` | 选定的 SSH 主机 |
| `ssh_username` | 选定的 SSH 用户名 |
| `ssh_port` | 选定的 SSH 端口 |
| `environment` | 选定的环境 |
| `key` | 处理后的 SSH 密钥（输出为 "key" 或 "{prefix}_key" 或 "{environment}_key"） |

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
- uses: your-org/process-ssh-connection-action@v1
  with:
    suffix: 'prod'
```

### 2. 使用正则表达式提取后缀

```yaml
- uses: your-org/process-ssh-connection-action@v1
  with:
    ref: 'refs/tags/release-prod'
    suffix_regex: 'refs/(tags|heads)/.*-(.+)'
    suffix_group: '2'
```

### 3. 使用密钥前缀

```yaml
- uses: your-org/process-ssh-connection-action@v1
  with:
    suffix: 'prod'
    key_prefix: 'myapp'  # 将输出为 myapp_key
```

### 4. 使用环境名作为密钥前缀

```yaml
- uses: your-org/process-ssh-connection-action@v1
  with:
    suffix: 'prod'
    key_prefix_environment: 'true'  # 将输出为 prod_key
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
    env:
      SSH_HOST_PROD: "prod.example.com"
      SSH_USERNAME_PROD: "deploy"
      SSH_PORT_PROD: "22"
      SSH_KEY_PROD: ${{ secrets.SSH_KEY_PROD }}
    steps:
      - uses: your-org/process-ssh-connection-action@v1
        id: ssh
        with:
          key_prefix: 'myapp'  # 可选：添加密钥输出的前缀
      
      - name: 使用 SSH 连接
        run: |
          echo "主机: ${{ steps.ssh.outputs.ssh_host }}"
          echo "用户名: ${{ steps.ssh.outputs.ssh_username }}"
          echo "端口: ${{ steps.ssh.outputs.ssh_port }}"
          echo "环境: ${{ steps.ssh.outputs.environment }}"
```