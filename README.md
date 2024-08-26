## Pterodactyl Install Guide
This guide showcases an easy way to setup and install Pterodactyl.
<br>
We cover both manual installs and automatic installs of Pterodactyl.


## **Automatic Install Script (by [@TrivCodez](https://github.com/TrivCodez))**
```
#!/bin/bash

# Step 1: Update System
apt update && apt upgrade && apt install sudo && apt install curl && apt install wget

# Step 2: Install Required Packages
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install -y php8.0 php8.0-fpm php8.0-cli php8.0-mysql php8.0-xml php8.0-mbstring php8.0-curl php8.0-zip php8.0-bcmath php8.0-json git unzip curl

# Step 3: Install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Step 4: Install Database Server
sudo apt install -y mysql-server
sudo mysql_secure_installation

# Step 5: Create a Database for Pterodactyl
mysql -u root -p <<EOF
CREATE DATABASE pterodactyl;
CREATE USER 'pterodactyl'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON pterodactyl.* TO 'pterodactyl'@'localhost';
FLUSH PRIVILEGES;
EOF

# Step 6: Download Pterodactyl Panel
cd /var/www/
git clone https://github.com/pterodactyl/panel.git pterodactyl
cd pterodactyl

# Step 7: Install Dependencies
composer install --no-dev --optimize-autoloader

# Step 8: Configure Environment File
cp .env.example .env
nano .env
# Set database credentials and other settings
# DB_CONNECTION=mysql
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=pterodactyl
# DB_USERNAME=pterodactyl
# DB_PASSWORD=your_password

# Step 9: Generate Application Key
php artisan key:generate --force

# Step 10: Run Migrations
php artisan migrate --seed --force

# Step 11: Set Up Web Server (Nginx Example)
sudo nano /etc/nginx/sites-available/pterodactyl
# Add the following configuration:
server {
    listen 80;
    server_name your_domain.com; # Change to your domain
    root /var/www/pterodactyl/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock; # Adjust PHP version as necessary
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
sudo ln -s /etc/nginx/sites-available/pterodactyl /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Step 12: Set Permissions
sudo chown -R www-data:www-data /var/www/pterodactyl/*
sudo chmod -R 755 /var/www/pterodactyl/storage /var/www/pterodactyl/bootstrap/cache

# Step 13: Access the Panel
echo "Pterodactyl Panel installed and configured. Access it at http://your_domain.com"
```

## **Manual Install**

**Step 1: Update your system:**

```
apt update && apt upgrade && apt install curl wget
```

**Step 2: Install required packages:**

```
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install -y php8.0 php8.0-fpm php8.0-cli php8.0-mysql php8.0-xml php8.0-mbstring php8.0-curl php8.0-zip php8.0-bcmath php8.0-json git unzip
```

**Step 3: Install Composer:**

```
curl -sS [https://getcomposer.org/installer](https://getcomposer.org/installer) | php
sudo mv composer.phar /usr/local/bin/composer
```

**Step 4: Install Database Server**

 - **For MySQL:**

```
sudo apt install -y mysql-server
sudo mysql_secure_installation
```

 - **For MariaDB:**

```
sudo apt install -y mariadb-server
sudo mysql_secure_installation
```

**Step 5: Create a database for Pterodactyl:**

 - **Log into MySQL/MariaDB:**

```
mysql -u root -p
```

 - **Run the following commands:**

```
CREATE DATABASE pterodactyl;
CREATE USER 'pterodactyl'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON pterodactyl.* TO 'pterodactyl'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Step 6: Download the Pterodactyl Panel:**

```
cd /var/www/
git clone [https://github.com/pterodactyl/panel.git](https://github.com/pterodactyl/panel.git) pterodactyl
cd pterodactyl
```

**Step 7: Install dependencies:**

```
composer install --no-dev --optimize-autoloader
```

**Step 8: Configure Environment File:**

  - **Copy our example environment file:**

```
cp .env.example .env
```

  - **Edit the .env file to contain your database credentials and other settings:**

```
nano .env
```

- **Make sure to set the following:**

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=pterodactyl
DB_USERNAME=pterodactyl
DB_PASSWORD=your_password
```

**Step 9: Generate an Application Key**

```
php artisan key:generate --force
```

**Step 10: Run Migrations:**

```
php artisan migrate --seed --force
```

**Step 11: Set up web server (Nginx):**

 - **Create a new configuration file for Nginx:**

```
sudo nano /etc/nginx/sites-available/pterodactyl
```

 - **Add the following configuration:**

```
server {
    listen 80;
    server_name your_domain.com; # Replace with your actual domain name

    root /var/www/pterodactyl/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock; # Adjust PHP version as necessary
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

 - **Enable the site and test the configuration:**

```
sudo ln -s /etc/nginx/sites-available/pterodactyl /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

**Step 12: Set appropriate permissions:**

```
sudo chown -R www-data:www-data /var/www/pterodactyl/*
sudo chmod -R 755 /var/www/pterodactyl/storage /var/www/pterodactyl/bootstrap/cache
```
