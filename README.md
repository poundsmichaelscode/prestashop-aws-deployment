# PrestaShop AWS Cloud Deployment

A production-style deployment of **PrestaShop 9.1.4** on **Amazon Web Services (AWS)** using an Amazon EC2 application server and Amazon RDS for managed MySQL database hosting.

This project demonstrates the deployment of a PHP-based e-commerce application on AWS, including Linux server administration, Apache configuration, PHP setup, database provisioning, networking, security groups, application troubleshooting, Git version control, and cloud infrastructure configuration.

---

## Project Overview

The goal of this project was to deploy a fully functional PrestaShop e-commerce platform on AWS.

The application is hosted on an **Amazon EC2 Ubuntu server**, while the PrestaShop database is hosted on **Amazon RDS for MySQL**.

The final architecture separates the application and database layers, creating a basic two-tier cloud architecture.

### Architecture

```text
                         Internet
                            │
                            │ HTTP / HTTPS
                            ▼
                  ┌─────────────────────┐
                  │     Amazon EC2      │
                  │   Ubuntu Server     │
                  │                     │
                  │ Apache HTTP Server  │
                  │ PHP 8.x             │
                  │ PrestaShop 9.1.4    │
                  └──────────┬──────────┘
                             │
                             │ MySQL : 3306
                             │
                             ▼
                  ┌─────────────────────┐
                  │     Amazon RDS       │
                  │    MySQL 8.4.9      │
                  │                     │
                  │ Database: prestashop│
                  └─────────────────────┘
```

The source code is managed using Git and hosted on GitHub.

```text
Developer Machine
       │
       │ git push
       ▼
    GitHub
       │
       │ git clone / git pull
       ▼
    EC2 Server
       │
       ▼
   PrestaShop
       │
       ▼
   Amazon RDS
```

---

## Live Infrastructure

### Application Server

* **Platform:** Amazon EC2
* **Operating System:** Ubuntu Linux
* **Web Server:** Apache 2
* **Application:** PrestaShop 9.1.4
* **Application Directory:**

```text
/var/www/prestashop
```

### Database Server

* **Platform:** Amazon RDS
* **Engine:** MySQL Community
* **Version:** MySQL 8.4.9
* **Database Name:**

```text
prestashop
```

* **Port:**

```text
3306
```

* **RDS Endpoint:**

```text
prestashop-db.cwhiymqckumd.us-east-1.rds.amazonaws.com
```

> The database password and other sensitive credentials are intentionally not included in this repository.

---

# Technologies Used

## Cloud Infrastructure

* Amazon EC2
* Amazon RDS
* Amazon VPC
* AWS Security Groups
* AWS IAM
* AWS CloudShell

## Server Administration

* Ubuntu Linux
* Apache HTTP Server
* PHP
* MySQL Client
* SSH
* SCP
* Linux file permissions

## Application

* PrestaShop 9.1.4
* PHP
* MySQL
* Composer
* Apache

## Version Control

* Git
* GitHub

---

# Project Structure

The main PrestaShop application is located at:

```text
/var/www/prestashop
```

Important directories include:

```text
prestashop/
├── admin-dev/
├── app/
├── cache/
├── classes/
├── config/
├── controllers/
├── img/
├── install-dev/
├── js/
├── modules/
├── override/
├── src/
├── themes/
├── tools/
├── translations/
├── upload/
├── var/
├── vendor/
├── webservice/
├── .env
├── composer.json
├── composer.lock
├── index.php
└── robots.txt
```

The `themes` directory contains the PrestaShop themes used by the application.

For example:

```text
themes/
├── classic/
└── hummingbird/
```

The Classic theme contains the required assets:

```text
themes/classic/assets/js/theme.js
themes/classic/assets/css/theme.css
```

---

# AWS Infrastructure Configuration

## EC2

The EC2 instance serves as the application server.

The server runs:

* Ubuntu
* Apache
* PHP
* PrestaShop

The application is served from:

```text
/var/www/prestashop
```

Apache was configured to serve this directory as the website document root.

---

## Amazon RDS

An Amazon RDS MySQL database was created for PrestaShop.

### Database configuration

```text
Engine: MySQL Community
Version: MySQL 8.4.9
Database: prestashop
Port: 3306
```

The database was created using:

```sql
CREATE DATABASE prestashop
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

The database uses `utf8mb4` to support a wide range of characters and languages.

---

# Database Connection

During the PrestaShop installation, the database configuration was set to use the Amazon RDS endpoint rather than the local MySQL server.

The configuration concept is:

```text
Database server address:
prestashop-db.cwhiymqckumd.us-east-1.rds.amazonaws.com

Database name:
prestashop

