# Terraform Plan - GitHub Action (Azure)

[![Dependabot](https://badgen.net/badge/Dependabot/enabled/green?icon=dependabot)](https://dependabot.com/)

This GitHub Action runs `terraform plan` with an Azure Remote Backend and uploads the plan as a workflow artifact that can be used with `haflidif/terraform-azure-apply`.

## Inputs

| Name                      | Description                                                                                                      | Required | Type    | Default                |
|---------------------------|------------------------------------------------------------------------------------------------------------------|----------|---------|------------------------|
| `path`                    | Path to the Terraform configuration files. Defaults to the current working directory.                             | No       | string  | `.`                    |
| `artifact_retention_days` | Number of days to retain the plan artifact. Defaults to `5`.                                                      | No       | number  | `5`                    |
| `plan_mode`               | Mode to run the plan in. Options are `deploy` or `destroy`. Defaults to `deploy`.                                 | No       | string  | `deploy`               |
| `tf_version`              | Version of Terraform to use. Defaults to the latest version.                                                      | No       | string  | `latest`               |
| `tf_var_file`             | Variable file to use. Defaults to `terraform.tfvars`.                                                             | No       | string  | `terraform.tfvars`     |
| `tf_state_file`           | State file to use. Defaults to `terraform.tfstate`.                                                               | No       | string  | `terraform.tfstate`    |
| `az_resource_group`       | Name of the Azure Resource Group for the remote backend.                                                          | Yes      | string  |                        |
| `az_storage_account_name` | Name of the Azure Storage Account for the remote backend.                                                         | Yes      | string  |                        |
| `az_storage_container_name`| Name of the Azure Storage Container for the remote backend.                                                      | Yes      | string  |                        |
| `arm_client_id`           | Client ID of the Azure Service Principal or User Assigned Managed Identity for authentication.                    | Yes      | string  |                        |
| `arm_use_oidc`            | Whether to use OIDC for authentication. Defaults to `false`.                                                      | No       | boolean | `false`                |
| `arm_use_azuread`         | Whether to use Azure AD for authentication towards Azure Remote Backend Storage Account and Containers. Requires RBAC permissions `Storage Blob Data Owner` or `Storage Blob Data Contributor`. Defaults to `false`. | No       | boolean | `false`                |
| `arm_client_secret`       | Client secret of the Azure Service Principal for authentication. Required if `arm_client_id` is a Service Principal with Client Secret. | No       | string  | `''`                   |
| `arm_tenant_id`           | Tenant ID of the Azure Service Principal for authentication.                                                      | Yes      | string  |                        |
| `arm_subscription_id`     | Subscription ID of the Azure Service Principal for authentication.                                                | Yes      | string  |                        |
| `github_token`            | GitHub token for posting comments on pull requests. Specify a valid GitHub token.                                 | Yes      | string  |                        |

## Argument Reference

- `path` - Path to the Terraform configuration files. Defaults to the current working directory.
- `artifact_retention_days` - Number of days to retain the plan artifact. Defaults to `5`.
- `plan_mode` - Mode to run the plan in. Options are `deploy` or `destroy`. Defaults to `deploy`.
- `tf_version` - Version of Terraform to use. Defaults to the latest version.
- `tf_var_file` - Variable file to use. Defaults to `terraform.tfvars`.
- `tf_state_file` - State file to use. Defaults to `terraform.tfstate`.
- `az_resource_group` - Name of the Azure Resource Group for the remote backend.
- `az_storage_account_name` - Name of the Azure Storage Account for the remote backend.
- `az_storage_container_name` - Name of the Azure Storage Container for the remote backend.
- `arm_client_id` - Client ID of the Azure Service Principal or User Assigned Managed Identity for authentication.
- `arm_use_oidc` - Whether to use OIDC for authentication. Defaults to `false`.
- `arm_use_azuread` - Whether to use Azure AD for authentication towards Azure Remote Backend Storage Account and Containers. Requires RBAC permissions `Storage Blob Data Owner` or `Storage Blob Data Contributor`. Defaults to `false`.
- `arm_client_secret` - Client secret of the Azure Service Principal for authentication. Required if `arm_client_id` is a Service Principal with Client Secret.
- `arm_tenant_id` - Tenant ID of the Azure Service Principal for authentication.
- `arm_subscription_id` - Subscription ID of the Azure Service Principal for authentication.
- `github_token` - GitHub token for posting comments on pull requests. Specify a valid GitHub token.

## Usage Examples

### Installation

```yaml
name: Terraform Plan (Azure)

on:
  push:
    branches:
      - main

jobs:
  terraform-plan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4.2.0

      - name: Terraform Azure Plan
        uses: haflidif/terraform-azure-plan@v1.0.0
        with:
          path: 'where-your-terraform-files-are-located'
          artifact_retention_days: 5
          plan_mode: 'deploy'
          tf_version: 'latest'
          tf_var_file: 'terraform.tfvars'
          tf_state_file: 'terraform.tfstate'
          az_resource_group: 'your-resource-group'
          az_storage_account_name: 'your-storage-account-name'
          az_storage_container_name: 'your-container-name'
          arm_client_id: 'your-client-id'
          arm_use_oidc: false
          arm_use_azuread: false
          arm_client_secret: 'your-client-secret'
          arm_tenant_id: 'your-tenant-id'
          arm_subscription_id: 'your-subscription-id'
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Usage with User Assigned Managed Identity with Federated Credentials and RBAC permissions

#### Prerequisites
- Create a User Assigned Managed Identity in Azure.
- Create a Federated Credential for the User Assigned Managed Identity. [More information](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation-create-trust-user-assigned-managed-identity?pivots=identity-wif-mi-methods-azp#configure-a-federated-identity-credential-on-a-user-assigned-managed-identity)
- Assign RBAC permissions `Storage Blob Data Owner` or `Storage Blob Data Contributor` on the Storage Account/Container for the User Assigned Managed Identity.

Set the right `jobs` permissions to use the id-token permission in your workflow:

```yaml
permissions:
  id-token: write
  contents: read
```

#### Example

```yaml
name: Terraform Plan (Azure)

on:
  push:
    branches:
      - main

jobs:
  terraform-plan:
    runs-on: ubuntu-latest
    environment: your-environment-name # Optional, if using environment scope in your Federated Credentials.
    permissions:
      id-token: write
      contents: read
      pull-requests: write
      issues: write
      actions: read
    steps:
      - name: Checkout
        uses: actions/checkout@v4.2.0

      - name: Terraform Azure Plan
        uses: haflidif/terraform-azure-plan@v1.0.0
        with:
          az_resource_group: 'your-resource-group'
          az_storage_account_name: 'your-storage-account-name'
          az_storage_container_name: 'your-container-name'
          arm_client_id: 'your-client-id'
          arm_use_oidc: true
          arm_use_azuread: true
```

### Usage with Entra ID Service Principal (Workload Identity) and Secret

```yaml
name: Terraform Plan (Azure)

on:
  push:
    branches:
      - main

jobs:
  terraform-plan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Terraform Azure Plan
        uses: ./
        with:
          az_resource_group: 'your-resource-group'
          az_storage_account_name: 'your-storage-account-name'
          az_storage_container_name: 'your-container-name'
          arm_client_id: 'your-client-id'
          arm_client_secret: ${{ secrets.ARM_CLIENT_SECRET }}
```

## Credits
Inspired and based on GitHub Action by [Marcel Lupo (Pwd9000-ML)](https://github.com/Pwd9000-ML)
- [terraform-azurerm-plan by Marcel Lupo (Pwd9000-ML)](https://github.com/Pwd9000-ML/terraform-azurerm-plan)

## Authors
- Modified and refactored by [Haflidi Fridthjofsson](https://github.com/haflidif)
- Idea, code-samples, and inspiration by [Marcel Lupo / Pwd9000-ML](https://github.com/Pwd9000-ML)

## Disclaimer
The code examples in this repository are provided “as-is” and can be used in production environments at your own responsibility. They come with no warranty of any kind. Use them at your own risk. I am not responsible for any issues, damages, or costs that may arise from using these code examples.

## License
This project is licensed under the GPL-3.0 License - see the [LICENSE](LICENSE) file for details.
