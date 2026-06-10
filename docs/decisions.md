# Decisions

## Domain

Use a mini e-commerce backend because it matches the reference architecture and
keeps the business logic simple.

## Compute

Use one EC2 instance with Docker. This keeps the deployment close to the course
labs and lets Ansible demonstrate configuration management.

The instance was sized as `t3.small` with a `30 GB` `gp3` root volume because
three Spring Boot services plus Docker images were too heavy for the smallest
initial EC2 setup.

## Messaging

Use SQS for the order-created event. This gives a clear producer/consumer story
and supports retries and a DLQ.

## Database

Use PostgreSQL. Locally each stateful service has its own database container. In
AWS, the initial version uses one RDS PostgreSQL database shared by the services,
with separate tables and a documented trade-off. A later version could create
separate schemas or databases during deployment.

## Networking

RDS is private because the services are the only clients that need direct
database access. The EC2 instance is public to keep the demo simple and avoid
additional ALB/NAT costs.

## CI/CD

CI builds and publishes Docker images. The deploy workflow is prepared for
Terraform and Ansible, but it requires GitHub-AWS OIDC and repository secrets to
be configured before it can be used as the primary deploy path.