Database port:
3306

Database login:
prestashopadmin
```

The actual database password should never be committed to GitHub.

---

# PrestaShop Installation

The PrestaShop installation wizard was accessed through the EC2-hosted website.

The installation process included:

1. Selecting the installation language.
2. Accepting the license agreement.
3. Checking system compatibility.
4. Configuring store information.
5. Configuring database connectivity.
6. Selecting the PrestaShop theme.
7. Installing modules.
8. Installing demo products.
9. Creating the database tables.
10. Completing the store installation.

The installation successfully completed after resolving the theme asset issue.

---

# Major Challenge: Missing Theme Assets

One of the most significant deployment issues encountered was the following PrestaShop installation error:

```text
An error occurred during installation.

The template "assets/js/theme.js" is missing.
The template "assets/css/theme.css" is missing.
```

Initially, the expected files could not be found at:

```text
/var/www/prestashop/assets/js/theme.js
/var/www/prestashop/assets/css/theme.css
```

The issue was investigated by examining the PrestaShop installation archive.

The required assets were found under the theme directory:

```text
themes/classic/assets/js/theme.js
themes/classic/assets/css/theme.css
```

The correct files were verified using:

```bash
ls -lh /var/www/prestashop/themes/classic/assets/js/theme.js
```

and:

```bash
ls -lh /var/www/prestashop/themes/classic/assets/css/theme.css
```

The files were successfully detected:

```text
theme.js
theme.css
```

The issue was ultimately caused by deploying an incomplete or corrupted PrestaShop archive rather than the complete application package.

A clean PrestaShop package was obtained, extracted, and transferred to the EC2 instance.

---

# Challenge: Corrupted or Incomplete ZIP Transfer

During deployment, the PrestaShop ZIP archive was transferred from the local Mac computer to the EC2 server using SCP.

An incomplete transfer resulted in an archive that appeared to be a ZIP file but could not be extracted.

The server returned:

```text
End-of-central-directory signature not found.
```

The issue was caused by the file being only partially transferred.

The incomplete archive was removed:

```bash
rm -f /home/ubuntu/prestashop.zip
```

The file was then transferred again.

The integrity of the archive was verified using:

```bash
unzip -t /home/ubuntu/prestashop.zip
```

The successful result included:

```text
No errors detected in compressed data
```

The application was then extracted into:

```text
/var/www/prestashop
```

---

# Challenge: Choosing the Correct PrestaShop Package

The initial downloaded package contained a wrapper archive:

```text
prestashop_edition_basic_version_9.1.4-5.0.zip
```

The archive contained:

```text
prestashop.zip
index.php
Install_PrestaShop.html
```

The actual PrestaShop application was located inside:

```text
prestashop.zip
```

The inner archive was extracted and inspected to verify that it contained the required theme assets.

The following files were confirmed to exist:

```text
themes/classic/assets/js/theme.js
themes/classic/assets/css/theme.css
```

This helped identify that the original deployment package on the EC2 server was incomplete.

---

# Challenge: Incorrect Database Configuration

Initially, the PrestaShop installer was configured with:

```text
Database server:
127.0.0.1

Database login:
root
```

However, the production database was hosted on Amazon RDS.

The configuration was changed to use the RDS endpoint:

```text
prestashop-db.cwhiymqckumd.us-east-1.rds.amazonaws.com
```

The database name was:

```text
prestashop
```

The database port was:

```text
3306
```

This allowed PrestaShop to communicate with the remote RDS MySQL database.

---

# Challenge: PostgreSQL vs MySQL Compatibility

An earlier database configuration attempt involved an AWS Aurora/PostgreSQL-compatible database.

However, PrestaShop's database requirements and the existing deployment architecture were based on MySQL-compatible database functionality.

This created a compatibility issue.

The solution was to provision:

```text
Amazon RDS
    │
    └── MySQL Community
         │
         └── MySQL 8.4.9
```

This provided a compatible database backend for PrestaShop.

---

# Challenge: EC2 and RDS Networking

The application server and database server needed to communicate securely.

The RDS instance was configured with a VPC security group:

```text
rds-ec2-1
```

The database was configured as:

```text
Publicly accessible: No
```

The database therefore remained private to the VPC.

The intended architecture is:

```text
Internet
   │
   ▼
EC2
   │
   │ TCP 3306
   ▼
