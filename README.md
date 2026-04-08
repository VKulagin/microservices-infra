# microservices-infra

This test assignment demonstrates a simple event-driven microservice architecture built with Symfony 6.4. It consists of three parts: `Product Service`, `Order Service`, and a shared bundle with common DTOs, messages, value objects, and mapped superclasses. `Product Service` manages products and publishes product events to RabbitMQ. `Order Service` maintains a local product snapshot, creates orders, and publishes order completion events. RabbitMQ is used as the integration layer between the services.

The current implementation is suitable as a proof of concept and demonstrates service separation, asynchronous communication, and shared contract reuse. At the same time, in a real production system, several important concerns would need to be addressed more carefully.

First, race conditions should be considered. For example, `Order Service` validates product quantity using its local copy, but the real quantity is updated later in `Product Service`. This creates a risk of stale reads and concurrent overselling. In a production-ready design, this should be handled with reservation logic, stronger consistency rules, or compensating flows.

Second, idempotency should be added both on the API level and on the messaging level. Repeated HTTP requests to create an order should not create duplicates, so `Idempotency-Key` support would be useful. Message consumers should also be idempotent, because RabbitMQ may redeliver messages. This usually requires event IDs and a `processed_messages` store.

Third, logging and observability should be improved. Structured logs, correlation IDs, message IDs, and request tracing would make debugging and monitoring much easier. It would also be useful to add health checks, readiness checks, and basic metrics.

Fourth, exception handling should be more detailed and centralized. Instead of handling errors ad hoc in controllers, a global exception listener or kernel exception subscriber could return consistent JSON error responses with proper status codes and error structures.

In addition, the solution could be improved by adding retry policies, dead-letter queues, validation hardening, automated tests, pagination for list endpoints, and better separation between command and query responsibilities.

So, the implemented solution covers the main functional requirements of the task, but a production-grade version would need stronger consistency guarantees, idempotency, observability, resilience, and more robust error handling.

**Deployment:**

Start microservices-infra with command:

docker compose up -d --build

General testing scenario:

1. Start microservices-infra (this project)
2. Start products (https://github.com/VKulagin/evotym_products)
3. Start orders (https://github.com/VKulagin/evotym_orders)
4. Create a product in products
5. Verify that orders consumes product events and updates its local product snapshot
6. Create an order in orders
7. Verify that products consumes OrderCompletedMessage and updates product quantity
8. Verify that orders receives the updated product quantity back via ProductUpdatedMessage

**RabbitMQ UI**
http://localhost:15672

Default credentials:

guest / guest

**Full smoke test example**

1. Create product


   curl -s -X POST http://localhost:8000/products \
   -H "Content-Type: application/json" \
   -d '{
   "name": "Headphones",
   "price": 59.99,
   "quantity": 15
   }'


    Copy returned id.

2. Create order using product ID


   curl -s -X POST http://localhost:8001/orders \
   -H "Content-Type: application/json" \
   -d '{
   "productId": "PASTE_PRODUCT_ID_HERE",
   "customerName": "Victor",
   "quantityOrdered": 2
   }'


3. Check updated product


   curl -s http://localhost:8000/products/PASTE_PRODUCT_ID_HERE


4. Check all orders


   curl -s http://localhost:8001/orders


   **Negative scenario**

   Try to order more than available


   curl -X POST http://localhost:8001/orders \
   -H "Content-Type: application/json" \
   -d '{
   "productId": "PASTE_PRODUCT_ID_HERE",
   "customerName": "Bob Smith",
   "quantityOrdered": 999
   }'