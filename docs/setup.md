# Setup

## Required Tools

- JDK 21
- Docker Desktop with Docker Compose
- AWS CLI v2
- Terraform `>= 1.9`
- Ansible
- Git

## AWS Account Preparation

1. Enable MFA on the AWS root account.
2. Create an IAM user for daily work and CLI access.
3. Set the default region to `eu-west-1`.
4. Create a budget or billing alert.
5. Avoid using root credentials for daily project work.

Validate the CLI account:

```bash
aws sts get-caller-identity
aws configure get region
```

Expected region:

```text
eu-west-1
```

## Terraform Remote State

Terraform state is stored remotely in S3 and locked with DynamoDB.

Current backend resources:

- S3 bucket: `cloud-ecommerce-tf-state-337058962986-eu-west-1`
- DynamoDB table: `cloud-ecommerce-tf-locks`
- State key: `envs/dev/terraform.tfstate`

The bucket must have:

- versioning enabled
- server-side encryption enabled
- public access blocked

The DynamoDB table must use:

- partition key: `LockID`
- type: `String`
- billing mode: on-demand/pay-per-request

## Local Configuration Files

Create local-only files from examples:

```bash
cp infra/envs/dev/terraform.tfvars.example infra/envs/dev/terraform.tfvars
cp ansible/inventory.ini.example ansible/inventory.ini
```

Do not commit the generated files.

## Local Development

Start the local stack:

```bash
make up
```

Run the demo:

```bash
make demo-order
make logs
```

Stop the local stack:

```bash
make down
```