RDS MySQL
```

The database should only accept MySQL connections from the EC2 application's security group.

---

# Challenge: SSH Key Management

SSH access to the EC2 instance required the correct private key.

An attempted connection failed with:

```text
Warning: Identity file prestashop-key.pem not accessible:
No such file or directory.
```

The issue was caused by the private key not being available in the current Mac directory.

The correct approach is to keep the `.pem` file securely on the local computer and use its correct path when connecting.

Example:

```bash
chmod 400 /path/to/prestashop-key.pem
```

Then:

```bash
ssh -i /path/to/prestashop-key.pem \
ubuntu@EC2_PUBLIC_DNS
```

The private key should never be committed to GitHub.

---

# File Permissions

After deploying the application, ownership was configured for Apache:

```bash
sudo chown -R www-data:www-data /var/www/prestashop
```

The application files were then verified.

For example:

```bash
ls -lh /var/www/prestashop/themes/classic/assets/js/theme.js
```

and:

```bash
ls -lh /var/www/prestashop/themes/classic/assets/css/theme.css
```

The files were owned by:

```text
www-data:www-data
```

This ensured that the Apache web server could access the application files.

---

# Apache Configuration

Apache was used as the web server for PrestaShop.

The application document root was configured as:

```text
/var/www/prestashop
```

After making configuration changes, Apache was restarted:

```bash
sudo systemctl restart apache2
```

The Apache service was verified using:

```bash
sudo systemctl status apache2
```

The expected status was:

```text
Active: active (running)
```

---

# Git and GitHub

The PrestaShop project is managed using Git.

The GitHub repository is:

[PrestaShop AWS Deployment Repository](https://github.com/poundsmichaelscode/prestashop-aws-deployment.git?utm_source=chatgpt.com)

The Git remote is configured as:

```bash
git remote add origin \
https://github.com/poundsmichaelscode/prestashop-aws-deployment.git
```

The main branch is:

```text
main
```

The project can be pushed using:

```bash
git push -u origin main
```

---

# Recommended Git Workflow

Development is performed locally and changes are pushed to GitHub.

```text
Local Mac
    │
    │ git add
    │ git commit
    │ git push
    ▼
GitHub
    │
    │ git pull
    ▼
AWS EC2
    │
    ▼
Production Application
```

Typical workflow:

```bash
git status

git add .

git commit -m "Update PrestaShop deployment"

git push origin main
```

On the EC2 server:

```bash
git pull origin main
```

---

# Environment Variables

Sensitive configuration should not be stored in the Git repository.

The production environment file should remain on the EC2 server.

Example:

```text
.env
```

The `.env` file should be added to `.gitignore`:

```gitignore
.env
.env.*
!.env.example
```

An example environment file can be committed:

```text
.env.example
```

The example file should contain placeholders instead of real credentials.

Example:

```env
PS_FF_FRONT_CONTAINER_V2=false
PS_TRUSTED_PROXIES=
PS_FF_DEFAULT_THEME=hummingbird

DB_HOST=your-rds-endpoint
DB_PORT=3306
DB_NAME=prestashop
DB_USER=your-database-user
DB_PASSWORD=your-database-password
```

> Never commit production database passwords, AWS credentials, private SSH keys, or API secrets.

---

# Security Considerations

The following security practices should be implemented before considering the deployment production-ready:

* Keep RDS private where possible.
* Restrict RDS port `3306` to the EC2 security group.
* Restrict SSH port `22` to trusted IP addresses.
* Use HTTPS with an SSL/TLS certificate.
* Never commit `.env` files containing secrets.
* Never commit `.pem` private keys.
* Use AWS Secrets Manager for production secrets.
* Use strong database passwords.
* Keep Ubuntu and server packages updated.
* Enable automated RDS backups.
* Configure monitoring and alerts.
* Enable HTTPS redirects.
* Disable or remove the PrestaShop installer directory after installation.
* Use least-privilege IAM permissions.

---

# Post-Installation Security

After successfully installing PrestaShop, the installer directory must be removed or disabled.

PrestaShop's installation interface requires the installation directory to be removed for security purposes.

Before removing anything, verify the installation has completed successfully.

The installation directory should then be removed according to the PrestaShop installation instructions.

---

# Troubleshooting Commands

### Check Apache

```bash
sudo systemctl status apache2
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

Check Apache configuration:

```bash
sudo apache2ctl configtest
```

---

### Check Apache logs

```bash
sudo tail -f /var/log/apache2/error.log
```

Access logs:

```bash
sudo tail -f /var/log/apache2/access.log
```

---

### Check PrestaShop files

```bash
ls -lah /var/www/prestashop
```

Check theme assets:

```bash
ls -lah /var/www/prestashop/themes/classic/assets/js/
```

```bash
ls -lah /var/www/prestashop/themes/classic/assets/css/
```

---

### Check file ownership

```bash
ls -ld /var/www/prestashop
```

Change ownership:

```bash
sudo chown -R www-data:www-data /var/www/prestashop
```

