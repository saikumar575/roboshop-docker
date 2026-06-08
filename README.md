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

## Docker Installation

#!/bin/bash

#check whether root user or not

R="\e[31m"

N="\e[0m"

yum install -y yum-utils

yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

systemctl start docker

systemctl enable docker

usermod -aG docker ec2-user

echo -e "$R Logout and Login again $N"

## RoboShop Docker Deployment Commands

1. Build Frontend Image
   
```
docker build -t frontend:v1 .
```
Explanation:

Builds a Docker image from the Dockerfile in the current directory.
-t frontend:v1 assigns the image name frontend and tag v1.
. means use the current directory as the build context.
2. Create Docker Network
docker network create roboshop

Explanation:

Creates a custom bridge network named roboshop.
Allows containers to communicate using container names instead of IP addresses.
3. Start MongoDB Container
```
docker run -d --name mongodb --network roboshop mongodb:v1
```
Explanation:

Runs MongoDB in detached mode.
Container name: mongodb
Connected to the roboshop network.
Uses image mongodb:v1.
4. Start MySQL Container
```
docker run -d --name mysql --network roboshop mysql:v1
```
Explanation:

Runs MySQL container.
Accessible to other containers as mysql.
5. Start Catalogue Service
```
docker run -d --name catalogue --network roboshop catalogue:v1
```
Explanation:

Runs the Catalogue microservice.
Connected to the roboshop network.
6. Start User Service

```
docker run -d --name user --network roboshop user:v1
```
Explanation:

Runs the User microservice.
Communicates with MongoDB through the Docker network.
7. Start Cart Service

```
docker run -d --name cart --network roboshop cart:v1
```
Explanation:

Runs the Cart microservice.
Connected to the same network as other services.
8. Start Shipping Service
```
docker run -d --name shipping --network roboshop shipping:v1
```
Explanation:

Runs the Shipping microservice.
Uses MySQL for backend data storage.
9. Start Payment Service

```
docker run -d --name payment --network roboshop payment:v1
```
Explanation:

Runs the Payment microservice.
Handles payment processing requests.
10. Start Frontend Service

```
docker run -d --name frontend --network roboshop -p 80:80 frontend:v1
```
Explanation:

Runs the Frontend container.
Maps:
Host Port: 80
Container Port: 80
Makes the application accessible from the browser.
Verify Running Containers

```
docker ps
```
Explanation:

Displays all running containers.
Access Application

```
http://<EC2-Public-IP>
```
Example:

```
http://34.207.138.52
```

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
