### API Gateway in Practice with Spring Cloud Gateway

API Gateways are crucial elements in microservices-based architectures, offering a single entry point for requests and managing various functionalities such as routing, security, caching, and monitoring. This article explores how to set up and test an API Gateway using Spring Cloud Gateway.

------

### What is an API Gateway and Its Importance

An API Gateway is responsible for:

- **Managing requests**: Directing them to the appropriate microservice.
- **Adding security**: Implementing authentication and authorization.
- **Improving performance**: Applying caching and response compression.
- **Monitoring and auditing**: Logging and metrics recording.

Imagine an e-commerce system where the API Gateway manages requests for payment, inventory, and user profile services, ensuring that each call is routed correctly and securely.

------

### Creating an API Gateway with Spring Cloud Gateway

#### 1. Project Setup

To get started, create a new Spring project with the required dependencies:

```bash
curl https://start.spring.io/starter.zip \
-d dependencies=cloud-gateway-reactive,webflux,actuator \
-d name=spring-cloud-gateway-demo \
-d type=maven-project \
-d language=java \
-d packaging=jar \
-d javaVersion=17 \
-o spring-cloud-gateway-demo.zip

unzip spring-cloud-gateway-demo.zip -d spring-cloud-gateway && cd spring-cloud-gateway

./mvnw spring-boot:run
```

#### 2. Testing the Initial Setup

Use the Actuator endpoint to verify that the application is running:

```bash
http GET localhost:8080/actuator/health
```

The response should be:

```json
{
  "status": "UP"
}
```

------

### Simulating Services with Mockoon

#### Installing Mockoon

Mockoon is a tool for creating fake APIs (mocks):

```bash
sudo npm install -g @mockoon/cli
```

#### Configuring the Mocks

Save the following content as `mockoon-demo.json`:

```json
{
  "lastMigration": 33,
  "name": "Demo API",
  "routes": [
    {
      "method": "get",
      "responses": [
        {
          "body": "{\"service\": \"Demo API\", \"baseUrl\": \"{{baseUrl}}\"}",
          "statusCode": 200,
          "headers": [{"key": "Content-Type", "value": "application/json"}]
        }
      ]
    }
  ]
}
```

Start three instances:

```bash
mockoon-cli start -d mockoon-demo.json -p 8081
mockoon-cli start -d mockoon-demo.json -p 8082
mockoon-cli start -d mockoon-demo.json -p 8083
```

Testing the mocks:

```bash
http GET localhost:8081
http GET localhost:8082
http GET localhost:8083
```

------

### Configuring Routes in the API Gateway

In the project's `application.yml` file, configure the routes:

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: service1
        uri: http://localhost:8081
        predicates:
        - Path=/service1/**
        filters:
        - RewritePath=/service1, /
      - id: service2
        uri: http://localhost:8082
        predicates:
        - Path=/service2/**
        filters:
        - RewritePath=/service2, /
      - id: service3
        uri: http://localhost:8083
        predicates:
        - Path=/service3/**
        filters:
        - RewritePath=/service3, /
```

This configuration:

1. **Identifies the routes**: Each route has a unique `id`.
2. **Defines the destination**: Specifies the URI of the corresponding microservice.
3. **Applies predicates**: Checks the request path before routing it.
4. **Applies filters**: Rewrites the path to adjust the request to the destination service.

------

### Testing the Configured Routes

Run the tests to verify functionality:

```bash
http GET localhost:8080/service1
http GET localhost:8080/service2
http GET localhost:8080/service3
```

The responses should redirect to the configured mocks. For example:

```json
{
  "baseUrl": "http://localhost:8081",
  "service": "Demo API"
}
```

------

### Conclusion and Next Steps

With this basic setup, you can explore additional functionalities of Spring Cloud Gateway, such as:

- **Security**: Integration with JWT for authentication.
- **Observability**: Monitoring with tools like Prometheus and Grafana.
- **Resilience**: Implementing circuit breakers with Resilience4j.

Want to learn more or share your experience? Leave a comment or send a message!
