# Terraform Plan GitHub Action (Azure)

[![Dependabot](https://badgen.net/badge/Dependabot/enabled/green?icon=dependabot)](https://dependabot.com/)

This GitHub Action runs `terraform plan` with an Azure Remote Backend and uploads the plan as a workflow artifact that can be used with `haflidif/terraform-azure-apply`

## Inputs

| Name                      | Description                                                                                                      | Required | Type    | Default                |
|---------------------------|------------------------------------------------------------------------------------------------------------------|----------|---------|------------------------|
| `path`                    | (Optional) Path to the Terraform configuration files, defaults to the current working directory                  | No       | string  | `.`                    |
| `artifact_retention_days` | (Optional) The number of days to retain the plan artifact. Defaults to `5`.                                       | No       | number  | `5`                    |
| `plan_mode`               | (Optional) The mode to run the plan in. Valid options are `deploy` or `destroy`. Defaults to `deploy`.           | No       | string  | `deploy`               |
| `tf_version`              | (Optional) The version of Terraform to use. Defaults to the latest version.                                       | No       | string  | `latest`               |
| `tf_var_file`             | (Optional) The variable file to use. Defaults to `terraform.tfvars`.                                              | No       | string  | `terraform.tfvars`     |
| `tf_state_file`           | (Optional) The state file to use. Defaults to `terraform.tfstate`.                                                | No       | string  | `terraform.tfstate`    |
| `az_resource_group`       | (Required) The name of the Azure Resource Group to use for the remote backend.                                    | Yes      | string  |                        |
| `az_storage_account_name` | (Required) The name of the Azure Storage Account to use for the remote backend.                                   | Yes      | string  |                        |
| `az_storage_container_name`| (Required) The name of the Azure Storage Container to use for the remote backend.                                | Yes      | string  |                        |
| `az_storage_key`          | (Required) The key to use for the remote backend.                                                                 | Yes      | string  |                        |
| `arm_client_id`           | (Required) The client ID of the `Azure Service Principal` or `User Assigned Managed Identity` to use for authentication. | Yes      | string  |                        |
| `arm_use_oidc`            | (Optional) Whether to use OIDC for authentication. Defaults to `false`.                                           | No       | boolean | `false`                |
| `arm_use_azuread`         | (Optional) Whether to use Azure AD for authentication towards Azure Remote Backend Storage Account and Containers. Requires RBAC permissions `Storage Blob Data Owner` or `Storage Blob Data Contributor` on the Storage Account/Container for the identity being used. Defaults to `false`. | No       | boolean | `false`                |
| `arm_client_secret`       | (Conditionally required) The client secret of the Azure Service Principal to use for authentication. Required if `arm_client_id` is a Service Principal with Client Secret. | No       | string  | `''`                   |
| `arm_tenant_id`           | (Required) The tenant ID of the Azure Service Principal to use for authentication.                                | Yes      | string  |                        |
| `arm_subscription_id`     | (Required) The subscription ID of the Azure Service Principal to use for authentication.                          | Yes      | string  |                        |
| `github_token`            | (Optional) The GitHub token to use for posting comments on pull requests. Defaults to `${{ github.token }}`.      | No       | string  | `${{ github.token }}`  |

## Argument Reference
The following arguments are supported:

- `path` - (Optional) Path to the Terraform configuration files, defaults to the current working directory.
- `artifact_retention_days` - (Optional) The number of days to retain the plan artifact. Defaults to `5`.
- `plan_mode` - (Optional) The mode to run the plan in. Valid options are `deploy` or `destroy`. Defaults to `deploy`.
- `tf_version` - (Optional) The version of Terraform to use. Defaults to the latest version.
- `tf_var_file` - (Optional) The variable file to use. Defaults to `terraform.tfvars`.
- `tf_state_file` - (Optional) The state file to use. Defaults to `terraform.tfstate`.
- `az_resource_group` - (Required) The name of the Azure Resource Group to use for the remote backend.
- `az_storage_account_name` - (Required) The name of the Azure Storage Account to use for the remote backend.
- `az_storage_container_name` - (Required) The name of the Azure Storage Container to use for the remote backend.
- `az_storage_key` - (Required) The key to use for the remote backend.
- `arm_client_id` - (Required) The client ID of the `Azure Service Principal` or `User Assigned Managed Identity` to use for authentication.
- `arm_use_oidc` - (Optional) Whether to use OIDC for authentication. Defaults to `false`.
- `arm_use_azuread` - (Optional) Whether to use Azure AD for authentication towards Azure Remote Backend Storage Account and Containers. Requires RBAC permissions `Storage Blob Data Owner` or `Storage Blob Data Contributor` on the Storage Account/Container for the identity being used. Defaults to `false`.
- `arm_client_secret` - (Conditionally required) The client secret of the Azure Service Principal to use for authentication. Required if `arm_client_id` is a Service Principal with Client Secret, otherwise the workflow will fail.
- `arm_tenant_id` - (Required) The tenant ID of the Azure Service Principal to use for authentication.
- `arm_subscription_id` - (Required) The subscription ID of the Azure Service Principal to use for authentication.
- `github_token` - (Optional) The GitHub token to use for posting comments on pull requests. Defaults to `${{ github.token }}`.

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
          az_storage_key: 'your-storage-key'
          arm_client_id: 'your-client-id'
          arm_use_oidc: false
          arm_use_azuread: false
          arm_client_secret: 'your-client-secret'
          arm_tenant_id: 'your-tenant-id'
          arm_subscription_id: 'your-subscription-id'
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Usage with User Assigned Managed Identity with Federated Credentials and RBAC permissions.

#### Prerequisites
- Create a User Assigned Managed Identity in Azure.
- Create a Federated Credential for the User Assigned Managed Identity - See here for more information: [Configure a federated identity credential on a user-assigned managed identity](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation-create-trust-user-assigned-managed-identity?pivots=identity-wif-mi-methods-azp#configure-a-federated-identity-credential-on-a-user-assigned-managed-identity)
- Assign RBAC permissions `Storage Blob Data Owner` or `Storage Blob Data Contributor` on the Storage Account/Container for the User Assigned Managed Identity.

Remember to set the right `jobs` permissions to use the id-token permission in your workflow. 

Example:
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
    environment: your-environment-name # Optional, if your are using environment scope in your Federated Credentials.
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
          az_storage_key: 'your-storage-key'
          arm_client_id: 'your-client-id'
          arm_use_oidc: true
          arm_use_azuread: true
```

### Usage with Entra ID Service Principal (Workload Identity) and Secret.

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
          az_storage_key: 'your-storage-key'
          arm_client_id: 'your-client-id'
          arm_client_secret: ${{ secrets.ARM_CLIENT_SECRET }}
```