
# CoreFlow Backend Deployment Guide

## 1. Clone and Prepare Source

```bash
# Navigate to the project directory
cd /var/www/domains/coreflow.astraval.com/CoreFlowBackend

# Clone the repository into 'source' folder
sudo git clone https://github.com/Astraval25/CoreFlowBackend.git source

# Enter source folder
cd source

# Set ownership and permissions
sudo chown -R $USER:$USER .
sudo chmod -R 755 .
```

## 2. Build the Application

```bash
# Clean and build the project (skip tests)
./gradlew clean build -x test
```

## 3. Deploy JAR

```bash
# Create deployment directory
sudo mkdir -p /var/www/coreflow

# Copy JAR to deployment directory
sudo cp /var/www/domains/coreflow.astraval.com/source/build/libs/coreflow-0.0.1-SNAPSHOT.jar /var/www/coreflow/coreflow.jar

# Set permissions
sudo chmod 755 /var/www/coreflow/coreflow.jar
sudo chown astraval:astraval /var/www/coreflow/coreflow.jar
```

## 4. Prepare Uploads Directory

```bash
sudo mkdir -p /uploads
sudo chown -R $USER:$USER /uploads
sudo chmod -R 755 /uploads
```

## 5. Configure Systemd Service

```bash
sudo nano /etc/systemd/system/coreflow.service
```

**coreflow.service content:**

```ini
[Unit]
Description=CoreFlow Spring Boot App
After=network.target

[Service]
User=astraval
ExecStart=/usr/bin/java -jar /var/www/coreflow/coreflow.jar --spring.profiles.active=prod
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```bash
# Reload systemd and start the service
sudo systemctl daemon-reload
sudo systemctl start coreflow
sudo systemctl enable coreflow
sudo systemctl status coreflow
```

## 6. Configure PostgreSQL Database

```bash
sudo -u postgres psql
```

```sql
CREATE DATABASE "CoreFlow_prod";
CREATE USER postgres WITH PASSWORD 'root@IET25';
GRANT ALL PRIVILEGES ON DATABASE "CoreFlow_prod" TO postgres;
\q
```

```bash
# Restart the application after DB setup
sudo systemctl restart coreflow
sudo systemctl status coreflow
sudo journalctl -u coreflow -f
```

## 7. Configure Apache Reverse Proxy

```bash
sudo nano /etc/apache2/sites-available/coreflow.astraval.com.conf
```

**coreflow.astraval.com.conf content:**

```apache
<VirtualHost *:80>
    ServerName coreflow.astraval.com

    ProxyPreserveHost On
    ProxyPass "/" "http://127.0.0.1:8085/"
    ProxyPassReverse "/" "http://127.0.0.1:8085/"

    ErrorLog ${APACHE_LOG_DIR}/coreflow-error.log
    CustomLog ${APACHE_LOG_DIR}/coreflow-access.log combined
</VirtualHost>
```

```bash
# Enable site and reload Apache
sudo a2ensite coreflow.astraval.com.conf
sudo systemctl reload apache2
```
## 8. File saving permision 
```bash
# create folder if not exists
sudo mkdir -p /var/www/coreflow/uploads

# give ownership to web app user (example: www-data)
sudo chown -R www-data:www-data /var/www/coreflow/uploads

# give read/write/execute permission
sudo chmod -R 775 /var/www/coreflow/uploads

sudo chmod -R 777 /var/www/coreflow/uploads
```
---
