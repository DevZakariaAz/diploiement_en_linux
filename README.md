# diploiement_en_linux
# **Complete Guide: Deploying a Laravel Project with Nginx on Linux (Step-by-Step)**

This guide walks you through setting up a **Linux server**, installing **Nginx**, configuring **Laravel**, and setting up **MySQL**.

---

## **1. Setting Up a Linux Server**

### **1.1 Setting Up a Virtual Machine on Windows (Using VirtualBox)**

#### **1.1.1 Download Required Software**

- **[VirtualBox](https://www.virtualbox.org/)**
- **[Ubuntu Server 22.04 LTS ISO](https://ubuntu.com/download/server)**

#### **1.1.2 Create and Configure the Virtual Machine**

1. Open VirtualBox → Click **"New"**.
2. **Name** it `Ubuntu Server`, select **Linux (Ubuntu 64-bit)**.
3. Allocate **2GB+ RAM**.
4. Create a **20GB+ Virtual Hard Disk** (Dynamically allocated).
5. Select the Ubuntu ISO and **Start** the VM.

#### **1.1.3 Install Ubuntu Server**

- Select **"Install Ubuntu Server"**.
- Create a username and password.
- Skip optional software for now.
- Reboot when done.

#### **1.1.4 Set Up Network Configuration**

Check your IP address:

```bash
ip a
```

If your IP is dynamic, make it static by editing:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Example:

```yaml
network:
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
          addresses: [8.8.8.8, 8.8.4.4]
  version: 2
```

Save and apply changes:

```bash
sudo netplan apply
```

---

## **2. Installing Required Software**

### **2.1 Update System Packages**

```bash
sudo apt update && sudo apt upgrade -y
```

### **2.2 Install Nginx**

```bash
sudo apt install nginx -y
sudo systemctl enable --now nginx
```

Allow web traffic:

```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

### **2.3 Install MySQL**

```bash
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

Log in to MySQL:

```bash
sudo mysql -u root -p
```

Create a database:

```sql
CREATE DATABASE laravel_db;
CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### **2.4 Install PHP & Extensions**

```bash
sudo apt install php8.1 php8.1-fpm php8.1-mysql php8.1-xml php8.1-bcmath php8.1-mbstring php8.1-curl php8.1-zip unzip -y
```

Check PHP version:

```bash
php -v
```

---

## **3. Setting Up Laravel Project**

### **3.1 Move Project to `/var/www/`**

Upload your Laravel project (via `scp` or `git clone`):

```bash
sudo mkdir -p /var/www/your_project
sudo chown -R $USER:$USER /var/www/your_project
```

Move your Laravel files inside `/var/www/your_project`.

### **3.2 Set Up `.env` File**

```bash
cp .env.example .env
nano .env
```

Edit database connection:

```
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=yourpassword
```

### **3.3 Install Laravel Dependencies**

```bash
composer install
php artisan key:generate
php artisan storage:link
```

### **3.4 Set Permissions**

```bash
sudo chown -R www-data:www-data /var/www/your_project/storage /var/www/your_project/bootstrap/cache
sudo chmod -R 775 /var/www/your_project/storage /var/www/your_project/bootstrap/cache
```

### **3.5 Run Database Migrations**

```bash
php artisan migrate --force
```

---

### **3.6 Install and Configure Vite**
If your project uses Vite:

```bash
npm install
npm run build
```

Make sure your `vite.config.js` file is correct:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/sass/app.scss', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

---

## **4. Configuring Nginx for Laravel**

### **4.1 Create an Nginx Configuration File**

```bash
sudo nano /etc/nginx/sites-available/your_project
```

Paste the following:

```nginx
server {
    listen 80;
    server_name your_domain_or_ip;
    root /var/www/your_project/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

### **4.2 Enable and Test the Configuration**

```bash
sudo ln -s /etc/nginx/sites-available/your_project /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## **5. Monitoring & Troubleshooting**

### **5.1 Check Server Status**

```bash
sudo systemctl status nginx
sudo systemctl status php8.1-fpm
sudo systemctl status mysql
```

### **5.2 Check Logs**

```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/mysql/error.log
```

### **5.3 Restart Services**

```bash
sudo systemctl restart nginx
sudo systemctl restart php8.1-fpm
sudo systemctl restart mysql
```

