# AWS VPC Terraform Module

Terraform module to provision an Amazon Web Services (AWS) Virtual Private Cloud (VPC) with optional subnets, route tables, internet gateways, NAT gateways, VPN gateways, and VPC endpoints.

## Permissions

To provision the AWS resources managed by this module, the IAM role or user running Terraform needs permissions such as:

- `AmazonVPCFullAccess` (or fine-grained privileges to manage VPCs, Subnets, Route Tables, Internet/NAT Gateways, DHCP Options, and VPC Peering Connections).
- Additional permissions are required if enabling database, Redshift, or ElastiCache subnet groups (e.g., `AmazonRDSFullAccess`, `AmazonRedshiftFullAccess`, `AmazonElastiCacheFullAccess` or granular equivalent actions).
- Permissions to manage IAM Roles and IAM Policies if `vpc_flow_logs` or certain VPC Endpoints require role creation.

## Authentications

Authentication to AWS can be configured using one of the following methods, with preference given to OIDC and dynamic provider credentials in CI/CD environments.

### HCP Terraform / Terraform Enterprise Dynamic Credentials (OIDC)

Use dynamic provider credentials via OpenID Connect (OIDC) for secure, short-lived credentials when running in HCP Terraform or Terraform Enterprise.

- **Using environment variables (HCP Terraform Workspace)**

  - `TFC_AWS_PROVIDER_AUTH=true`
  - `TFC_AWS_RUN_ROLE_ARN=<aws-iam-role-arn>`

### OIDC with GitHub Actions

When using GitHub Actions, configure OIDC via the `aws-actions/configure-aws-credentials` action.

- **Using GitHub Actions**

  ```yaml
  - name: Configure AWS credentials
    uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::111122223333:role/github-actions-role
      aws-region: us-east-1
  ```

### Static Access Keys

For local development or environments not supporting OIDC, use static IAM programmatic access keys.

- **Inside the provider block**

  ```hcl
  provider "aws" {
    region     = "us-east-1"
    access_key = "<aws-access-key-id>"
    secret_key = "<aws-secret-access-key>"
  }
  ```

- **Using environment variables**

  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
  - `AWS_DEFAULT_REGION` (optional)

Documentation:

- [AWS Provider Authentication](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication)
- [Dynamic Provider Credentials in HCP Terraform](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/dynamic-provider-credentials/aws-configuration)

## Features

- Complete foundational VPC networking setup (VPC, CIDR associations, default security groups).
- Highly configurable subnets (Public, Private, Database, ElastiCache, Redshift, Intra, Outpost).
- Internet Gateways, Egress Only Internet Gateways, NAT Gateways (single or multi-AZ).
- VPN Gateways, Customer Gateways, and route propagations.
- Integrations for VPC Endpoints and centralized VPC Flow Logs.

## Usage example

```hcl
module "vpc" {
  source  = "app.terraform.io/benoitblais-hashicorp/vpc/aws"
  version = "5.21.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true
  enable_vpn_gateway = false

  tags = {
    Terraform   = "true"
    Environment = "prod"
  }
}
```
