# Limitations

This project intentionally keeps the application and infrastructure small enough
for an academic cloud project.

## Current Limitations

- Services are deployed on a single EC2 instance.
- The EC2 instance is public and exposes demo ports directly.
- There is no Application Load Balancer.
- There is no autoscaling group.
- RDS uses a small single-instance setup.
- Database password is provided through Terraform/EC2 environment generation
  instead of Secrets Manager.
- Centralized logging and CloudWatch dashboards are not implemented.
- GitHub Actions deploy workflow requires OIDC role setup before it can replace
  manual deployment.

## Why This Is Acceptable For The Project

The mandatory requirements focus on demonstrating AWS infrastructure,
containerization, persistence, asynchronous communication, Terraform, Ansible,
CI/CD and IAM concepts. The current architecture covers those requirements while
staying simple enough to explain and defend.

## Possible Future Improvements

- Add an ALB in front of the services.
- Move services to ECS/Fargate.
- Store secrets in AWS Secrets Manager or SSM Parameter Store.
- Add CloudWatch log groups and alarms.
- Add Trivy or ECR/GHCR image scanning.
- Add database migrations with Flyway or Liquibase.
- Add autoscaling for the worker service based on SQS queue depth.
