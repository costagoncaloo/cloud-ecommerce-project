# Deployment

## Current Deployment Status

The project has been deployed manually to AWS using Terraform and Ansible.

Validated resources:

- EC2 app host in `eu-west-1`
- RDS PostgreSQL in private subnets
- SQS queue and DLQ
- Terraform remote state in S3 with DynamoDB locking
- Dockerized services running on EC2

Always get the current public IP from Terraform:

```bash
terraform -chdir=infra/envs/dev output app_public_ip
```

## Manual Infrastructure Deployment

Initialize Terraform:

```bash
terraform -chdir=infra/envs/dev init
```

Review changes:

```bash
terraform -chdir=infra/envs/dev plan
```

Apply only after reviewing the plan:

```bash
terraform -chdir=infra/envs/dev apply
```

Read outputs:

```bash
terraform -chdir=infra/envs/dev output
```

## Manual Application Deployment With Ansible

Create `ansible/inventory.ini` from the example and set the EC2 public IP and
private key path.

Run:

```bash
ansible-playbook -i ansible/inventory.ini ansible/deploy.yml
```

The playbook:

1. Installs Docker on EC2.
2. Copies the production Docker Compose file.
3. Reuses Terraform-generated environment values.
4. Pulls images.
5. Starts the services.

## GitHub Actions CI

The CI workflow is defined in `.github/workflows/ci.yml`.

It:

- runs Maven tests for all services
- builds Docker images
- publishes images to GitHub Container Registry on non-PR runs

## GitHub Actions Deploy

The deploy workflow is defined in `.github/workflows/deploy.yml`.

It is prepared to:

- assume an AWS role via OIDC
- run Terraform
- configure EC2 with Ansible

Required GitHub secrets:

- `AWS_ROLE_TO_ASSUME`
- `EC2_PRIVATE_KEY`
- `EC2_KEY_NAME`
- `ALLOWED_SSH_CIDR`
- `DB_PASSWORD`

Current caveat: the AWS account must have a GitHub OIDC provider and a deploy
role configured before this workflow can be used as the primary deployment path.

## Destroy After Evaluation

Only after the project has been evaluated:

```bash
terraform -chdir=infra/envs/dev destroy
```
