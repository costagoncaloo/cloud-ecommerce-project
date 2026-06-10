# Cloud E-Commerce Project

Mini e-commerce backend for the Cloud Information Systems final project.

The business domain is intentionally small. The goal is to demonstrate cloud
engineering practices: AWS infrastructure, Terraform, Docker, RDS, SQS, Ansible,
GitHub Actions, IAM and deployment automation.

## Services

- `catalog-service`: product REST API backed by PostgreSQL.
- `order-service`: order REST API backed by PostgreSQL. Publishes
  `ORDER_CREATED` events to SQS.
- `notification-service`: background worker that consumes SQS events and logs
  notification processing.

## AWS Architecture

- Custom VPC with public and private subnets.
- Public EC2 instance running the Dockerized services.
- Private RDS PostgreSQL instance.
- SQS queue with dead-letter queue for order-created events.
- EC2 IAM role scoped to the SQS actions required by the application.
- Terraform remote state stored in S3 with DynamoDB locking.

See [docs/architecture.md](docs/architecture.md) for the architecture diagram and
request flow.

## Project Documentation

- [Setup](docs/setup.md): local tools, AWS account preparation and Terraform
  backend.
- [Deployment](docs/deployment.md): manual AWS deployment and CI/CD notes.
- [Demo](docs/demo.md): local and AWS demo commands.
- [Security](docs/security.md): IAM, networking, secrets and repository hygiene.
- [Decisions](docs/decisions.md): design choices and trade-offs.
- [Limitations](docs/limitations.md): known limitations and possible
  improvements.
- [Evidence Checklist](docs/evidence.md): screenshots and proof to collect for
  the defense.
- [Final Checklist](docs/final-checklist.md): pre-submission checks.

## Quick Local Run

```bash
make up
make demo-order
make logs
```

Stop the local stack:

```bash
make down
```

## Quick AWS Checks

Get the current EC2 public IP from Terraform:

```bash
terraform -chdir=infra/envs/dev output app_public_ip
```

Run health checks:

```bash
APP_PUBLIC_IP=$(terraform -chdir=infra/envs/dev output -raw app_public_ip)
curl "http://$APP_PUBLIC_IP:8081/actuator/health"
curl "http://$APP_PUBLIC_IP:8082/actuator/health"
curl "http://$APP_PUBLIC_IP:8083/actuator/health"
```

Do not hardcode the EC2 public IP in documentation or scripts. It can change if
the instance is recreated.

## Mandatory Requirements Mapping

| Requirement | Where it is satisfied |
| --- | --- |
| Cloud infrastructure | `infra/envs/dev`, `infra/modules/vpc`, AWS VPC/subnets/security groups |
| Infrastructure as Code | Terraform modules under `infra/modules` |
| Containerization | Dockerfiles in each service and `docker-compose.yml` |
| Distributed architecture | `catalog-service`, `order-service`, `notification-service` |
| Event-driven communication | `order-service -> SQS -> notification-service` |
| Persistence layer | PostgreSQL locally, RDS PostgreSQL in AWS |
| Automation/config management | `ansible/deploy.yml` |
| CI/CD pipeline | `.github/workflows/ci.yml`, `.github/workflows/deploy.yml` |
| Security and permissions | IAM roles, private RDS, security groups, ignored secrets |

## Cleanup After Evaluation

Only after the defense/evaluation, destroy AWS resources to avoid ongoing costs:

```bash
terraform -chdir=infra/envs/dev destroy
```
