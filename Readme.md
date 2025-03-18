# Process SSH Connection Action

This GitHub Action allows you to dynamically select SSH connection configurations based on the suffix of a branch or tag. It is particularly useful when your GitHub Secrets contain multiple SSH connection details.

## Purpose

This Action's primary purpose is to:

1.  **Extract Suffix:** Extract the suffix from `github.ref` or a user-specified string.
2.  **Generate Environment Variable Names:** Dynamically generate SSH connection environment variable names based on the extracted suffix, such as `SSH_HOST_DEV`, `SSH_USERNAME_PROD`, etc.
3.  **Retrieve Environment Variable Values:** Retrieve the values of the generated environment variables from GitHub Secrets.
4.  **Set Outputs:** Set the SSH connection details (host, username, port, key) as outputs of the Action for use in subsequent steps.

## Use Cases

Use this Action when your GitHub Secrets contain multiple SSH connection details, and you want to dynamically select the SSH connection to use based on the suffix of a branch or tag.

For example, if your Secrets contain the following environment variables:

* `SSH_HOST_DEV`
* `SSH_USERNAME_DEV`
* `SSH_PORT_DEV`
* `SSH_KEY_DEV`
* `SSH_HOST_PROD`
* `SSH_USERNAME_PROD`
* `SSH_PORT_PROD`
* `SSH_KEY_PROD`

When your branch name is `feature-dev`, this Action will extract the `dev` suffix and use the values from the `SSH_*_DEV` environment variables. When your tag name is `release-prod`, it will extract the `prod` suffix and use the values from the `SSH_*_PROD` environment variables.

## How to Use

1.  **Add the Action to Your Workflow:**

    ```yaml
    jobs:
      deploy:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout code
            uses: actions/checkout@v3

          - name: Process SSH connection
            id: ssh_connection
            uses: ilaipi-freedom/process-ssh-connection-action@v1 # Replace with your Action's path
            with:
              # ref: 'my-custom-ref' # Optional: Specify a custom ref string
              # suffix_regex: 'your-custom-regex' # Optional: Specify a custom regex for suffix extraction
              # suffix_group: '1' # Optional: Specify the capture group in the regex

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

2.  **Set GitHub Secrets:**

    In your GitHub repository settings, add Secrets containing the SSH connection details, named according to the following format:

    * `SSH_HOST_<SUFFIX>`
    * `SSH_USERNAME_<SUFFIX>`
    * `SSH_PORT_<SUFFIX>`
    * `SSH_KEY_<SUFFIX>`

    Where `<SUFFIX>` is the uppercase form of the extracted suffix from the branch or tag name.

## Inputs

| Name           | Description                                                                     | Default                     | Required |
| :------------- | :------------------------------------------------------------------------------ | :-------------------------- | :------- |
| `ref`          | Reference string to process (defaults to `github.ref`)                          | `${{ github.ref }}`         | No       |
| `suffix_regex` | Regular expression to extract the suffix                                        | `refs/(tags\|heads)/.*-(.+)` | No       |
| `suffix_group` | Group number in the regex that contains the suffix                              | `2`                         | No       |

## Outputs

| Name          | Description                  |
| :------------ | :--------------------------- |
| `ssh_host`    | SSH host name                |
| `ssh_username`| SSH username                 |
| `ssh_port`    | SSH port number              |
| `environment` | Extracted environment name   |
| `key`         | SSH private key content      |

## Example

Assuming your branch name is `feature-dev` and your Secrets contain the following environment variables:

* `SSH_HOST_DEV=dev.example.com`
* `SSH_USERNAME_DEV=user`
* `SSH_PORT_DEV=22`
* `SSH_KEY_DEV=-----BEGIN RSA PRIVATE KEY-----...-----END RSA PRIVATE KEY-----`

Then the Action's outputs will be:

* `ssh_host=dev.example.com`
* `ssh_username=user`
* `ssh_port=22`
* `environment=dev`
* `key=-----BEGIN RSA PRIVATE KEY-----...-----END RSA PRIVATE KEY-----`

## Notes

* Ensure your GitHub Secrets contain all the required SSH connection details with the correct naming.
* If your branch or tag names do not conform to the default suffix extraction rules, use the `suffix_regex` input to provide a custom regular expression.