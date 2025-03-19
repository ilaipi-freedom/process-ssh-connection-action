# Process SSH Connection Action

This action processes SSH connection details based on tag or branch name, with support for environment-specific configurations.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `suffix` | Direct suffix value to use (takes precedence over ref) | false | `''` |
| `ref` | Reference string to process | false | `${{ github.ref }}` |
| `suffix_regex` | Regular expression to extract suffix from ref | false | `'refs/(tags\|heads)/.*-(.+)'` |
| `suffix_group` | Group number in regex that contains the suffix | false | `'2'` |
| `key_prefix` | Prefix to add to the SSH key name (takes precedence over key_prefix_environment) | false | `''` |
| `key_prefix_environment` | Whether to use environment as prefix for the SSH key name (only used if key_prefix is empty) | false | `'false'` |

## Outputs

| Output | Description |
|--------|-------------|
| `ssh_host` | Selected SSH host |
| `ssh_username` | Selected SSH username |
| `ssh_port` | Selected SSH port |
| `environment` | Selected environment |
| `key` | Processed SSH key (output as "key" or "{prefix}_key" or "{environment}_key") |

## Environment Variables Required

For each environment (suffix), you need to set these environment variables:
- `SSH_HOST_{SUFFIX}`
- `SSH_USERNAME_{SUFFIX}`
- `SSH_PORT_{SUFFIX}`
- `SSH_KEY_{SUFFIX}`

Where `{SUFFIX}` is the uppercase version of your environment suffix.

## Usage Examples

### 1. Using direct suffix

```yaml
- uses: your-org/process-ssh-connection-action@v1
  with:
    suffix: 'prod'
```

### 2. Using ref with regex extraction

```yaml
- uses: your-org/process-ssh-connection-action@v1
  with:
    ref: 'refs/tags/release-prod'
    suffix_regex: 'refs/(tags|heads)/.*-(.+)'
    suffix_group: '2'
```

### 3. Using key prefix

```yaml
- uses: your-org/process-ssh-connection-action@v1
  with:
    suffix: 'prod'
    key_prefix: 'myapp'  # Will output as myapp_key
```

### 4. Using environment as key prefix

```yaml
- uses: your-org/process-ssh-connection-action@v1
  with:
    suffix: 'prod'
    key_prefix_environment: 'true'  # Will output as prod_key
```

## Complete Workflow Example

```yaml
name: Deploy
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
          key_prefix: 'myapp'  # Optional: adds prefix to key output
      
      - name: Use SSH Connection
        run: |
          echo "Host: ${{ steps.ssh.outputs.ssh_host }}"
          echo "Username: ${{ steps.ssh.outputs.ssh_username }}"
          echo "Port: ${{ steps.ssh.outputs.ssh_port }}"
          echo "Environment: ${{ steps.ssh.outputs.environment }}"
```