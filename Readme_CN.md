# Process SSH Connection Action

这个 GitHub Action 允许您根据分支或标签的后缀，动态选择 SSH 连接配置。当您的 GitHub Secrets 中包含多个 SSH 连接信息时，此 Action 特别有用。

## 作用

此 Action 的主要作用是：

1.  **提取后缀：** 从 `github.ref` 或用户指定的字符串中提取后缀。
2.  **生成环境变量名：** 根据提取的后缀，动态生成 SSH 连接环境变量名，例如 `SSH_HOST_DEV`、`SSH_USERNAME_PROD` 等。
3.  **获取环境变量值：** 从 GitHub Secrets 中获取生成的环境变量的值。
4.  **设置输出：** 将 SSH 连接的详细信息（主机、用户名、端口、密钥）设置到 Action 的输出中，方便后续步骤使用。

## 使用场景

当您的 GitHub Secrets 中包含多个 SSH 连接信息，并且您希望根据分支或标签的后缀，动态选择要使用的 SSH 连接时，可以使用此 Action。

例如，您的 Secrets 中包含以下环境变量：

* `SSH_HOST_DEV`
* `SSH_USERNAME_DEV`
* `SSH_PORT_DEV`
* `SSH_KEY_DEV`
* `SSH_HOST_PROD`
* `SSH_USERNAME_PROD`
* `SSH_PORT_PROD`
* `SSH_KEY_PROD`

当您的分支名称为 `feature-dev` 时，此 Action 将提取 `dev` 后缀，并使用 `SSH_*_DEV` 环境变量中的值。当您的标签名称为 `release-prod` 时，此 Action 将提取 `prod` 后缀，并使用 `SSH_*_PROD` 环境变量中的值。

## 如何使用

1.  **添加 Action 到您的 workflow：**

    ```yaml
    jobs:
      deploy:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout code
            uses: actions/checkout@v3

          - name: Process SSH connection
            id: ssh_connection
            uses: ilaipi-freedom/process-ssh-connection-action@v1 # 替换为您的 action 路径
            with:
              # ref: 'my-custom-ref' # 可选：指定自定义的 ref 字符串
              # suffix_regex: 'your-custom-regex' # 可选：指定自定义的后缀提取正则表达式
              # suffix_group: '1' # 可选：指定正则表达式中的捕获组

          - name: executing remote ssh commands using ssh key
            uses: appleboy/ssh-action@v1.0.0
            with:
              host: ${{ steps.ssh_connection.outputs.ssh_host }}
              username: ${{ steps.ssh_connection.outputs.ssh_username }}
              key: ${{ steps.ssh_connection.outputs.key }}
              port: ${{ steps.ssh_connection.outputs.ssh_port }}
              script: |
                # your remote commands here
                echo "Connected via SSH!"
    ```

2.  **设置 GitHub Secrets：**

    在您的 GitHub 仓库设置中，添加包含 SSH 连接信息的 Secrets，并按照以下格式命名：

    * `SSH_HOST_<SUFFIX>`
    * `SSH_USERNAME_<SUFFIX>`
    * `SSH_PORT_<SUFFIX>`
    * `SSH_KEY_<SUFFIX>`

    其中 `<SUFFIX>` 是分支或标签名称中提取的后缀的大写形式。

## 输入参数

| 参数名         | 描述                                     | 默认值                  | 是否必填 |