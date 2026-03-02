```
cd /var/www/domains
mkdir villpo.astraval.com
cd villpo.astraval.com
```

```
git clone https://github.com/kkbughunter/ecommerce.git source
sudo chown -R $USER:$USER .
sudo chmod -R 755 .
cd source
```

```
cd ecommercebackend
./gradlew clean build -x test
```

```
sudo mkdir -p /var/www/villpo
sudo cp /var/www/domains/villpo.astraval.com/source/ecommercebackend/build/libs/ecommercebackend-0.0.1-SNAPSHOT.jar /var/www/villpo/villpo.jar

sudo chmod 755 /var/www/villpo/villpo.jar
sudo chown astraval:astraval /var/www/villpo/villpo.jar
```

```
sudo mkdir -p /uploads
sudo chown -R $USER:$USER /uploads
sudo chmod -R 755 /uploads
```

```
sudo nano /etc/systemd/system/villpo.service
```
```
[Unit]
Description=Villpo Spring Boot App
After=network.target

[Service]
User=astraval
ExecStart=/usr/bin/java -jar /var/www/villpo/villpo.jar --spring.profiles.active=prod
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
EnvironmentFile=/var/www/villpo/.env

[Install]
WantedBy=multi-user.target
```
```
nano /var/www/villpo/.env
```
```
SERVER_PORT=8090

DB_URL=jdbc:postgresql://localhost:5432/ecommerce
DB_USERNAME=postgres
DB_PASSWORD=password

JPA_DDL_AUTO=update
JPA_SHOW_SQL=true
SQL_INIT_MODE=always

MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=karthikeyantesting1@gmail.com
MAIL_PASSWORD=qanz rbuv azyb irqr
MAIL_SMTP_AUTH=true
MAIL_SMTP_STARTTLS_ENABLE=true

UPLOAD_PATH=uploads
FRONTEND_URL=http://localhost:3000

JWT_SECRET=T4@kZ9!mR2&xW7$Lp8^Qs6&Hv1%An3Y
JWT_EXPIRATION=3600
JWT_REFRESH_EXPIRATION=86400

AUTH_OTP_EXPIRATION_MINUTES=10


SPRINGDOC_SWAGGER_UI_ENABLED=true
```
```
sudo chown astraval:astraval /var/www/villpo/.env
sudo chmod 600 /var/www/villpo/.env
```
```
sudo -u postgres psql
CREATE DATABASE "ecommerce";
GRANT ALL PRIVILEGES ON DATABASE "ecommerce" TO postgres;
\q
```

```
sudo systemctl daemon-reload
sudo systemctl start villpo
sudo systemctl enable villpo
sudo systemctl status villpo
sudo systemctl restart villpo
```

```
sudo journalctl -u villpo -f
```

## Deploy Frontend 

```
sudo nano /etc/apache2/sites-available/villpo.astraval.com.conf
```
```
<VirtualHost *:80>
    ServerName villpo.astraval.com

    ProxyPreserveHost On
    ProxyPass "/" "http://127.0.0.1:8090/"
    ProxyPassReverse "/" "http://127.0.0.1:8090/"

    ErrorLog ${APACHE_LOG_DIR}/villpo-error.log
    CustomLog ${APACHE_LOG_DIR}/villpo-access.log combined
</VirtualHost>
```
```
sudo a2ensite villpo.astraval.com.conf
sudo systemctl reload apache2
```
```
sudo mkdir -p /var/www/villpo/uploads
sudo chown -R www-data:www-data /var/www/villpo/uploads
sudo chmod -R 775 /var/www/villpo/uploads
sudo chmod -R 777 /var/www/villpo/uploads
```
