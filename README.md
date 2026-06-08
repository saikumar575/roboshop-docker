# RoboShop

## Project Overview

RoboShop is a containerized e-commerce microservices application deployed using Docker Compose. The solution consists of multiple business services, supporting databases, a message broker, and an Nginx-based frontend.


## Deployment Using Docker Compose

1. Install Docker and Docker Compose.
2. Clone or extract the project.
3. Navigate to the project root.
4. Run: docker compose up -d
5. Verify: docker ps
6. Access the application through http://<server-ip>

## Key Configuration
The docker-compose.yaml file defines service dependencies, environment variables, persistent volumes, and a custom bridge network named 'roboshop'.

## Persistent Volumes
mongodb, redis, mysql, and rabbitmq use Docker volumes for data persistence.

## Architecture Diagram

```text
                    +------------------+
                    |     Frontend     |
                    |      Nginx       |
                    +--------+---------+
                             |
      -------------------------------------------------
      |            |            |           |          |
      v            v            v           v          v
 +---------+  +---------+  +---------+ +---------+ +---------+
 |Catalogue|  |  User   |  |  Cart   | |Shipping | | Payment |
 +----+----+  +----+----+  +----+----+ +----+----+ +----+----+
      |            |            |           |          |
      v            v            |           v          v
 +---------+   +--------+       |      +---------+ +----------+
 | MongoDB |   | Redis  | <-----+      | MySQL   | | RabbitMQ |
 +---------+   +--------+              +---------+ +----------+
```

## Service Dependency Flow

Frontend
 ├── Catalogue → MongoDB
 ├── User → MongoDB + Redis
 ├── Cart → Redis + Catalogue
 ├── Shipping → MySQL + Cart
 └── Payment → Cart + User + RabbitMQ

## Dockerfile Explanations

### Frontend
- Uses Nginx.
- Copies static web content.
- Loads custom nginx.conf.
- Exposes port 80.

### Catalogue/User/Cart
- Node.js based services.
- Install dependencies using package.json.
- Copy application source.
- Start service on port 8080.

### Shipping
- Java/Spring based service.
- Builds application artifact.
- Connects with MySQL backend.

### Payment
- Application service integrating Cart, User and RabbitMQ.
- Uses environment variables for service discovery.

### MongoDB & MySQL
- Custom images initialize schemas and seed data.

## Network Design

Docker Compose creates a custom bridge network:

Network Name: roboshop
Driver: bridge

Benefits:
- Automatic DNS resolution.
- Service-to-service communication.
- Network isolation.

## Volume Design

| Volume | Purpose |
|----------|---------|
| mongodb | MongoDB persistent data |
| redis | Redis persistence |
| mysql | MySQL database files |
| rabbitmq | Queue persistence |

## CI/CD Pipeline

```text
Developer Commit
        |
        v
Git Repository
        |
        v
Jenkins / GitHub Actions
        |
        +--> Build Docker Images
        |
        +--> Run Unit Tests
        |
        +--> Security Scan
        |
        +--> Push Images
        |
        v
Docker Registry
        |
        v
Deployment Server / AWS
        |
        v
docker compose up -d
```

Suggested GitHub Actions stages:
1. Checkout source
2. Build images
3. Test services
4. Push to Docker Hub/ECR
5. Deploy

## AWS Deployment Architecture

```text
                Internet
                    |
             Route53 (Optional)
                    |
              Application Load Balancer
                    |
               EC2 Instance(s)
                    |
            Docker Compose Stack
                    |
 ---------------------------------------------------
 | Frontend | Catalogue | User | Cart | Payment |
 ---------------------------------------------------
                    |
         --------------------------------
         | MongoDB | MySQL | Redis |
         --------------------------------
                    |
                RabbitMQ
```

Recommended AWS Services:
- EC2
- ECR
- ALB
- Route53
- CloudWatch
- IAM
- Secrets Manager

## Deployment

```bash
docker compose up -d
docker ps
```

Access:

http://server-ip

## Troubleshooting

• Check container status: docker ps -a

• View logs: docker logs <container>

• Restart service: docker restart <container>

• Validate compose file: docker compose config

## Conclusion
This project demonstrates deployment of a microservices-based e-commerce platform using Docker containers, service dis