---

### Check database connectivity

From the EC2 server:

```bash
mysql \
-h prestashop-db.cwhiymqckumd.us-east-1.rds.amazonaws.com \
-P 3306 \
-u prestashopadmin \
-p
```

After entering the password, select the database:

```sql
USE prestashop;
```

Check the tables:

```sql
SHOW TABLES;
```

---

# Deployment Challenges Summary

Throughout the deployment process, several real-world infrastructure and application challenges were encountered and resolved.

### 1. EC2 SSH connection issues

Resolved SSH authentication and private-key path issues.

### 2. SCP file transfer interruption

A large PrestaShop archive was partially transferred, resulting in a corrupted ZIP file.

The solution was to remove the incomplete file, re-transfer the archive, and verify its integrity.

### 3. Incorrect PrestaShop package structure

The downloaded PrestaShop package contained an inner `prestashop.zip` archive.

The inner archive was extracted and verified before deployment.

### 4. Missing theme assets

PrestaShop installation initially failed because:

```text
themes/classic/assets/js/theme.js
themes/classic/assets/css/theme.css
```

were missing from the deployed application.

The issue was resolved by deploying a complete PrestaShop package.

### 5. Database engine mismatch

An earlier Aurora PostgreSQL-compatible database setup was not appropriate for the PrestaShop deployment.

The solution was to provision Amazon RDS for MySQL.

### 6. Database connectivity

The application was configured to communicate with Amazon RDS using the RDS endpoint and MySQL port `3306`.

### 7. Linux permissions

The application files were assigned to the Apache web server user:

```text
www-data:www-data
```

### 8. Apache configuration

Apache was configured and restarted to serve the PrestaShop application.

### 9. Git and GitHub integration

The project was initialized with Git and connected to a GitHub repository for version control and deployment management.

---

# Key Lessons Learned

This project provided practical experience with:

* AWS EC2 provisioning.
* Amazon RDS database deployment.
* AWS VPC networking.
* Security group configuration.
* Linux server administration.
* SSH authentication.
* SCP file transfers.
* Apache web server configuration.
* PHP application deployment.
* MySQL database management.
* Database connectivity troubleshooting.
* PrestaShop installation and configuration.
* Application file permissions.
* Git and GitHub workflows.
* Production environment configuration.
* Troubleshooting corrupted deployment artifacts.
* Diagnosing application errors using logs and filesystem inspection.

The project also demonstrated the importance of validating deployment artifacts before deploying them to production.

For example:

```bash
file application.zip
```

and:

```bash
unzip -t application.zip
```

can be used to verify that an archive is valid before attempting to deploy it.

---

# Future Improvements

The next phase of this project can include:

* Configure a custom domain.
* Enable HTTPS with AWS Certificate Manager or Let's Encrypt.
* Add an Application Load Balancer.
* Implement Auto Scaling.
* Move secrets to AWS Secrets Manager.
* Add Amazon CloudWatch monitoring.
* Configure automated backups.
* Implement CI/CD using GitHub Actions.
* Add deployment automation.
* Use AWS Systems Manager instead of direct SSH where appropriate.
* Implement infrastructure as code using Terraform or AWS CloudFormation.
* Add a staging environment.
* Configure RDS Multi-AZ for high availability.
* Add CloudFront for content delivery.
* Use Amazon S3 for media and static asset storage where appropriate.
* Implement automated security scanning.

---

# Project Outcome

The project successfully demonstrated the deployment of a PrestaShop e-commerce application on AWS using a separated application and database architecture.

The final solution consists of:

```text
PrestaShop 9.1.4
        │
        ▼
Apache + PHP
        │
        ▼
Ubuntu EC2
        │
        │ MySQL 3306
        ▼
Amazon RDS MySQL
```

The deployment process involved solving multiple real-world infrastructure challenges, including corrupted file transfers, missing application assets, database compatibility issues, networking configuration, Linux permissions, and Git-based project management.

This project serves as a practical demonstration of cloud deployment, Linux administration, AWS infrastructure, database management, and DevOps troubleshooting skills.

---

## Author

**Olayenikan Michael**

Full-Stack Software Engineer | Cloud & DevOps Engineer

GitHub: [poundsmichaelscode on GitHub](https://github.com/poundsmichaelscode?utm_source=chatgpt.com)

LinkedIn: [Olayenikan Michael on LinkedIn](https://linkedin.com/in/olayenikan-michael?utm_source=chatgpt.com)

---

## License

This repository contains deployment and configuration work for a PrestaShop-based project.

PrestaShop itself is distributed under its applicable open-source license. Refer to the official PrestaShop project documentation and license files included with the application for licensing information.

