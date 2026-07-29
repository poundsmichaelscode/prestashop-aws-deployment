# PrestaShop AWS Deployment Documentation

## Project Overview

This project documents the deployment of **PrestaShop 9.1.4** on Amazon Web Services (AWS), using an **Amazon EC2 instance** as the application server and **Amazon RDS for MySQL** as the managed database service.

The deployment demonstrates how to provision, configure, secure, and connect a PHP-based e-commerce application to a managed relational database in AWS.

The project also covers troubleshooting real-world deployment issues involving:

* EC2 SSH connectivity
* AWS Security Groups
* VPC networking
* Amazon RDS connectivity
* MySQL configuration
* Apache web server configuration
* PHP dependencies
* PrestaShop theme assets
* File transfer failures
* ZIP archive corruption
* Linux file ownership and permissions
* Git and GitHub version control
* Production deployment security

---

# Table of Contents

1. [Project Architecture](#project-architecture)
2. [Technology Stack](#technology-stack)
3. [AWS Infrastructure](#aws-infrastructure)
4. [Application Architecture](#application-architecture)
5. [Prerequisites](#prerequisites)
6. [EC2 Server Configuration](#ec2-server-configuration)
7. [Apache Configuration](#apache-configuration)
8. [PHP Configuration](#php-configuration)
9. [PrestaShop Installation](#prestashop-installation)
10. [Amazon RDS Configuration](#amazon-rds-configuration)
11. [EC2 to RDS Connectivity](#ec2-to-rds-connectivity)
12. [PrestaShop Database Configuration](#prestashop-database-configuration)
13. [Theme Installation](#theme-installation)
14. [File Permissions](#file-permissions)
15. [Security Configuration](#security-configuration)
16. [Git and GitHub Deployment](#git-and-github-deployment)
17. [Challenges and Solutions](#challenges-and-solutions)
18. [Troubleshooting](#troubleshooting)
19. [Deployment Verification](#deployment-verification)
20. [Production Improvements](#production-improvements)
21. [Lessons Learned](#lessons-learned)
22. [Project Links](#project-links)

---

# Project Architecture

The final deployment architecture consists of an AWS EC2 application server connected to an Amazon RDS MySQL database.

```text
                         Internet
                            │
                            │ HTTP / HTTPS
                            ▼
                 ┌──────────────────────┐
                 │      Web Browser     │
                 │      Customer/User   │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │      AWS EC2         │
                 │  prestashop-app-     │
                 │      server          │
                 │                      │
                 │  Ubuntu Linux        │
                 │  Apache 2            │
                 │  PHP                 │
                 │  PrestaShop 9.1.4    │
                 └──────────┬───────────┘
                            │
                            │ MySQL
                            │ TCP Port 3306
                            ▼
                 ┌──────────────────────┐
                 │     Amazon RDS       │
                 │                      │
                 │   MySQL Community    │
                 │      MySQL 8.4       │
                 │                      │
                 │   prestashop-db      │
                 └──────────────────────┘
```

---

# Technology Stack

| Component           | Technology            |
| ------------------- | --------------------- |
| E-commerce Platform | PrestaShop 9.1.4      |
| Operating System    | Ubuntu Linux          |
| Web Server          | Apache 2              |
| Application Runtime | PHP                   |
| Database            | MySQL 8.4             |
| Database Hosting    | Amazon RDS            |
| Application Hosting | Amazon EC2            |
| Networking          | AWS VPC               |
| Firewall            | AWS Security Groups   |
| Version Control     | Git                   |
| Source Code Hosting | GitHub                |
| Deployment Region   | US East (N. Virginia) |

---

# AWS Infrastructure

## EC2 Application Server

The PrestaShop application runs on an Ubuntu EC2 instance.

### EC2 Instance

```text
Instance Name:
prestashop-app-server
```

The application is deployed to:

```text
/var/www/prestashop
```

The EC2 server is responsible for:

* Serving the PrestaShop application
* Running Apache
* Running PHP
* Handling HTTP requests
* Connecting to Amazon RDS
* Serving static assets
* Processing application requests

---

## Amazon RDS Database

The PrestaShop application uses Amazon RDS for managed MySQL database hosting.

### Database Configuration

```text
DB Identifier:
prestashop-db

Engine:
MySQL Community

Engine Version:
MySQL 8.4

Database:
prestashop

Port:
3306
```

### RDS Endpoint

```text
prestashop-db.cwhiymqckumd.us-east-1.rds.amazonaws.com
```

The endpoint should be treated as infrastructure configuration and should not be confused with database credentials.

---

# Application Architecture

The application follows a traditional three-layer architecture:

```text
Presentation Layer
        │
        ▼
PrestaShop Application
        │
        ▼
MySQL Database
```

The EC2 server hosts the application while RDS handles persistent database storage.

The database is not hosted directly on the EC2 server.

This separation provides several benefits:

* Managed database infrastructure
* Automated backups
* Database monitoring
* Independent database scaling
* Reduced database administration overhead
* Better separation of application and data layers

---

# Prerequisites

Before beginning deployment, the following are required:

* AWS account
* AWS EC2 instance
* AWS RDS access
* SSH key pair
* Ubuntu EC2 server
* Local Mac/Linux machine
* Git
* GitHub account
* PrestaShop 9.1.4 package

The following AWS services were used:

* Amazon EC2
* Amazon RDS
* Amazon VPC
* AWS Security Groups

---

# EC2 Server Configuration

After launching the EC2 instance, connect to the server using SSH.

```bash
ssh -i prestashop-key.pem ubuntu@YOUR_EC2_PUBLIC_DNS
```

Example:

```bash
ssh -i prestashop-key.pem ubuntu@ec2-32-197-239-183.compute-1.amazonaws.com
```

The private key must be stored securely on the local machine.

Set the correct permissions:

```bash
chmod 400 prestashop-key.pem
```

Do not upload the private key to GitHub.

---

# Update Ubuntu

Update the package repository:

```bash
sudo apt update
```

Upgrade installed packages:

```bash
sudo apt upgrade -y
```

Restart the server if required:

```bash
sudo reboot
```

Reconnect to the EC2 instance after the restart.

---

# Apache Configuration

Install Apache:

```bash
sudo apt install apache2 -y
```

Check Apache status:

```bash
sudo systemctl status apache2
```

Enable Apache to start automatically:

```bash
sudo systemctl enable apache2
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

Verify the Apache service:

```bash
sudo systemctl status apache2
```

The expected status is:

```text
Active: active (running)
```

---

# PHP Configuration

PrestaShop requires PHP and several PHP extensions.

The required packages installed for the application included:

```bash
sudo apt install php \
libapache2-mod-php \
php-mysql \
php-curl \
php-gd \
php-intl \
php-mbstring \
php-xml \
php-zip \
php-bcmath \
php-soap \
php-opcache \
-y
```

Verify PHP:

```bash
php -v
```

Verify installed PHP modules:

```bash
php -m
```

---

# PrestaShop Installation

The official PrestaShop package was downloaded and transferred to the EC2 server.

The distribution package had the following structure:

```text
prestashop_edition_basic_version_9.1.4-5.0.zip
```

The outer archive contained:

```text
prestashop.zip
index.php
Install_PrestaShop.html
```

The actual PrestaShop application archive was:

```text
prestashop.zip
```

---

# Transferring PrestaShop to EC2

From the local Mac, the archive was transferred using SCP.

```bash
scp -i prestashop-key.pem \
prestashop-extracted/prestashop.zip \
ubuntu@YOUR_EC2_PUBLIC_DNS:/home/ubuntu/
```

The uploaded file can be verified on EC2:

```bash
ls -lh /home/ubuntu/prestashop.zip
```

Check the file type:

```bash
file /home/ubuntu/prestashop.zip
```

The expected output should identify the file as a ZIP archive.

---

# Verifying the ZIP Archive

Before extraction, verify the integrity of the archive:

```bash
unzip -t /home/ubuntu/prestashop.zip
```

A successful validation should end with a message similar to:

```text
No errors detected in compressed data
```

This step is important because an incomplete SCP transfer previously resulted in a corrupted ZIP archive.

---

# Extracting PrestaShop

Create the application directory:

```bash
sudo mkdir -p /var/www/prestashop
```

Extract the archive:

```bash
sudo unzip /home/ubuntu/prestashop.zip -d /var/www/prestashop
```

Verify the application files:

```bash
ls -lah /var/www/prestashop
```

The directory should contain files and directories such as:

```text
app/
classes/
config/
controllers/
img/
js/
modules/
src/
themes/
tools/
translations/
vendor/
webservice/
index.php
composer.json
```

---

# Apache Document Root

The Apache VirtualHost should point to:

```text
/var/www/prestashop
```

A typical configuration is:

```apache
<VirtualHost *:80>
    ServerName YOUR_DOMAIN_OR_EC2_DNS

    DocumentRoot /var/www/prestashop

    <Directory /var/www/prestashop>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/prestashop_error.log
    CustomLog ${APACHE_LOG_DIR}/prestashop_access.log combined
</VirtualHost>
```

Enable required Apache modules:

```bash
sudo a2enmod rewrite
```

Optionally enable:

```bash
sudo a2enmod headers
sudo a2enmod expires
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

Test Apache configuration:

```bash
sudo apache2ctl configtest
```

Expected result:

```text
Syntax OK
```

---

# File Ownership

The Apache web server runs under the `www-data` user.

Set ownership:

```bash
sudo chown -R www-data:www-data /var/www/prestashop
```

Verify:

```bash
ls -lah /var/www/prestashop
```

The files should be owned by:

```text
www-data www-data
```

---

# Amazon RDS Configuration

The RDS database was configured using:

```text
Engine:
MySQL Community

Version:
MySQL 8.4

DB Identifier:
prestashop-db

Instance Class:
db.t4g.micro

Storage:
20 GiB

Port:
3306
```

The database was placed in the same AWS VPC environment as the EC2 application server.

---

# RDS Security Group

The RDS Security Group must allow MySQL traffic from the EC2 application server.

The required inbound rule is:

```text
Type:
MySQL/Aurora

Protocol:
TCP

Port:
3306

Source:
EC2 Application Server Security Group
```

For production environments, avoid opening port `3306` to:

```text
0.0.0.0/0
```

The preferred configuration is to allow only the EC2 application's Security Group.

---

# Testing RDS Connectivity

From the EC2 server, install the MySQL client if necessary:

```bash
sudo apt install mysql-client -y
```

Connect to RDS:

```bash
mysql -h prestashop-db.cwhiymqckumd.us-east-1.rds.amazonaws.com \
-P 3306 \
-u prestashopadmin \
-p
```

Enter the RDS master password when prompted.

A successful connection should display:

```text
mysql>
```

---

# Creating the PrestaShop Database

Once connected to MySQL:

```sql
CREATE DATABASE prestashop
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

Verify the database:

```sql
SHOW DATABASES;
```

The output should include:

```text
prestashop
```

---

# PrestaShop Database Configuration

During the PrestaShop installation wizard, use the following configuration:

```text
Database server address:
prestashop-db.cwhiymqckumd.us-east-1.rds.amazonaws.com

Database name:
prestashop

Database login:
prestashopadmin

Database password:
YOUR_RDS_PASSWORD

Database port:
3306
```

The critical configuration change is:

```text
127.0.0.1
```

must be replaced with the RDS endpoint:

```text
prestashop-db.cwhiymqckumd.us-east-1.rds.amazonaws.com
```

The database server address must not be set to `127.0.0.1` because the database is hosted remotely on Amazon RDS.

---

# PrestaShop Theme Installation

The deployment initially encountered the following error:

```text
The template "assets/js/theme.js" is missing.

The template "assets/css/theme.css" is missing.
```

Investigation showed that the required files were located inside the theme directories.

For the Classic theme:

```text
/var/www/prestashop/themes/classic/assets/js/theme.js
```

```text
/var/www/prestashop/themes/classic/assets/css/theme.css
```

The files were verified with:

```bash
ls -lh /var/www/prestashop/themes/classic/assets/js/theme.js
```

and:

```bash
ls -lh /var/www/prestashop/themes/classic/assets/css/theme.css
```

Expected output confirmed that the files existed.

The issue was resolved by transferring and extracting a complete PrestaShop distribution package containing the correct theme assets.

---

# PrestaShop Installation Completion

After the database connection was configured correctly, the installation proceeded through:

1. Language selection
2. License agreement
3. System compatibility
4. Store information
5. Database configuration
6. Theme selection
7. Demo product configuration
8. Module installation
9. Database population
10. Store installation

The installation completed successfully.

---

# Remove the Installation Directory

For security reasons, the PrestaShop installation directory must be removed after installation.

Run:

```bash
sudo rm -rf /var/www/prestashop/install
```

Verify that it no longer exists:

```bash
ls -lah /var/www/prestashop/install
```

The expected result is:

```text
No such file or directory
```

This step is required before accessing the Back Office.

---

# PrestaShop Environment Configuration

The `.env` file contains application-level environment configuration.

Example:

```env
PS_FF_FRONT_CONTAINER_V2=false

PS_TRUSTED_PROXIES=

PS_FF_DEFAULT_THEME=hummingbird
```

Do not commit sensitive credentials to GitHub.

If database credentials or other secrets are stored in environment variables, use:

```text
.env
```

in `.gitignore`.

---

# Git and GitHub Deployment

The project was version-controlled using Git.

Initialize Git:

```bash
git init
```

Check repository status:

```bash
git status
```

Add project files:

```bash
git add .
```

Create the initial commit:

```bash
git commit -m "Initial PrestaShop AWS deployment"
```

Add the GitHub repository:

```bash
git remote add origin https://github.com/poundsmichaelscode/prestashop-aws-deployment.git
```

Verify the remote:

```bash
git remote -v
```

The output should show:

```text
origin  https://github.com/poundsmichaelscode/prestashop-aws-deployment.git (fetch)
origin  https://github.com/poundsmichaelscode/prestashop-aws-deployment.git (push)
```

Rename the branch to `main`:

```bash
git branch -M main
```

Push the project:

```bash
git push -u origin main
```

---

# Git Security

The following files must never be committed:

```text
.env
*.pem
*.key
```

A recommended `.gitignore` includes:

```gitignore
.env
*.pem
*.key
.DS_Store
vendor/
var/cache/
```

Check for sensitive files before pushing:

```bash
git status
```

Review staged files:

```bash
git diff --cached --name-only
```

If credentials have already been committed, remove them immediately and rotate the exposed credentials.

---

# Challenges and Solutions

## Challenge 1: EC2 SSH Connection Failure

### Problem

The SSH connection initially failed with:

```text
Warning: Identity file prestashop-key.pem not accessible
Permission denied (publickey)
```

### Cause

The SSH private key was not located in the current directory.

### Solution

The key was located on the local Mac and the correct path was used.

Example:

```bash
ssh -i ~/Downloads/prestashop-key.pem \
ubuntu@YOUR_EC2_PUBLIC_DNS
```

The key was also assigned secure permissions:

```bash
chmod 400 ~/Downloads/prestashop-key.pem
```

---

## Challenge 2: RDS PostgreSQL vs MySQL Compatibility

### Problem

The initial database setup involved an Aurora PostgreSQL-compatible database.

### Cause

PrestaShop requires MySQL-compatible database support for this deployment.

### Solution

The database architecture was changed to:

```text
Amazon RDS
    │
    └── MySQL
```

This allowed PrestaShop to connect using the MySQL protocol on port `3306`.

---

## Challenge 3: RDS Connection Timeout

### Problem

The application server could not connect to the database.

### Cause

The database connection was affected by AWS networking and Security Group configuration.

### Solution

The EC2 and RDS networking configuration was reviewed.

The RDS Security Group was configured to allow:

```text
TCP 3306
```

from the EC2 application server's Security Group.

---

## Challenge 4: PrestaShop Theme Assets Missing

### Problem

The installer reported:

```text
theme.js is missing
theme.css is missing
```

### Cause

The initial PrestaShop application files did not contain the expected theme assets in the correct distribution structure.

### Solution

The official PrestaShop distribution was downloaded again and inspected.

The required files were confirmed:

```text
themes/classic/assets/js/theme.js
themes/classic/assets/css/theme.css
```

The complete archive was transferred to EC2 and extracted.

---

## Challenge 5: Incomplete ZIP Transfer

### Problem

The ZIP file transferred to EC2 was only partially uploaded.

The server returned:

```text
End-of-central-directory signature not found.
```

### Cause

The SCP transfer stalled before the complete archive was uploaded.

### Solution

The incomplete archive was removed:

```bash
rm -f /home/ubuntu/prestashop.zip
```

The file was uploaded again.

The archive was then verified:

```bash
unzip -t /home/ubuntu/prestashop.zip
```

The validation completed successfully.

---

## Challenge 6: Apache Permission Issues

### Problem

Apache could not properly serve the application.

### Cause

The application files had incorrect ownership and permissions.

### Solution

Ownership was changed to the Apache web server user:

```bash
sudo chown -R www-data:www-data /var/www/prestashop
```

Apache was restarted:

```bash
sudo systemctl restart apache2
```

The service was verified:

```bash
sudo systemctl status apache2
```

---

## Challenge 7: PrestaShop Security Restriction

### Problem

The PrestaShop Back Office could not be accessed after installation.

### Cause

The `/install` directory still existed.

### Solution

The directory was deleted:

```bash
sudo rm -rf /var/www/prestashop/install
```

The Back Office became available after the security requirement was satisfied.

---

# Troubleshooting

## Check Apache Status

```bash
sudo systemctl status apache2
```

---

## Restart Apache

```bash
sudo systemctl restart apache2
```

---

## Check Apache Configuration

```bash
sudo apache2ctl configtest
```

Expected:

```text
Syntax OK
```

---

## View Apache Error Logs

```bash
sudo tail -f /var/log/apache2/error.log
```

---

## View PrestaShop Logs

Depending on the PrestaShop configuration:

```bash
sudo find /var/www/prestashop/var/logs -type f
```

---

## Check PHP Version

```bash
php -v
```

---

## Check PHP Modules

```bash
php -m
```

---

## Check RDS Connectivity

```bash
mysql -h prestashop-db.cwhiymqckumd.us-east-1.rds.amazonaws.com \
-P 3306 \
-u prestashopadmin \
-p
```

---

## Check Database

After connecting to MySQL:

```sql
SHOW DATABASES;
```

Select the PrestaShop database:

```sql
USE prestashop;
```

Show tables:

```sql
SHOW TABLES;
```

---

## Check PrestaShop Theme Assets

```bash
ls -lh /var/www/prestashop/themes/classic/assets/js/theme.js
```

```bash
ls -lh /var/www/prestashop/themes/classic/assets/css/theme.css
```

---

## Check PrestaShop Files

```bash
ls -lah /var/www/prestashop
```

---

# Deployment Verification

The deployment should be considered successful when the following checks pass:

### EC2

```text
EC2 instance running
```

### Apache

```text
Active: active (running)
```

### PHP

```text
PHP installed and operational
```

### RDS

```text
RDS instance available
```

### Database

```text
prestashop database exists
```

### Network

```text
EC2 → RDS connection successful
```

### Application

```text
PrestaShop installation completed
```

### Theme

```text
theme.js exists
theme.css exists
```

### Security

```text
/install directory deleted
```

### Storefront

```text
Storefront accessible
```

### Back Office

```text
Back Office accessible
```

---

# Live Deployment

The deployed PrestaShop storefront is available at:

```text
http://ec2-32-197-239-183.compute-1.amazonaws.com/
```

The current deployment uses the EC2 public DNS name.

For production, the recommended architecture is:

```text
User
 │
 ▼
Custom Domain
 │
 ▼
HTTPS / SSL
 │
 ▼
Load Balancer or Reverse Proxy
 │
 ▼
EC2
 │
 ▼
Amazon RDS
```

---

# Production Improvements

The current deployment provides a functional AWS-hosted PrestaShop environment.

For a production-ready environment, the following improvements are recommended.

## 1. Custom Domain

Configure a domain using Amazon Route 53 or another DNS provider.

Example:

```text
www.example.com
```

---

## 2. HTTPS

Configure SSL/TLS using AWS Certificate Manager and a suitable AWS load-balancing architecture.

All production traffic should use HTTPS.

---

## 3. Private RDS

The RDS database should not be publicly accessible.

Recommended architecture:

```text
Internet
    │
    ▼
EC2
    │
    │ Private VPC Network
    ▼
RDS
```

---

## 4. Secrets Management

Use AWS Secrets Manager or AWS Systems Manager Parameter Store for sensitive credentials.

Avoid storing passwords in:

```text
.env
GitHub
Source code
README files
```

---

## 5. Monitoring

Configure:

* Amazon CloudWatch
* EC2 monitoring
* RDS monitoring
* Apache logs
* Application logs

---

## 6. Automated Backups

Enable and verify:

* RDS automated backups
* Database snapshots
* EC2 backup strategy

---

## 7. High Availability

For production deployments, consider:

* Multiple EC2 instances
* Application Load Balancer
* Multi-AZ RDS deployment
* Auto Scaling
* CloudFront
* Route 53

---

## 8. CI/CD

The deployment can be automated using:

* GitHub Actions
* AWS CodeDeploy
* AWS CodePipeline

A future pipeline could follow:

```text
Developer
    │
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── Run Tests
    ├── Build Application
    ├── Validate Code
    └── Deploy
            │
            ▼
         AWS EC2
            │
            ▼
        Amazon RDS
```

---

# Lessons Learned

This project provided practical experience in deploying a real-world PHP application to AWS.

Key lessons include:

### 1. Application and database compatibility matter

PrestaShop requires a compatible MySQL database configuration. Selecting the wrong database engine can result in connectivity and compatibility issues.

### 2. AWS networking is critical

The EC2 server and RDS database must be correctly configured within the VPC and Security Groups.

### 3. Security Groups act as virtual firewalls

A database can be running correctly but remain inaccessible if the correct inbound rules are not configured.

### 4. File integrity matters

Large application archives can fail during transfer. Always verify uploaded files with:

```bash
unzip -t archive.zip
```

### 5. Linux permissions affect application behavior

Web applications running under Apache require appropriate ownership and permissions.

### 6. Production security should be considered from the beginning

SSH keys, passwords, environment variables, and database credentials must never be committed to public repositories.

### 7. Managed services simplify infrastructure management

Amazon RDS reduces the operational burden of managing database servers directly on EC2.

---

# Project Links

## Live Store

http://ec2-32-197-239-183.compute-1.amazonaws.com/

## GitHub Repository

https://github.com/poundsmichaelscode/prestashop-aws-deployment

---

# Author

**Olayenikan Michael**

Full-Stack Software Engineer | Cloud & DevOps Engineer

GitHub:

https://github.com/poundsmichaelscode

---

# Final Deployment Summary

This project successfully deployed PrestaShop 9.1.4 on AWS using an EC2-based application server and Amazon RDS for MySQL.

The deployment required troubleshooting several real-world infrastructure and application issues, including database compatibility, network connectivity, SSH access, incomplete file transfers, missing theme assets, Apache configuration, Linux permissions, and PrestaShop installation security.

The completed architecture demonstrates practical experience with:

```text
AWS EC2
    +
Ubuntu Linux
    +
Apache
    +
PHP
    +
PrestaShop
    +
Amazon RDS MySQL
    +
AWS VPC
    +
Security Groups
    +
Git/GitHub
```

The project serves as a practical demonstration of deploying and troubleshooting a production-style web application in a cloud environment.

