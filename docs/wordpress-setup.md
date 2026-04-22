# WordPress VM Deployment (LEMP Stack)

## Overview
This project sets up a local WordPress environment inside an Ubuntu VirtualBox VM using a LEMP stack:

- Linux (Ubuntu Server)
- Nginx (Web Server)
- MariaDB (Database)
- PHP-FPM (Backend processing)

The purpose of this setup is to simulate a real web server environment and deploy a self-hosted WordPress instance in a controlled lab environment.

---

## 1. Virtual Machine & Network Setup

- Created an Ubuntu Server VM in VirtualBox
- Configured NAT networking with port forwarding:
  - Host Port: 80
  - Guest Port: 80
- Started the VM in headless mode for server operation

---

## 2. System Update & Dependencies

Update system packages:

```bash
sudo apt update

```
Install Required services
```bash
sudo apt install nginx mariadb-server php-fpm php-mysql wget tar -y
```

## 3. Database Setup (MariaDB)
Run secure installation script:
```bash
sudo mysql_secure_installation
```

Configuration steps completed:
- Set root password
- Removed anonymous users
- Disabled remote root login
- Removed test database
- Reloaded privilege tables

Create WordPress Database
```sql
CREATE DATABASE example_db DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;

CREATE USER 'example_user'@'localhost' IDENTIFIED BY 'example_pw';

GRANT ALL PRIVILEGES ON example_db.* TO 'example_user'@'localhost';

FLUSH PRIVILEGES;

EXIT;
```

## 4. WordPress Installation
Navigate to web directory
```bash
cd /var/www/
```
Download WordPress:
```bash
sudo wget https://wordpress.org/latest.tar.gz
```
Extract archive:
```bash
sudo tar -xzvf latest.tar.gz
```
Remove archive:
```bash
sudo rm latest.tar.gz
```

## 5. File Permissions
Set correct permissions for web server access:
```bash
sudo find wordpress/ -type d -exec chmod 755 {} \;
sudo find wordpress/ -type f -exec chmod 644 {} \;
```

Set ownership for Nginx:
```bash
sudo chown -R www-data:www-data wordpress/
```

## 6. Nginx Configuration
Navigate to configuration directory:
```bash
cd /etc/nginx/sites-available/
```
![Nginx Config Test Success](/screenshots/nginx-config-test-success.png)

Create configuration file:
```bash
sudo nano wordpress.conf
```
Nginx Configuration
```Nginx
upstream php-handler {
    server unix:/var/run/php/php8.3-fpm.sock;
}

server {
    listen 80;
    server_name localhost;

    root /var/www/wordpress;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass php-handler;
    }
}
```
Enable Site
Create symbolic link:
```bash
sudo ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/
```
Test configuration:
```bash
sudo nginx -t
```
Restart Nginx:
```bash
sudo systemctl restart nginx
```

## 7. Verification
- Open browser and navigate to: http://localhost
- WordPress setup page should appear
- Complete installation form
- Log into WordPress admin dashboard

## 8. Issues & Fixes
File Permission Issue
If WordPress cannot write to `wp-config.php`:
```bash
sudo chown -R www-data:www-data /var/www/wordpress
```

## Notes
- Search engine indexing was disabled (lab environment)
- Default Nginx page confirmed server before WordPress configuration

## Result
A fully functional local WordPress deployment running on an Ubuntu VM with:
- Nginx web server
- MariaDB database
- PHP processing via PHP-FPM
- Proper file permissions and virtual host configuration
![WordPress Dashboard](/screenshots/wordpress-dashboard.png)
---