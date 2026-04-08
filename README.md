# microservices-infra

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