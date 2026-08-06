# AWS WordPress Migration Project

## Overview

This project demonstrates migrating a company's legacy application setup to AWS by provisioning a cloud server and deploying a full LAMP stack (Linux, Apache, MySQL, PHP) with WordPress as the site platform. The goal was to modernize an outdated system into something scalable, secure, and easier to manage.

## Stack

- **Cloud Provider:** AWS EC2
- **OS:** Ubuntu (latest LTS)
- **Web Server:** Apache
- **Database:** MySQL
- **Language:** PHP
- **CMS:** WordPress

## Steps Taken

### 1. Launch EC2 Instance
- Created an EC2 instance named `wordpress-migration`
- AMI: Ubuntu
- Instance type: t2.micro (free tier)
- Created a new key pair: `wordpress.pem`
- Security group configured to allow:
  - SSH (port 22)
  - HTTP (port 80)

### 2. Connect via SSH
```bash
ssh -i "wordpress.pem" ubuntu@<ec2-public-dns>
```

### 3. Update the System
```bash
sudo apt update && sudo apt upgrade -y
```

### 4. Install Apache
```bash
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```

### 5. Install MySQL
```bash
sudo apt install mysql-server -y
sudo systemctl start mysql
sudo systemctl enable mysql
```

### 6. Create the WordPress Database and User
Logged into MySQL:
```bash
sudo mysql
```

Ran the following inside the MySQL shell:
```sql
CREATE DATABASE wordpress_db;
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'your_password_here';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 7. Install PHP and Required Extensions
```bash
sudo apt install php libapache2-mod-php php-mysql -y
sudo systemctl restart apache2
```

### 8. Download and Configure WordPress
```bash
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/
```

Copied the sample config and edited it:
```bash
cd /var/www/html
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

Updated the database connection settings (see `wp-config.php` in this repo for the sanitized version):
```php
define( 'DB_NAME', 'wordpress_db' );
define( 'DB_USER', 'wordpress_user' );
define( 'DB_PASSWORD', 'your_password_here' );
define( 'DB_HOST', 'localhost' );
```

### 9. Set Permissions
```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

### 10. Access WordPress
Opened the EC2 instance's public IP in a browser to complete the WordPress installation wizard:
```
http://<ec2-public-ip>
```

Completed the five-minute WordPress install (site title, admin username, password, email), confirming the full stack was working end to end.

## Troubleshooting Notes

Ran into an "Error establishing a database connection" after the initial config. Root cause was the `DB_PASSWORD` field in `wp-config.php` still containing the placeholder value (`password_here`) instead of the actual MySQL user password. Fixed by:
1. Confirming MySQL was running (`sudo systemctl status mysql`)
2. Confirming the database and user existed (`SHOW DATABASES;`, `SELECT user, host FROM mysql.user;`)
3. Confirming user privileges (`SHOW GRANTS FOR 'wordpress_user'@'localhost';`)
4. Resetting the MySQL user password to match what was in `wp-config.php`, then restarting Apache

## Final Result

WordPress is fully operational on an AWS EC2 instance, running on Apache with a MySQL backend and PHP handling the application logic. The site is manageable through the standard WordPress admin dashboard, replacing what would have been an outdated, harder-to-maintain legacy system with a modern, cloud-hosted platform.

## Files in This Repo

- `wp-config.php` — sanitized WordPress configuration file (credentials replaced with placeholders)
- Screenshots documenting each stage of the setup, from EC2 launch through the final WordPress install