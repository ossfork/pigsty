# OpenTofu Cloud Templates

[OpenTofu](https://opentofu.org/) templates for provisioning cloud VMs to run Pigsty. OpenTofu is the default CLI; the shared `.tf` configurations remain compatible with Terraform 1.x.

Docs: https://pigsty.io/docs/deploy/terraform

Most templates create a single-node `pg-meta` instance; the multi-node Aliyun templates are listed separately below. Common defaults include:
- **OS**: Debian 12 (most providers) or Ubuntu 26.04 (Aliyun templates)
- **Arch**: amd64 (default) or arm64 (where supported)
- **Network**: VPC with `10.10.10.0/24` subnet, private IP `10.10.10.10`
- **Security**: All ports open (demo only - restrict in production!)

The current recommended and verified OS baselines are Rocky Linux 9.8 / 10.2, Debian 12.15 / 13.6, and Ubuntu 22.04.5 / 24.04.4 / 26.04.0.



## Supported Cloud Providers

There are lots of cloud providers out there. Choose one that fits your needs.

| Provider         | Template                                | Instance Type                   | Monthly Cost | ARM Support |
|------------------|-----------------------------------------|---------------------------------|--------------|-------------|
| **Aliyun**       | [aliyun.tf](spec/aliyun.tf)             | ecs.c9i.large / ecs.c8y.large   | ~$20         | Yes         |
| **AWS**          | [aws.tf](spec/aws.tf)                   | t3.medium / t4g.medium          | ~$30         | Yes         |
| **Azure**        | [azure.tf](spec/azure.tf)               | Standard_B2s / Standard_B2ps_v2 | ~$30         | Yes         |
| **GCP**          | [gcp.tf](spec/gcp.tf)                   | e2-medium / t2a-standard-2      | ~$25         | Yes         |
| **Qcloud**       | [qcloud.tf](spec/qcloud.tf)             | S5.MEDIUM4 / SR1.MEDIUM4        | ~$20         | Yes         |
| **Hetzner**      | [hetzner.tf](spec/hetzner.tf)           | cx22 / cax21                    | **~$4.5**    | Yes         |
| **Vultr**        | [vultr.tf](spec/vultr.tf)               | vc2-2c-4gb                      | ~$20         | No          |
| **DigitalOcean** | [digitalocean.tf](spec/digitalocean.tf) | s-2vcpu-4gb                     | ~$24         | No          |
| **Linode**       | [linode.tf](spec/linode.tf)             | g6-standard-2                   | ~$24         | No          |

> **Best Value**: Hetzner offers the best price-performance ratio at ~$4.5/mo for 2 vCPU, 4GB RAM.

> Test & Build environment: pigsty is build with aliyun ECS x86/arm instances.


## Quick Start

### 1. Install OpenTofu

```bash
# macOS
brew install opentofu

# Verify the installed CLI
tofu version
```

For Debian, Ubuntu, RHEL, and other platforms, follow the [official OpenTofu installation guide](https://opentofu.org/docs/intro/install/). The package name is `tofu`; the command is also `tofu`.

### 2. Choose and Configure Template

```bash
cd ~/pigsty/terraform

# Copy your preferred template
cp spec/hetzner.tf terraform.tf      # Best value
# cp spec/aws.tf terraform.tf        # AWS Global
# cp spec/azure.tf terraform.tf      # Microsoft Azure
# cp spec/gcp.tf terraform.tf        # Google Cloud

# Edit variables if needed (distro, region, etc.)
vim terraform.tf
```

### 3. Set Credentials

```bash
# Example for Hetzner
export HCLOUD_TOKEN="your-api-token"

# See "Credentials" section below for other providers
```

### 4. Deploy

```bash
tofu init      # Download provider plugins (first time only)
tofu plan      # Preview changes
tofu apply     # Create resources (type 'yes' to confirm)
```

### 5. Get Server IP

```bash
tofu output meta_ip
# Or
tofu output ssh_command
```

### 6. Install Pigsty

```bash
# SSH into the server
ssh root@<server-ip>

# Install Pigsty
curl -fsSL https://repo.pigsty.io/get | bash
cd ~/pigsty
./bootstrap
./configure
./deploy.yml
```

### 7. Cleanup

```bash
tofu destroy   # Remove all resources (type 'yes' to confirm)
```



## CLI Shortcuts and Terraform Compatibility

The Makefile uses OpenTofu by default and keeps Terraform as an explicit compatibility option:

```bash
cd ~/pigsty/terraform

make init
make validate
make plan
make apply          # Interactive confirmation
make destroy        # Interactive confirmation
make out

# Use Terraform explicitly in the same commands
make IAC_CLI=terraform plan
```

The historical `make u` and `make d` aliases are interactive. Automatic confirmation is available only through the deliberately named `make up-auto`, `make apply-auto`, and `make destroy-auto` targets.

OpenTofu intentionally retains compatibility names such as `.tf`, `terraform {}`, `terraform.tfvars`, `.terraform/`, `.terraform.lock.hcl`, and `terraform.tfstate`. Do not rename these as part of the migration.

Use one CLI per working directory. OpenTofu and Terraform resolve unqualified provider addresses through different registries and may rewrite `.terraform.lock.hcl` during `init`. If you switch engines, back up state and rerun that engine's `init` before planning.



## Configuration Variables

Most templates support these variables:

| Variable       | Description                                               | Default |
|----------------|-----------------------------------------------------------|---------|
| `distro`       | OS distribution (`d12` = Debian 12, `el10` = EL 10, etc.) | `d12` (`u26` on Aliyun templates) |
| `architecture` | CPU architecture (`amd64` or `arm64`)                     | `amd64` |
| `region`       | Cloud region/location                                     | Varies  |
| `zone`         | Availability zone / subnet zone (provider-specific)       | Varies  |

Current Aliyun public base image versions used by the templates:

| Code   | Distro                | amd64 | arm64 |
|--------|-----------------------|-------|-------|
| `el8`  | Rocky Linux 8         | 8.10  | 8.10  |
| `el9`  | Rocky Linux 9         | 9.8   | 9.8   |
| `el10` | Rocky Linux 10        | 10.2  | 10.2  |
| `d12`  | Debian 12             | 12.15 | 12.15 |
| `d13`  | Debian 13             | 13.6  | 13.6  |
| `u22`  | Ubuntu 22.04 LTS      | 22.04.5 | 22.04.5 |
| `u24`  | Ubuntu 24.04 LTS      | 24.04.4 | 24.04.4 |
| `u26`  | Ubuntu 26.04 LTS      | 26.04.0 | 26.04.0 |
| `an8`  | Anolis OS 8           | 8.10  | 8.10  |
| `al3`  | Alibaba Cloud Linux 3 | 3 (20260625 alibase) | 3 (20260625 alibase) |

Aliyun Ubuntu image IDs only include the LTS release (`22_04` / `24_04` / `26_04`), so templates use `most_recent = true` to select the latest point-release image.

Other cloud templates use provider-native rolling image selectors for Debian 12/13:

| Provider                        | Selector style                                                                                     |
|---------------------------------|----------------------------------------------------------------------------------------------------|
| AWS Global                      | Debian official AMI owner with `debian-12-*` / `debian-13-*` name filters and `most_recent = true` |
| Azure                           | Debian Marketplace image references with `version = "latest"`                                      |
| GCP                             | `debian-cloud` image families (`debian-12`, `debian-13`, and ARM64 variants)                       |
| Tencent Cloud                   | Public image lookup by `Debian Server 12` / `Debian Server 13` OS name                             |
| DigitalOcean / Hetzner / Linode | Provider image slugs for Debian 12/13 major releases                                               |
| Vultr                           | Provider OS labels for Debian 12/13 major releases                                                 |

These providers generally do not expose stable point-release image IDs such as `12.15` or `13.6`; templates therefore select the latest published image for the major release. The AWS China legacy template keeps its hardcoded regional AMI because Debian's official AWS AMI catalog does not cover China regions.

Override via command line:
```bash
tofu apply -var="distro=d13" -var="architecture=arm64"
```

Or create a `terraform.tfvars` file:
```hcl
distro       = "d13"
architecture = "arm64"
region       = "us-west-2"
```



## Credentials

### Aliyun

```bash
export ALICLOUD_ACCESS_KEY="<your_access_key>"
export ALICLOUD_SECRET_KEY="<your_secret_key>"
export ALICLOUD_REGION="cn-shanghai"
```

### AWS

```bash
# Option 1: Environment variables
export AWS_ACCESS_KEY_ID="<your_access_key>"
export AWS_SECRET_ACCESS_KEY="<your_secret_key>"
export AWS_REGION="us-west-2"

# Option 2: AWS credentials file (~/.aws/credentials)
[default]
aws_access_key_id = <YOUR_AWS_ACCESS_KEY>
aws_secret_access_key = <AWS_ACCESS_SECRET>
```

### Azure

```bash
# Option 1: Azure CLI (recommended)
az login

# Option 2: Service Principal
export ARM_CLIENT_ID="<your_client_id>"
export ARM_CLIENT_SECRET="<your_client_secret>"
export ARM_SUBSCRIPTION_ID="<your_subscription_id>"
export ARM_TENANT_ID="<your_tenant_id>"
```

### GCP

```bash
# Option 1: gcloud CLI (recommended)
gcloud auth application-default login

# Option 2: Service Account
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"

# Note: GCP requires project ID - set in terraform.tf or:
tofu apply -var="project=your-project-id"
```

### Tencent Cloud

```bash
export TENCENTCLOUD_SECRET_ID="<your_secret_id>"
export TENCENTCLOUD_SECRET_KEY="<your_secret_key>"
```

### Hetzner

```bash
export HCLOUD_TOKEN="<your_api_token>"
```
Get token from: https://console.hetzner.cloud → Project → Security → API tokens

### Vultr

```bash
export VULTR_API_KEY="<your_api_key>"
```
Get API key from: https://my.vultr.com/settings/#settingsapi

### DigitalOcean

```bash
export DIGITALOCEAN_TOKEN="<your_api_token>"
```
Get token from: https://cloud.digitalocean.com/account/api/tokens

### Linode

```bash
export LINODE_TOKEN="<your_api_token>"
```
Get token from: https://cloud.linode.com/profile/tokens



## SSH Key Setup

All templates expect an SSH public key at `~/.ssh/id_rsa.pub`. Generate one if needed:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_rsa -N ''
```

To use a different key, edit the template:
```hcl
public_key = file("~/.ssh/your-key.pub")
```



## Security Warning

**These templates are for demo/development only!**

All security groups/firewalls are configured to allow all traffic from anywhere (`0.0.0.0/0`). For production:

1. Restrict inbound rules to your IP or CIDR blocks
2. Use SSH keys only, disable password authentication
3. Enable cloud provider's security features (WAF, DDoS protection, etc.)
4. Review Pigsty's [security documentation](https://pigsty.io/docs/security/)



## Troubleshooting

### Migrating Existing Terraform State

OpenTofu can read Terraform 1.x state directly. Preserve the state and configuration before initializing OpenTofu:

```bash
cp -p terraform.tfstate "terraform.tfstate.pre-tofu.$(date +%Y%m%d%H%M%S)"
tofu init
tofu plan
```

Continue only when the OpenTofu plan matches the Terraform plan you expected. Do not run both CLIs concurrently against the same state, and never delete `terraform.tfstate` while reinitializing providers.

### SSH Connection Issues

```bash
# Check if server is reachable
ping $(tofu output -raw meta_ip)

# Check SSH with verbose output
ssh -v root@$(tofu output -raw meta_ip)

# Verify SSH key
ssh-add -l
```

### OpenTofu State Issues

```bash
# Review detected state-only changes before accepting them
tofu plan -refresh-only
tofu apply -refresh-only

# Review and force replacement of a degraded resource
tofu plan -replace=<resource_name>
tofu apply -replace=<resource_name>
```

### Provider Plugin Issues

```bash
# Keep recoverable copies and reinitialize
test ! -d .terraform || mv .terraform ".terraform.pre-tofu.$(date +%s)"
test ! -f .terraform.lock.hcl || mv .terraform.lock.hcl ".terraform.lock.hcl.pre-tofu.$(date +%s)"
tofu init
```



## All Templates

### Global Regions
* [spec/aws.tf](spec/aws.tf) - AWS Global regions
* [spec/azure.tf](spec/azure.tf) - Microsoft Azure
* [spec/gcp.tf](spec/gcp.tf) - Google Cloud Platform
* [spec/hetzner.tf](spec/hetzner.tf) - Hetzner Cloud (best value)
* [spec/vultr.tf](spec/vultr.tf) - Vultr
* [spec/digitalocean.tf](spec/digitalocean.tf) - DigitalOcean
* [spec/linode.tf](spec/linode.tf) - Linode/Akamai

### China Regions
* [spec/aliyun.tf](spec/aliyun.tf) - Aliyun single node (all distros, amd64/arm64)
* [spec/aliyun-s3.tf](spec/aliyun-s3.tf) - Aliyun single node with OSS/S3 bucket for PITR
* [spec/aliyun-full.tf](spec/aliyun-full.tf) - Aliyun 4-node sandbox
* [spec/aliyun-pro.tf](spec/aliyun-pro.tf) - Aliyun multi-distro build environment
* [spec/qcloud.tf](spec/qcloud.tf) - Tencent Cloud single node
* [spec/aws-cn.tf](spec/aws-cn.tf) - AWS China region
