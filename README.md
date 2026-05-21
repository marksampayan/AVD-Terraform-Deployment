# AVD Deployment

Terraform deployment for Azure Virtual Desktop using the [avd-terraform-modules](https://github.com/marksampayan/avd-terraform-modules) shared module.

---

## How to use this template for a new deployment

### 1. Create your repo from this template

Click **"Use this template"** → **"Create a new repository"** under your GitHub account.

Clone it locally:
```bash
git clone https://github.com/<your-account>/<your-repo>.git
cd <your-repo>
```

### 2. Pin the module version

In `main.tf`, update the `?ref=` tag to the latest release from [avd-terraform-modules releases](https://github.com/marksampayan/avd-terraform-modules/releases):

```hcl
module "avd" {
  source = "github.com/marksampayan/avd-terraform-modules//modules/avd-core?ref=v1.0.0"
  ...
}
```

### 3. Create your terraform.tfvars

```bash
cp terraform.tfvars.example terraform.tfvars
```

Fill in your deployment-specific values — subscription ID, tenant ID, location, session host count, etc.

**Required values per deployment:**

| Variable | Description |
|----------|-------------|
| `subscription_id` | Target Azure subscription ID |
| `tenant_id` | Azure Entra tenant ID |
| `location` | Azure region ([AVD-supported regions](https://aka.ms/avd-data-locations)) |
| `session_host_count` | Number of VMs (1–50) |
| `avd_users_group_object_id` | Object ID of the Entra group for AVD users |

### 4. Bootstrap Azure + GitHub (one-time per deployment)

```bash
export GITHUB_ORG=<your-github-username-or-org>
export GITHUB_REPO=<your-repo-name>
export TARGET_SUBSCRIPTION_ID=<your-subscription-id>
export TENANT_ID=<your-tenant-id>
export STATE_LOCATION=<azure-region>

chmod +x scripts/bootstrap.sh
./scripts/bootstrap.sh
```

### 5. Configure GitHub repository

#### Secrets
`Settings → Secrets and variables → Actions`

| Secret | Value |
|--------|-------|
| `AZURE_CLIENT_ID` | From bootstrap output |
| `AZURE_TENANT_ID` | Your tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Your subscription ID |
| `TF_VAR_VM_ADMIN_PASSWORD` | Local admin password for session host VMs |
| `TF_VAR_DOMAIN_JOIN_PASSWORD` | Domain join password *(only if using `traditional_dc`)* |

#### Variables
`Settings → Secrets and variables → Actions`

| Variable | Value |
|----------|-------|
| `TF_STATE_RESOURCE_GROUP` | From bootstrap output |
| `TF_STATE_STORAGE_ACCOUNT` | From bootstrap output |
| `TF_STATE_CONTAINER` | `tfstate` |
| `TF_STATE_KEY` | Unique state file name e.g. `avd-eastus.tfstate` |

#### Production environment
`Settings → Environments → New environment → Name: production`
- Enable **Required reviewers** → add yourself or your team

#### Branch protection
`Settings → Branches → Add rule → Branch: main`
- ✅ Require a pull request before merging
- ✅ Require approvals (minimum 1)
- ✅ Require status checks: `Terraform Plan / Terraform Plan`

### 6. Deploy

Push to main (or merge a PR). GitHub Actions handles plan → approval → apply.

---

## Upgrading the module version

To adopt a new module release:

1. Check [avd-terraform-modules releases](https://github.com/marksampayan/avd-terraform-modules/releases) for what changed
2. Update `?ref=` in `main.tf`:
   ```hcl
   source = "github.com/marksampayan/avd-terraform-modules//modules/avd-core?ref=v1.1.0"
   ```
3. Open a PR — the plan will show any infrastructure changes the new version introduces
4. Review the plan, approve, and merge

---

## CI/CD workflows

| Workflow | Trigger | What it does |
|----------|---------|--------------|
| `terraform-plan.yml` | Pull request to `main` | Runs `terraform plan`, posts output as PR comment |
| `terraform-apply.yml` | Merge to `main` | Runs `terraform apply` after production gate approval |
