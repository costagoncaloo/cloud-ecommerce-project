# Security Notes

- AWS credentials are not committed to the repository.
- GitHub Actions deployment should use OIDC through `AWS_ROLE_TO_ASSUME`.
- RDS is placed in private subnets and is not publicly accessible.
- PostgreSQL ingress is restricted to the application security group.
- SSH ingress is restricted by `allowed_ssh_cidr`; use your public IP with `/32`.
- The EC2 instance has an IAM instance role allowing only the SQS actions needed
  by the services.
- Terraform state must use an encrypted S3 bucket and DynamoDB locking.

## Repository Hygiene

The following files must not be committed:

- `infra/envs/dev/terraform.tfvars`
- `.terraform/`
- `*.tfstate`
- `*.pem`
- `*.key`
- `ansible/inventory.ini`
- local `.env` files

The repository keeps example files such as `terraform.tfvars.example`,
`backend.tf.example` and `inventory.ini.example` so the setup can be reproduced
without exposing secrets.

## IAM

- The EC2 instance uses an instance profile instead of static AWS keys.
- The application role is scoped to the SQS actions it needs:
  `SendMessage`, `ReceiveMessage`, `DeleteMessage` and `GetQueueAttributes`.
- GitHub Actions deployment should assume an AWS role through OIDC. If the OIDC
  provider and role are not configured yet, the manual Terraform/Ansible deploy
  remains the source of truth for the current AWS deployment.

## Network Security

- The RDS security group allows PostgreSQL only from the application security
  group.
- The application security group exposes only SSH and the demo service ports.
- SSH should be restricted to the current developer IP through
  `allowed_ssh_cidr`.

## Known Improvements

- Move `db_password` from Terraform variables/user data into AWS Secrets Manager
  or SSM Parameter Store.
- Add image scanning in CI.
- Add CloudWatch log groups and alarms.
- Put an ALB in front of the EC2 instance if the project is evolved beyond the
  academic demo.
