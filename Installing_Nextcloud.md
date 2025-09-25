### Installing dependencies:
` apt install -y sudo nano curl unzip bzip2 cron ffmpeg`

### Install Nginx, MariaDB, PHP
```
apt install -y nginx mariadb-server php-fpm php-mysql \
php-curl php-gd php-xml php-zip php-mbstring php-intl php-bcmath \
php-gmp php-imagick redis-server php-redis
``` 
Enable services:
`systemctl enable --now nginx mariadb redis-server`

### Secure MariaDB
` mysql_secure_installation `

Set root password
Remove anonymous users
Disallow remote root login
Remove test DB

Then create DB + user for Nextcloud:

```
mysql -u root -p

CREATE DATABASE nextcloud;
CREATE USER 'anshul'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'anshul'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Download Nextcloud
```

cd /var/www
curl -o nextcloud-latest.zip https://download.nextcloud.com/server/releases/latest.zip
unzip nextcloud-latest.zip
rm nextcloud-latest.zip
chown -R www-data:www-data nextcloud
```

### Configure PHP (optional)
Edit the file:
`nano /etc/php/8.2/fpm/php.ini` 

Set these (search and adjust):
`memory_limit = 512M`


### Configure Nginx
Create configuration file:
`nano /etc/nginx/sites-available/nextcloud`
Paste:
```
server {
    listen 80;
    server_name your-ip-or-domain;

    root /var/www/nextcloud;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}

```

Enable site:
```
ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx

```
