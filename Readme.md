# Process SSH Connection Action

This action processes SSH connection details based on tag or branch name. It helps standardize environment variable names for SSH connections.

## Features

- Extract environment suffix from ref or use direct input
- Convert environment-specific variable names to standard outputs
- Automatically process SSH key line breaks
- Support for multiple environments (prod, staging, etc.)

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `suffix` | Direct suffix value to use (takes precedence over ref) | false | `''` |
| `ref` | Reference string to process | false | `${{ github.ref }}` |
| `suffix_regex` | Regular expression to extract suffix from ref | false | `'refs/(tags\|heads)/.*-(.+)'` |
| `suffix_group` | Group number in regex that contains the suffix | false | `'2'` |

## Outputs

| Output | Description |
|--------|-------------|
| `host` | Selected SSH host variable name |
| `username` | Selected SSH username variable name |
| `port` | Selected SSH port variable name |
| `key` | SSH key with processed line breaks |
| `environment` | Selected environment (lowercase suffix) |
| `environment_upper` | Selected environment in uppercase |

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
- uses: ilaipi-freedom/process-ssh-connection-action@v1.0.5
  with:
    suffix: 'prod'
```

### 2. Using ref with regex extraction

```yaml
- uses: ilaipi-freedom/process-ssh-connection-action@v1.0.5
  with:
    ref: 'refs/tags/release-prod'
    suffix_regex: 'refs/(tags|heads)/.*-(.+)'
    suffix_group: '2'
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
    steps:
      - uses: actions/checkout@v3

      # Load environment variables from .env file
      - name: Load Environment Variables
        uses: ilaipi-freedom/load-env-action@v1.0.2

      - name: Process SSH Connection
        id: ssh
        uses: ilaipi-freedom/process-ssh-connection-action@v1.0.5
      
      - name: Use SSH Connection
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ env[steps.ssh.outputs.host] }}
          username: ${{ env[steps.ssh.outputs.username] }}
          port: ${{ env[steps.ssh.outputs.port] }}
          key: ${{ steps.ssh.outputs.key }}
          script: |
            echo "Connected to ${{ steps.ssh.outputs.environment }} environment!"
```

## Notes

1. The action will automatically convert the suffix to uppercase when constructing environment variable names.
2. SSH key line breaks (`\n`) are automatically processed.
3. Make sure all required environment variables are set before using this action.
4. The action outputs both lowercase environment name (`environment`) and uppercase version (`environment_upper`) for flexibility.