# PrestaShop 9.1.4 on AWS

[![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20RDS-232F3E?logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![PrestaShop](https://img.shields.io/badge/PrestaShop-9.1.4-DF0067?logo=prestashop&logoColor=white)](https://www.prestashop-project.org/)
[![PHP](https://img.shields.io/badge/PHP-8.x-777BB4?logo=php&logoColor=white)](https://www.php.net/)
[![MySQL](https://img.shields.io/badge/MySQL-8.4-4479A1?logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-Server-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![License](https://img.shields.io/badge/License-Educational-blue)](#license)

A production-style deployment of **PrestaShop 9.1.4** on **Amazon Web Services** using a two-tier architecture:

- **Amazon EC2** for the application layer
- **Amazon RDS for MySQL** for the database layer
- **Apache 2** and **PHP** for the web runtime
- **AWS VPC and Security Groups** for controlled network access

This project demonstrates practical cloud infrastructure provisioning, Linux server administration, secure application-to-database connectivity, PHP application deployment, troubleshooting, and deployment documentation.

---

## Live Deployment

**Storefront**  
<http://ec2-32-197-239-183.compute-1.amazonaws.com/>

**Repository**  
<https://github.com/poundsmichaelscode/prestashop-aws-deployment>

> The application is hosted on an EC2 public DNS address. The URL may change if the instance is stopped and restarted without an Elastic IP.

---

## Project Objectives

The deployment was designed to meet the following requirements:

- Host PrestaShop on a publicly accessible AWS server.
- Keep the application and database on separate infrastructure.
- Use Amazon RDS instead of hosting MySQL on the EC2 application server.
- Restrict database access to the EC2 application server.
- Configure Apache, PHP, Linux permissions, and PrestaShop correctly.
- Document implementation decisions and troubleshooting steps.
- Use AWS resources suitable for a Free Tier or low-cost assessment environment.

---

## Architecture

```text
                         Internet
                            │
                            │ HTTP :80
                            ▼
                 ┌─────────────────────┐
                 │   Amazon EC2        │
                 │ prestashop-app-     │
                 │ server              │
                 │                     │
                 │ Ubuntu Linux        │
                 │ Apache 2            │
                 │ PHP                 │
                 │ PrestaShop 9.1.4    │
                 └──────────┬──────────┘
                            │
                            │ MySQL :3306
                            ▼
                 ┌─────────────────────┐
                 │   Amazon RDS        │
                 │ MySQL 8.4           │
                 │ prestashop-db       │
                 │                     │
                 │ Database:           │
                 │ prestashop          │
                 └─────────────────────┘
```

### Traffic flow

1. A user opens the EC2 public DNS address in a browser.
2. Apache receives the HTTP request on port `80`.
3. Apache and PHP run the PrestaShop application.
4. PrestaShop connects privately to Amazon RDS over MySQL port `3306`.
5. RDS accepts database traffic only from the EC2 application security group.

---

## Technology Stack

| Layer | Technology |
|---|---|
| E-commerce platform | PrestaShop 9.1.4 |
| Cloud provider | Amazon Web Services |
| Compute | Amazon EC2 |
| Database | Amazon RDS for MySQL 8.4 |
| Operating system | Ubuntu Linux |
| Web server | Apache 2 |
| Runtime | PHP 8.x |
| Networking | AWS VPC |
| Access control | AWS Security Groups |
| Remote administration | SSH |
| File transfer | SCP |
| Version control | Git and GitHub |

---

## AWS Infrastructure

### EC2 application server

| Setting | Value |
|---|---|
| Instance name | `prestashop-app-server` |
| Instance type | `t3.micro` |
| Operating system | Ubuntu Server |
| Web server | Apache 2 |
| Application directory | `/var/www/prestashop` |
| Public protocol | HTTP |
| Public port | `80` |
| Administrative protocol | SSH |
| SSH port | `22` |

### Amazon RDS database

| Setting | Value |
|---|---|
| DB identifier | `prestashop-db` |
| Database engine | MySQL Community |
| Engine version | MySQL 8.4 |
| Database name | `prestashop` |
| Database port | `3306` |
| Public accessibility | Disabled |
| Access source | EC2 application security group |

The RDS endpoint and database credentials are intentionally excluded from the repository. They should be provided through secure configuration and never committed to source control.

---

## Security Design

The deployment applies the following controls:

- RDS is not directly accessible from the public internet.
- MySQL port `3306` is restricted to the EC2 application security group.
- SSH access should be restricted to a trusted public IP address.
- HTTP port `80` is publicly accessible for assessment review.
- Database passwords, AWS credentials, private keys, and administrator passwords are excluded from Git.
- The PrestaShop installer directory is removed after installation.
- Apache serves the application from a dedicated VirtualHost.
- Linux file ownership is assigned to the Apache service account.

### Recommended security group rules

#### EC2 web security group

| Type | Protocol | Port | Source |
|---|---|---:|---|
| SSH | TCP | 22 | Trusted IP address only |
| HTTP | TCP | 80 | `0.0.0.0/0` |
| HTTPS | TCP | 443 | `0.0.0.0/0` |

#### RDS database security group

| Type | Protocol | Port | Source |
|---|---|---:|---|
| MySQL/Aurora | TCP | 3306 | EC2 web security group |

---

## Deployment Workflow

### 1. Provision the EC2 instance

- Launch an Ubuntu EC2 instance.
- Select an eligible small instance type such as `t3.micro`.
- Enable a public IPv4 address.
- Attach a security group that allows HTTP and restricted SSH access.
- Create and securely store an SSH key pair.

### 2. Connect to EC2

```bash
chmod 400 prestashop-key.pem

ssh -i prestashop-key.pem \
  ubuntu@EC2_PUBLIC_DNS
```

### 3. Update the server

```bash
sudo apt update
sudo apt upgrade -y
sudo hostnamectl set-hostname prestashop-app-server
```

### 4. Install Apache, PHP, and required extensions

```bash
sudo apt install -y \
  apache2 \
  php \
  libapache2-mod-php \
  php-cli \
  php-common \
  php-curl \
  php-gd \
  php-intl \
  php-mbstring \
  php-mysql \
  php-xml \
  php-zip \
  php-bcmath \
  php-soap \
  php-opcache \
  unzip \
  curl \
  wget \
  mysql-client
```

Enable and start Apache:

```bash
sudo systemctl enable apache2
sudo systemctl restart apache2
sudo systemctl status apache2
```

### 5. Enable required Apache modules

```bash
sudo a2enmod rewrite
sudo a2enmod headers
sudo a2enmod expires
sudo systemctl restart apache2
```

### 6. Configure PHP

The PHP configuration was adjusted to support the PrestaShop installation workload.

Example values:

```ini
memory_limit = 512M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 300
max_input_time = 300
max_input_vars = 10000
date.timezone = Africa/Lagos
```

Restart Apache after editing `php.ini`:

```bash
sudo systemctl restart apache2
```

### 7. Provision Amazon RDS for MySQL

- Create an Amazon RDS MySQL instance.
- Use the same AWS Region and VPC as EC2.
- Disable public accessibility.
- Attach the database security group.
- Permit MySQL traffic only from the EC2 security group.
- Create or initialize the `prestashop` database.

### 8. Test EC2-to-RDS connectivity

Network-level test:

```bash
nc -zv RDS_ENDPOINT 3306
```

Database login test:

```bash
mysql -h RDS_ENDPOINT \
  -P 3306 \
  -u DB_USERNAME \
  -p
```

Create the application database when required:

```sql
CREATE DATABASE prestashop
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

### 9. Upload and validate PrestaShop

Upload the verified application package:

```bash
scp -i prestashop-key.pem \
  prestashop.zip \
  ubuntu@EC2_PUBLIC_DNS:/home/ubuntu/
```

Validate the archive before extraction:

```bash
ls -lh /home/ubuntu/prestashop.zip
file /home/ubuntu/prestashop.zip
unzip -t /home/ubuntu/prestashop.zip
```

Expected final output:

```text
No errors detected in compressed data
```

### 10. Extract the application

```bash
sudo mkdir -p /var/www/prestashop

sudo unzip /home/ubuntu/prestashop.zip \
  -d /var/www/prestashop
```

### 11. Configure ownership and permissions

```bash
sudo chown -R www-data:www-data /var/www/prestashop

sudo find /var/www/prestashop \
  -type d \
  -exec chmod 755 {} \;

sudo find /var/www/prestashop \
  -type f \
  -exec chmod 644 {} \;
```

### 12. Configure the Apache VirtualHost

Example `/etc/apache2/sites-available/prestashop.conf`:

```apache
<VirtualHost *:80>
    ServerAdmin admin@example.com
    ServerName EC2_PUBLIC_DNS

    DocumentRoot /var/www/prestashop

    <Directory /var/www/prestashop>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/prestashop-error.log
    CustomLog ${APACHE_LOG_DIR}/prestashop-access.log combined
</VirtualHost>
```

Enable the site:

```bash
sudo a2dissite 000-default.conf
sudo a2ensite prestashop.conf
sudo apache2ctl configtest
sudo systemctl restart apache2
```

Expected validation result:

```text
Syntax OK
```

### 13. Complete the web installer

Open the EC2 public DNS in a browser and configure:

```text
Database server: RDS endpoint
Database name: prestashop
Database login: secure database user
Database password: stored securely
Database port: 3306
```

Do not use `127.0.0.1` or `localhost`, because MySQL runs on Amazon RDS rather than the EC2 instance.

### 14. Remove the installation directory

After installation:

```bash
sudo rm -rf /var/www/prestashop/install
```

For some packages, the directory may be named `install-dev`. Confirm the actual directory name before deleting it.

### 15. Restart and verify

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```

Verify:

- The storefront loads through the public DNS.
- Static assets load correctly.
- The Back Office is accessible through its generated admin path.
- RDS remains private.
- EC2 can connect to MySQL on port `3306`.

---

## Major Challenges and Resolutions

### 1. SSH connection failures

#### Symptoms

```text
Operation timed out
Could not resolve hostname
```

#### Root causes

- A placeholder hostname was used instead of the real EC2 DNS.
- The `.pem` key path was incorrect.
- SSH access was not consistently available from the current public IP.

#### Resolution

- Located the correct private key.
- Applied secure permissions with `chmod 400`.
- Used the real EC2 public DNS.
- Confirmed that port `22` was permitted from the trusted IP address.

```bash
ssh -i prestashop-key.pem \
  ubuntu@ec2-32-197-239-183.compute-1.amazonaws.com
```

---

### 2. Incorrect database engine

#### Problem

The initial database attempt used an Aurora PostgreSQL-compatible configuration. PrestaShop required a MySQL-compatible database for this implementation.

#### Resolution

- Replaced the PostgreSQL-compatible database with Amazon RDS for MySQL.
- Used MySQL port `3306`.
- Created the `prestashop` database.
- Updated the installer to use the RDS MySQL endpoint.

#### Lesson

Application compatibility requirements should be validated before infrastructure provisioning.

---

### 3. EC2-to-RDS connectivity

#### Problem

The application initially attempted to connect to:

```text
127.0.0.1
```

This referenced the EC2 host, not the RDS database.

#### Resolution

- Used the Amazon RDS endpoint as the database host.
- Kept RDS private.
- Allowed MySQL port `3306` from the EC2 security group.
- Tested connectivity from EC2 before continuing installation.

```bash
nc -zv RDS_ENDPOINT 3306
```

---

### 4. Incomplete SCP transfer

#### Error

```text
End-of-central-directory signature not found
```

#### Cause

The transferred ZIP archive was incomplete.

#### Resolution

- Removed the corrupted archive.
- Re-uploaded the complete package.
- Checked file size and type.
- Tested the ZIP integrity before extraction.

```bash
rm -f /home/ubuntu/prestashop.zip
file /home/ubuntu/prestashop.zip
unzip -t /home/ubuntu/prestashop.zip
```

#### Lesson

Deployment artifacts should always be validated before extraction or release.

---

### 5. Incorrect PrestaShop distribution package

#### Error

```text
The template "assets/js/theme.js" is missing.
The template "assets/css/theme.css" is missing.
```

#### Investigation

The first archive contained source or test resources but did not contain the complete production theme structure required by the installer.

#### Resolution

- Downloaded the official PrestaShop Basic Edition package.
- Extracted the outer archive.
- Located and extracted the nested `prestashop.zip`.
- Verified the required theme assets:

```text
themes/classic/assets/js/theme.js
themes/classic/assets/css/theme.css
```

- Replaced the failed deployment with a clean application directory.

#### Lesson

A repository source archive is not always equivalent to a production-ready distribution package.

---

### 6. Linux ownership and permission issues

#### Problem

Apache and PHP required appropriate access to PrestaShop files and writable directories.

#### Resolution

```bash
sudo chown -R www-data:www-data /var/www/prestashop
```

Directories and files were then assigned secure permissions rather than using `777`.

---

### 7. Apache configuration issues

#### Problem

The Apache default page was initially served, and at one point PHP source code was displayed instead of being executed.

#### Resolution

- Installed and enabled the correct Apache PHP module.
- Configured a dedicated VirtualHost.
- Disabled the default site.
- Enabled the PrestaShop site and rewrite module.
- Validated the configuration before restart.

```bash
sudo a2dissite 000-default.conf
sudo a2ensite prestashop.conf
sudo a2enmod rewrite
sudo apache2ctl configtest
sudo systemctl restart apache2
```

---

### 8. Installation directory removal

#### Problem

PrestaShop blocked normal administrative access until the installer directory was removed.

#### Resolution

```bash
sudo rm -rf /var/www/prestashop/install
```

The generated Back Office directory was preserved and kept out of public documentation.

---

## Project Structure

A typical deployed PrestaShop directory contains:

```text
/var/www/prestashop/
├── admin-api/
├── admin-dev/
├── app/
├── bin/
├── classes/
├── config/
├── controllers/
├── docs/
├── download/
├── img/
├── js/
├── mails/
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
├── .gitignore
├── composer.json
├── composer.lock
├── index.php
└── README.md
```

> The exact structure may vary depending on the PrestaShop distribution package and installation state.

---

## Environment Variables and Secrets

Do not commit real secrets.

Example placeholder configuration:

```dotenv
DB_HOST=your-rds-endpoint
DB_PORT=3306
DB_NAME=prestashop
DB_USER=your-database-user
DB_PASSWORD=your-secure-password
```

Recommended `.gitignore` entries:

```gitignore
# Secrets
.env
.env.*
!.env.example

# Private keys
*.pem
*.key
*.ppk

# Operating system files
.DS_Store
Thumbs.db

# Runtime and cache
var/cache/
var/logs/

# Local IDE configuration
.vscode/
.idea/

# Temporary archives and backups
*.zip
*.tar
*.tar.gz
*.bak
*.sql
```

> Do not ignore `vendor/` automatically unless your deployment process installs Composer dependencies on the target server. For a packaged PrestaShop distribution, `vendor/` may be required at runtime.

---

## Operational Commands

### Apache

```bash
sudo systemctl status apache2
sudo systemctl restart apache2
sudo apache2ctl configtest
```

### PHP

```bash
php -v
php -m
```

### Application files

```bash
cd /var/www/prestashop
ls -lah
```

### Theme assets

```bash
ls -lh themes/classic/assets/js/theme.js
ls -lh themes/classic/assets/css/theme.css
```

### Composer autoloader

```bash
ls -lh vendor/autoload.php
```

### Apache logs

```bash
sudo tail -f /var/log/apache2/prestashop-error.log
sudo tail -f /var/log/apache2/prestashop-access.log
```

### RDS connection

```bash
mysql -h RDS_ENDPOINT \
  -P 3306 \
  -u DB_USERNAME \
  -p
```

---

## Deployment Validation Checklist

- [x] EC2 instance provisioned and running
- [x] Ubuntu server updated
- [x] Apache installed and running
- [x] PHP and required extensions installed
- [x] RDS MySQL database provisioned separately
- [x] Database created
- [x] RDS kept private
- [x] MySQL access restricted to EC2 security group
- [x] EC2-to-RDS connectivity verified
- [x] Correct PrestaShop distribution deployed
- [x] ZIP integrity validated
- [x] Apache VirtualHost configured
- [x] File ownership and permissions corrected
- [x] Theme assets verified
- [x] PrestaShop installation completed
- [x] Installer directory removed
- [x] Storefront accessible through EC2 public DNS
- [x] Project documented and committed to GitHub

---

## Deployment Outcome

The final implementation successfully delivered:

- A working PrestaShop 9.1.4 storefront on Amazon EC2
- Apache and PHP configured on Ubuntu Linux
- A separate Amazon RDS MySQL database
- Private EC2-to-RDS connectivity over port `3306`
- Security-group-based access control
- Correct PrestaShop theme assets and dependencies
- Successful recovery from corrupted file transfer and package-selection issues
- A documented, repeatable deployment process

---

## Recommended Production Improvements

This project is production-style but remains an assessment and learning deployment. For a real production workload, add:

1. **Elastic IP or domain name** to avoid DNS changes.
2. **HTTPS with TLS** using Let's Encrypt or AWS Certificate Manager with a load balancer.
3. **HTTP-to-HTTPS redirection**.
4. **Private subnets for RDS** with properly designed route tables.
5. **Automated RDS backups and point-in-time recovery**.
6. **EC2 backups using AMIs or AWS Backup**.
7. **CloudWatch metrics, logs, and alarms**.
8. **AWS Systems Manager Session Manager** to reduce direct SSH exposure.
9. **AWS Secrets Manager or Parameter Store** for credentials.
10. **Web Application Firewall and load balancing** for internet-facing production use.
11. **Auto Scaling** for higher availability.
12. **CI/CD automation** for repeatable deployments.
13. **Infrastructure as Code** using Terraform, AWS CloudFormation, or AWS CDK.
14. **Separate development, staging, and production environments**.

---

## Skills Demonstrated

- AWS EC2 provisioning
- Amazon RDS deployment
- VPC networking
- Security Group configuration
- Linux server administration
- SSH and SCP troubleshooting
- Apache configuration
- PHP application deployment
- MySQL database administration
- Application-to-database integration
- File ownership and permission management
- Deployment artifact validation
- Log-based troubleshooting
- Secure cloud architecture design
- Git and GitHub documentation

---

## Repository

<https://github.com/poundsmichaelscode/prestashop-aws-deployment>

---

## Author

**Olayenikan Michael Olaniyi**  
Full-Stack Software Engineer | Cloud & DevOps Engineer

- GitHub: <https://github.com/poundsmichaelscode>
- LinkedIn: <https://www.linkedin.com/in/olayenikan-michael/>

---

## License

This repository is provided for educational, portfolio, and technical-assessment purposes. PrestaShop remains subject to its own open-source license and project terms.
