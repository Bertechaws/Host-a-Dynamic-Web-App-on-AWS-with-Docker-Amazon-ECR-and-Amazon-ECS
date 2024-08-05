# Host-a-Dynamic-Web-App-on-AWS-with-Docker-Amazon-ECR-and-Amazon-ECS

![IMG_2142](https://github.com/user-attachments/assets/79d37222-c9a4-4cfb-9569-1be9e43160e2)

# Dynamic Web App Deployment on AWS with Docker, Amazon ECR, and Amazon ECS
This project demonstrates how to deploy a dynamic web application on AWS using Docker, Amazon ECR (Elastic Container Registry), and Amazon ECS (Elastic Container Service). The deployment is achieved using a 3-tier architecture with public and private subnets in two availability zones, ensuring high availability and fault tolerance.


Tools & Technologies
Docker: Containerization of the application.
Git: Version control for source code.
GitHub: Repository hosting for Dockerfile and application code.
AWS CLI: Command-line interface for managing AWS services.
Flyway: Database schema migration tool.
Visual Studio Code: IDE for script development.
Amazon ECR: Docker image storage.
Amazon ECS: Container orchestration on AWS.
VPC: Virtual network setup.
Amazon RDS: Relational database service.
ECS Fargate: Serverless compute for containers.
Application Load Balancer: Load distribution.
Auto Scaling: Dynamic scaling of ECS tasks.
Route 53: Domain name registration and DNS management.
AWS S3: Storage for environment variables.
IAM Roles: Permissions management for ECS tasks.
Bastion Host: Secure SSH access.
Security Groups: Network access control.
Certificate Manager: SSL/TLS certificate management.
---

## Prerequisites

To get started with this project, ensure you have the following tools installed:

- **Git**: [Download Git](https://git-scm.com/downloads)
- **Visual Studio Code**: [Download Visual Studio Code](https://code.visualstudio.com/)
- **Docker**: [Install Docker](https://docs.docker.com/get-docker/)
- **AWS CLI**: [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- **Flyway**: [Download Flyway](https://flywaydb.org/download)

### Setup Instructions

1. **Sign up for a Docker Hub account** and enable virtualization on your computer.
2. **Clone this repository**:
   ```bash
   git clone https://github.com/your_github_username/your_repository_name.git
   ```
3. **Create and configure Dockerfile** based on the instructions below.

---

## Part 1: Building a 3-Tier AWS Network VPC
![IMG_2144](https://github.com/user-attachments/assets/8598bc82-8782-40d6-a263-9dbed3938c61)


1. **Create a VPC** with public and private subnets in two Availability Zones for high availability and fault tolerance.
2. **Set up an Internet Gateway** to allow communication between the VPC and the internet.
3. **Deploy resources** such as NAT Gateway, Bastion Host, and Application Load Balancer in the public subnets.
4. **Deploy web servers and database servers** in private subnets for enhanced security.
5. **Configure route tables**:
   - Public route table for public subnets, routing traffic through the Internet Gateway.
   - Main route table for private subnets.

---

## Part 2: Deployment Steps

### AWS Setup

1. **Create NAT Gateways**.
2. **Set Up Security Groups** for network access control.
3. **Set Up a MySQL RDS Instance** for the database.
4. **Register a domain** in Amazon Route 53.
5. **Create a GitHub repository** for your application code and Dockerfile.
6. **Generate a Personal Access Token** for GitHub.

### Docker Setup

1. **Create Dockerfile** for the dynamic web app:
   ```Dockerfile
   # Use the latest version of the Amazon Linux base image
   FROM amazonlinux:2

   # Update all installed packages to their latest versions
   RUN yum update -y 

   # Install necessary packages
   RUN yum install -y unzip wget httpd git

   # Install PHP and MySQL
   RUN amazon-linux-extras enable php7.4 && yum install -y \
     php php-common php-pear php-cgi php-curl php-mbstring php-gd php-mysqlnd \
     php-gettext php-json php-xml php-fpm php-intl php-zip mysql-community-server

   # Configure MySQL
   RUN wget https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm && \
       rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022 && \
       yum localinstall mysql80-community-release-el7-3.noarch.rpm -y && \
       yum install mysql-community-server -y

   # Clone and prepare the application
   WORKDIR /var/www/html
   ARG PERSONAL_ACCESS_TOKEN
   ARG GITHUB_USERNAME
   ARG REPOSITORY_NAME
   ARG WEB_FILE_ZIP
   ARG WEB_FILE_UNZIP
   ARG DOMAIN_NAME
   ARG RDS_ENDPOINT
   ARG RDS_DB_NAME
   ARG RDS_MASTER_USERNAME
   ARG RDS_DB_PASSWORD
   ENV PERSONAL_ACCESS_TOKEN=$PERSONAL_ACCESS_TOKEN \
       GITHUB_USERNAME=$GITHUB_USERNAME \
       REPOSITORY_NAME=$REPOSITORY_NAME \
       WEB_FILE_ZIP=$WEB_FILE_ZIP \
       WEB_FILE_UNZIP=$WEB_FILE_UNZIP \
       DOMAIN_NAME=$DOMAIN_NAME \
       RDS_ENDPOINT=$RDS_ENDPOINT \
       RDS_DB_NAME=$RDS_DB_NAME \
       RDS_MASTER_USERNAME=$RDS_MASTER_USERNAME \
       RDS_DB_PASSWORD=$RDS_DB_PASSWORD

   RUN git clone https://$PERSONAL_ACCESS_TOKEN@github.com/$GITHUB_USERNAME/$REPOSITORY_NAME.git && \
       unzip $REPOSITORY_NAME/$WEB_FILE_ZIP -d $REPOSITORY_NAME/ && \
       cp -av $REPOSITORY_NAME/$WEB_FILE_UNZIP/. /var/www/html && \
       rm -rf $REPOSITORY_NAME && \
       sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf && \
       chmod -R 777 /var/www/html && \
       chmod -R 777 storage/ && \
       sed -i '/^APP_ENV=/ s/=.*$/=production/' .env && \
       sed -i "/^APP_URL=/ s/=.*$/=https:\/\/$DOMAIN_NAME\//" .env && \
       sed -i "/^DB_HOST=/ s/=.*$/=$RDS_ENDPOINT/" .env && \
       sed -i "/^DB_DATABASE=/ s/=.*$/=$RDS_DB_NAME/" .env && \
       sed -i "/^DB_USERNAME=/ s/=.*$/=$RDS_MASTER_USERNAME/" .env && \
       sed -i "/^DB_PASSWORD=/ s/=.*$/=$RDS_DB_PASSWORD/" .env

   COPY AppServiceProvider.php app/Providers/AppServiceProvider.php
   EXPOSE 80 3306
   ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]
   ```

2. **Build Docker Image**:
   ```bash
   docker build --build-arg PERSONAL_ACCESS_TOKEN= \
                 --build-arg GITHUB_USERNAME= \
                 --build-arg REPOSITORY_NAME= \
                 --build-arg WEB_FILE_ZIP= \
                 --build-arg WEB_FILE_UNZIP= \
                 --build-arg DOMAIN_NAME= \
                 --build-arg RDS_ENDPOINT= \
                 --build-arg RDS_DB_NAME= \
                 --build-arg RDS_MASTER_USERNAME= \
                 --build-arg RDS_DB_PASSWORD= \
                 -t <image-tag> .
   ```

3. **Push Docker Image to Amazon ECR**:
   ```bash
   aws ecr create-repository --repository-name <repository-name> --region <region>
   docker tag <image-tag> <repository-uri>
   aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
   docker push <repository-uri>
   ```

### Additional AWS Configuration

1. **Create a Key Pair** for EC2 access.
2. **Set Up a Bastion Host** with EC2.
3. **Configure Flyway** for database migrations:
   ```ini
   flyway.url=jdbc:mysql://localhost:3306/
   flyway.user=
   flyway.password=
   flyway.locations=filesystem:sql
   flyway.cleanDisabled=false
   ```
4. **Create an Application Load Balancer** and register an SSL certificate with Amazon Certificate Manager.
5. **Create an HTTPS Listener** for the Load Balancer.
6. **Store Environment Variables** in an S3 bucket and create an IAM role for ECS tasks.

---

## Deployment Scripts

### Docker Build Script
```bash
#!/bin/bash
docker build --build-arg PERSONAL_ACCESS_TOKEN=<your_token> \
              --build-arg GITHUB_USERNAME=<your_username> \
              --build-arg REPOSITORY_NAME=<your_repo> \
              --build-arg WEB_FILE_ZIP=<your_zip> \
              --build-arg WEB_FILE_UNZIP=<your_unzip> \
              --build-arg DOMAIN_NAME=<your_domain> \
              --build-arg RDS_ENDPOINT=<rds_endpoint> \
              --build-arg RDS_DB_NAME=<rds_db_name> \
              --build-arg RDS_MASTER_USERNAME=<rds_username> \
              --build-arg RDS_DB_PASSWORD=<rds_password> \
              -t <image-tag> .
```

### AWS CLI Commands
```bash
# Create ECR repository
aws ecr create-repository --repository-name <repository-name> --region <region>

# Login to ECR and push Docker image
aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
docker tag <image-tag> <repository-uri>
docker push <repository-uri>
```

---

## Additional Resources

- [AWS Documentation](https://docs.aws.amazon.com/)
- [Docker Documentation](https://docs.docker.com/)
- [GitHub Documentation](https://docs.github.com/en)
- [Flyway Documentation](https://flywaydb.org/documentation/)

Feel free to customize the scripts and configurations based on your specific requirements. This README serves as a comprehensive guide to showcase your skills and the process followed in this DevOps project.
