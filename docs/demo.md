# Demo Script

Use this script during the defense. Keep the EC2 public IP dynamic by reading it
from Terraform outputs.

## Local

```bash
make up
make demo-order
make logs
```

Expected result:

- `catalog-service` stores a product.
- `order-service` stores an order and publishes an SQS message.
- `notification-service` logs that it processed the notification.

Useful endpoints:

- `GET http://localhost:8081/products`
- `POST http://localhost:8081/products`
- `GET http://localhost:8082/orders`
- `POST http://localhost:8082/orders`
- `GET http://localhost:8081/actuator/health`
- `GET http://localhost:8082/actuator/health`
- `GET http://localhost:8083/actuator/health`

## AWS

Read the public IP:

```bash
APP_PUBLIC_IP=$(terraform -chdir=infra/envs/dev output -raw app_public_ip)
ORDER_QUEUE_URL=$(terraform -chdir=infra/envs/dev output -raw order_created_queue_url)
echo "$APP_PUBLIC_IP"
```

Health checks:

```bash
curl "http://$APP_PUBLIC_IP:8081/actuator/health"
curl "http://$APP_PUBLIC_IP:8082/actuator/health"
curl "http://$APP_PUBLIC_IP:8083/actuator/health"
```

Create a product:

```bash
curl -X POST "http://$APP_PUBLIC_IP:8081/products" \
  -H "Content-Type: application/json" \
  -d '{"name":"Laptop Demo","description":"Produto criado na AWS","price":999.99,"stock":10}'
```

List products:

```bash
curl "http://$APP_PUBLIC_IP:8081/products"
```

Create an order:

```bash
curl -X POST "http://$APP_PUBLIC_IP:8082/orders" \
  -H "Content-Type: application/json" \
  -d '{"productId":1,"quantity":2}'
```

List orders:

```bash
curl "http://$APP_PUBLIC_IP:8082/orders"
```

Check SQS queue depth:

```bash
aws sqs get-queue-attributes \
  --region eu-west-1 \
  --queue-url "$ORDER_QUEUE_URL" \
  --attribute-names ApproximateNumberOfMessages ApproximateNumberOfMessagesNotVisible RedrivePolicy
```

Check notification logs on EC2:

```bash
ssh ec2-user@"$APP_PUBLIC_IP"
cd /opt/cloud-ecommerce
docker compose logs notification-service
```

Expected log:

```text
Notification processed for order <id> with product <id> and quantity <quantity>
```

## Cleanup After Evaluation

Do not destroy the infrastructure before the defense. After evaluation:

```bash
terraform -chdir=infra/envs/dev destroy
```
