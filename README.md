# Online Bookstore Services

## Global Tech Stack
These are elements of the tech stack that will be used in most of the services.

### AWS ECS Fargate

I chose Fargate because it removes a layer of management from the original ECS/EKS service. I also chose it because of the built-in security, container recovery, logging, and multi-az deployments.

### Github Actions

Github Actions has a growing community and a growing community of templates. It is already integrated into Github which makes a good solution.

### Redis Caching/Redis Cloud Enterprise

The order processing, book catalog, and user management services will need caching. While API Gateway caching was tempting we need more control on the cache keys and TTLs.  

### Application Load Balancer

The Application Load Balancer (ALB) offers several advantages, including independent health monitoring for each service, support for containerized applications, and seamless integration with Amazon Elastic Container Service (Amazon ECS). With ALB, Amazon ECS can dynamically select an available port while scheduling a task and subsequently register the task with the corresponding target group using the chosen port.

### Docker/Docker Compose

It is important to emphasize the utilization of Docker during the development phase, as the deployment will be carried out using ECS Fargate.

### PostgreSQL/Aurora

PostgreSQL is a mature database with a great community. 
Aurora is a solid choice that offers built-in security and Multi-AZ support out of the box.
I chose PostgreSQL for the user management, order processing, and book catalog because these services are more relational.

### Node.js/Express

Node.js is non-blocking and event-driven which made it a good choice for the user management, shopping cart, and book catalog services. 
There is also a decent pool of Node.js/Javascript engineers on the market.

## User Management Service

This service will be responsible for managing user data, including user registration, login, and profile management. 
For the purposes of this project, I kept this service simple and implemented a basic user registration and login flow.

### From Global Stack
- Github Actions
- Docker/Docker Compose
- PostgreSQL
- ECS Fargate
- Node.js/Express
- Application Load Balancer
### Tech Stack

#### AWS SES

We will need to send a large amount of emails and SES is a scalable service that does the behind the scenes work to ensure our emails are delivered. 

#### AWS SQS

Instead of sending emails directly through SES, you can send email requests to an SQS queue. 
This approach allows you to decouple the email sending process from the rest of your application. 
The application can then add email requests to the SQS queue, and another part of the application or a separate worker 
can handle the actual email sending from the queue. This way, if the email sending process fails or experiences issues, 
the messages in the queue can be retried or processed by another worker.

#### AWS Lambda

I chose Lambda to send the emails based on the SQS queue.

## Book Catalog Service

This service will manage the book inventory, including adding, updating, and deleting book details. 

### From Global Stack
- Github Actions
- Docker/Docker Compose
- PostgreSQL
- ECS Fargate
- Node.js/Express
- Application Load Balancer

### Tech Stack

#### ElasticSearch/OpenSearch

For users to find books you will need a full-text search engine that can also filter as well, and one that has proven scalability.

#### Amazon S3

S3 is the perfect place to store book images, publisher images, and potentially author photos.

#### CloudFront CDN

We need a CDN to serve the images.

## Order Processing Service

This service will manage order placement, payment processing, and order tracking.

### From Global Stack

- ECS Fargate
- Github Actions
- PostgreSQL/Aurora
- Docker/Docker Compose
- Application Load Balancer

### Tech Stack

#### Java/Spring Boot

It offers a balance of speed, reliability, and ease of development, making it a suitable choice for this critical 
component of the online bookstore. Like Javascript, there is a good pool of Java engineers on the market.


#### AWS SES

This service will need to send emails based on order updates.

#### AWS SQS

I selected the same option based on the identical reasons that were considered during the user management service.

#### AWS Lambda

I selected the same option based on the identical reasons that were considered during the user management service.

#### Stripe

Stripe is a proven leader in the world in payment processing. Their webhooks and Stripe Radar offerings will be utilized.
Also Stripe Connect could be useful in the future if the bookstore morphs into a marketplace.

#### Stripe Webhooks

This is the most scalable way to ingest the Stripe data needed to process the orders.

## Shopping Cart Service

This service will handle the shopping cart functionality, including adding items to the cart, updating quantities, and removing items. This also will be my most controversial plan because of Redis.

### From Global Stack
- Node.js/Express
- Docker/Docker Compose
- Github Actions
- ECS Fargate
- Application Load Balancer

### Tech Stack

#### Redis/Redis Cloud Enterprise

I chose Redis/Redis Cloud Enterprise because it is fast, has persistent storage options, it is easy to expire the data, and I don't view this data highly relational. The key for the cart is just the User ID or the Session ID.


## Communication Between Services

### User Management 
#### Data From Shopping Cart
None 
#### Data From Order Processing
None
#### Data From Book Catalog
None
#### Data To Shopping Cart
User ID via a client and JWT for the creation of the cart.

#### Data To Order Processing
User ID via a client and JWT for order creation.
User lookup via the /users/{userId} endpoint.

#### Data to BookCatalog
User ID and privileges via a client and JWT for logging who did what in the CRUD operations.  

### Shopping Cart
#### Data From User Management
User ID via JWT when creating the shopping cart
#### Data From Order Processing
Call via DELETE /delete endpoint when order is complete 
#### Data From Book Catalog
Book Data via GET /book endpoint
#### Data to User Management
None
#### Data to Order Processing
Cart Info via GET /cart endpoint
#### Data to Book Catalog
None

### Order Processsing

#### Data From User Management
User lookup via the /users/{userId} endpoint.
#### Data From Shopping Cart
Cart details via GET /cart endpoint
#### Data From Book Catalog
Book details via /book/{bookID} endpoint
#### Data to User Management
None
#### Data to Shopping Cart
Call delete after order is complete
#### Data to Book Catalog
Call to API to increase the decrease the stock on hand

### Book Catalog
#### Data From User Management
Who is completing the CRUD operations.

#### Data From Shopping Cart
None
#### Data From Order Processing
Update quantity of books sold, returned, or newly stocked.

#### Data To User Management
None
#### Data To Shopping Cart
Book Details
#### Data To Order Processing
Book details

## Diagram

You can view the diagram on [Figma](https://www.figma.com/file/mV286yDjWScHIi2uGZuF0F/Untitled?type=whiteboard&node-id=2%3A666&t=at3bW0szZK0iVLJE-1)
## Docs

I create an OpenAPI spec for the user management service. It is located in the docs folder.