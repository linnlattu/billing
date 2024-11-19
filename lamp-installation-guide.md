# LAMP Stack Installation Guide for Ubuntu 22.04

## Preliminary Steps

# Update system
```bash
sudo apt update
sudo apt upgrade -y
```

## 1. Apache Installation


### Install Apache and required extensions
```bash
sudo apt install apache2 apache2-utils -y
```

### Enable required Apache modules
```bash
sudo a2enmod rewrite
sudo a2enmod headers
sudo a2enmod ssl
```

### Start and enable Apache
```bash
sudo systemctl start apache2
sudo systemctl enable apache2
```

## 2. MySQL 8.0 Installation


### Install MySQL
```bash
sudo apt install mysql-server mysql-client -y
```

### Start and enable MySQL
```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

### Secure MySQL installation
```bash
sudo mysql_secure_installation
```

# Configure MySQL

```bash
sudo mysql
```

Run these commands in MySQL prompt:
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_strong_password';
FLUSH PRIVILEGES;
EXIT;
```

## 3. PHP Installation

### Add PHP repository
```bash
sudo add-apt-repository ppa:ondrej/php
sudo add-apt-repository ppa:ondrej/apache2
sudo apt update
```
### Install PHP 8.1
```bash
sudo apt install php8.1 php8.1-fpm -y
```

## 4. Install PHP Extensions

### Install required PHP extensions
```bash
sudo apt install -y php8.1-cli php8.1-common php8.1-curl php8.1-gd php8.1-imap php8.1-mysql php8.1-xml php8.1-bcmath php8.1-fileinfo 
sudo apt install -y php8.1-gmp php8.1-intl php8.1-mbstring php8.1-opcache php8.1-soap php8.1-zip
```

### Configure PDO_MySQL with mysqlnd

#### Ensure PDO_MySQL is using mysqlnd
```bash
sudo apt install php8.1-mysqlnd -y
```

#### Verify mysqlnd is being used
```bash
php -r "print_r(PDO::getAvailableDrivers());"
```

## 5. Install ionCube Loader

### Download ionCube Loader
```bash
cd /tmp
wget https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz
tar xzf ioncube_loaders_lin_x86-64.tar.gz
```
#### 0. First, verify that we have the ioncube loader file
cd /tmp
ls -l ioncube/ioncube_loader_lin_8.1.so

#### 1.Copy loader to PHP extensions directory

```bash
PHP_EXT_DIR=$(php -i | grep extension_dir | cut -d' ' -f3)
sudo cp ioncube/ioncube_loader_lin_8.1.so $PHP_EXT_DIR
```

#### 2.Create configuration file
```bash
sudo bash -c 'echo "zend_extension=ioncube_loader_lin_8.1.so" > /etc/php/8.1/mods-available/ioncube.ini'
```

#### 3. Ensure apache2 PHP module directory exists
```bash
sudo mkdir -p /etc/php/8.1/apache2/conf.d/
```

#### 4. Remove the broken symbolic links
```bash
sudo rm -f /etc/php/8.1/cli/conf.d/00-ioncube.ini
sudo rm -f /etc/php/8.1/apache2/conf.d/00-ioncube.ini
```

#### 5. Create the configuration file again
```bash
sudo bash -c 'echo "zend_extension=/usr/lib/php/20210902/ioncube_loader_lin_8.1.so" > /etc/php/8.1/mods-available/ioncube.ini'
```

#### 6. Create the symbolic links again
```bash
sudo ln -s /etc/php/8.1/mods-available/ioncube.ini /etc/php/8.1/cli/conf.d/00-ioncube.ini
sudo ln -s /etc/php/8.1/mods-available/ioncube.ini /etc/php/8.1/apache2/conf.d/00-ioncube.ini
```

#### 7. Install PHP Apache module if not already installed
```bash
sudo apt install libapache2-mod-php8.1 -y
```

#### 8. Restart Apache
```bash
sudo systemctl restart apache2
```

#### 9. Verify installation
```bash
php -v
```

## Verification Steps

### 1. Check Apache status:
```bash
sudo systemctl status apache2
```

### 2. Check MySQL status:
```bash
sudo systemctl status mysql
```

### 3. Verify PHP installation:
```bash
php -v
```

### 4. Check PHP modules:
```bash
php -m
```

### 5. Verify ionCube installation:
```bash
php -v | grep ionCube
```

### 6. Create a PHP info file:
```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

## Security Recommendations

1. Configure Apache security:
```bash
sudo a2enmod security2
```

2. Update PHP configuration:
```bash
sudo nano /etc/php/8.1/apache2/php.ini
```

Key settings to modify:
```ini
expose_php = Off
max_input_time = 60
memory_limit = 256M
post_max_size = 64M
upload_max_filesize = 64M
```

3. Restart services:
```bash
sudo systemctl restart apache2
sudo systemctl restart mysql
```
