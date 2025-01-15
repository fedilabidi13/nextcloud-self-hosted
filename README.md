# nextcloud-self-hosted
## Requirements
* Operating system: Ubuntu 22.04 LTS (recommended)
* Databse: MySQL 8.0 / 8.4 or MariaDB 10.6/ 10.11 (recommended) / 11.4
* Web Server: nginx
* Php runtime: 8.3 (recommended)

### Step 1: Install Required Dependencies

Install Nginx, MariaDB, PHP, and other dependencies:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx mariadb-server \
php8.3 php8.3-fpm php8.3-mysql php8.3-common php8.3-xml \
php8.3-mbstring php8.3-curl php8.3-zip php8.3-gd \
php8.3-intl php8.3-bz2 php8.3-imagick unzip wget
```
### Step 2: Secure MariaDB

Run the security script:

```bash
sudo mysql_secure_installation
```
* Set a root password.
* Answer "Y" to all prompts to improve security.

### Step 3: Create a Database for Nextcloud

Log in to MariaDB:

```bash
sudo mysql -u root -p
```
Run the following commands:
```sql
CREATE DATABASE nextcloud;
CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
* change password according to your preferences

### Step 4: Download and Configure Nextcloud
1- Download Nextcloud:
```bash
wget https://download.nextcloud.com/server/releases/latest.zip
```
2- Extract the files:

```bash
unzip latest.zip
sudo mv nextcloud /var/www/
sudo chown -R www-data:www-data /var/www/nextcloud
sudo chmod -R 755 /var/www/nextcloud
```
### Step 5: Configure PHP for Nextcloud
Edit the PHP configuration file:
```bash
sudo nano /etc/php/8.3/fpm/php.ini
```
Adjust the following settings:

```ini
memory_limit = 512M
upload_max_filesize = 100M
post_max_size = 100M
max_execution_time = 300
```

Restart PHP-FPM:
```bash
sudo systemctl restart php8.3-fpm
```
### Step 6: Configure nginx for Nextcloud
Make sure nginx is enabled:
```bash
sudo systemctl enable nginx
sudo systemctl restart nginx
```
Request an ssl certificate using certbot:
* Domain Configuration: Confirm that the domain (cal.chiralsoftware.com) is pointing to your server's public IP address.
* Port 80 and 443 Open: Make sure ports 80 (HTTP) and 443 (HTTPS) are open in your server's firewall and/or hosting provider.

Install Certbot and Dependencies
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```
Run Certbot to automatically configure SSL for Nginx:

```bash
sudo certbot --nginx -d cal.chiralsoftware.com
```
* Certbot will prompt you to provide an email address and agree to the terms.
* It will automatically modify your Nginx configuration to use SSL.


Ensure the Nextcloud directory is properly set up and owned by the www-data user:

```bash
sudo chown -R www-data:www-data /var/www/nextcloud
sudo chmod -R 755 /var/www/nextcloud
```

uodate chiralsoftware.com's nginx configuration:

```bash
sudo nano /etc/nginx/sites-available/chiralsoftware.com
```

add the following block: 

```nginx
location /nextcloud {



root /var/www;



# PHP handling
location ~ \.php(?:$|/) {
fastcgi_split_path_info ^(.+?\.php)(/.*)$;
try_files $fastcgi_script_name =404;



include fastcgi_params;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
fastcgi_param PATH_INFO $fastcgi_path_info;
fastcgi_param HTTPS on;



fastcgi_pass unix:/var/run/php/php-fpm.sock;



# Nextcloud specific PHP settings (timeouts, etc.)
fastcgi_read_timeout 300s; # adjust based on your needs
fastcgi_send_timeout 300s; # adjust based on your needs



}



# Try files (check if file exists)
try_files $uri $uri/ /nextcloud/index.php$request_uri;
}
```
Enable the newly created nginx configuration: 

```bash
sudo ln -s /etc/nginx/sites-available/chiralsoftware.com /etc/nginx/sites-enabled/
sudo nginx -t
```

Remove the default configuration if it is enabled:

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx

```


### Nextcloud UI installation
Acess nextcloud at ``` https://chiralsoftware.com/nextcloud ``` 

You'll be prompted to create a admin user.
* Choose your desired admin username and password.


You'll be prompted to establish MySQL database connection.
Fill the fields accordingly:
* Database account: ```nextclouduser```
* Database name: ```nextcloud```
* Database password: ```password```
* Database host: ```localhost```
