# IoTRoot Frontend Application Deploy
## 1. Clone and Build React App
```bash
cd /var/www/domains/iotroot.astraval.com/
git clone https://github.com/Astraval25/IotRootFrontend.git
cd IotRootFrontend

# Install dependencies and build
npm ci 
npm install
npm run build
```
#### Move build to secure public folder
```
mkdir /var/www/iotroot/public
cp -r dist/* /var/www/iotroot/public
```
## 2. Set secure permissions
```
sudo chown -R www-data:www-data /var/www/iotroot/public/
sudo chmod -R 755 /var/www/iotroot/public/
```

## 3. Create Apache Virtual Host Config
```bash
sudo nano /etc/apache2/sites-available/iotroot.astraval.com.conf
```

**Paste this INDUSTRY STANDARD config:**
```apache2
<VirtualHost *:80>
    ServerName iotroot.astraval.com
    ServerAlias www.iotroot.astraval.com

    DocumentRoot /var/www/iotroot/public

    <Directory /var/www/iotroot/public>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted

        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} -f [OR]
        RewriteCond %{REQUEST_FILENAME} -d
        RewriteRule ^ - [L]

        RewriteRule ^ /index.html [L]
    </Directory>

    ErrorLog /var/www/iotroot/logs/error.log
    CustomLog /var/www/iotroot/logs/access.log combined
RewriteCond %{SERVER_NAME} =www.iotroot.astraval.com [OR]
RewriteCond %{SERVER_NAME} =iotroot.astraval.com
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

## 4. Enable Site and Modules
```bash
sudo apache2ctl configtest  # Should say "Syntax OK"
sudo a2ensite iotroot.astraval.com.conf
sudo a2enmod rewrite headers expires deflate
sudo systemctl reload apache2
```

## 6. Set Permissions (Production Secure)
```bash
sudo chown -R www-data:www-data /var/www/iotroot/
sudo chmod -R 755 /var/www/iotroot/
sudo find /var/www/iotroot/public/ -type f -exec chmod 644 {} \;
```

## 7. Test Deployment
```bash
# Local test
curl -I http://localhost

# Apache status
sudo systemctl status apache2

# Check logs
sudo tail -f /var/www/domains/iotroot.astraval.com/logs/error.log
```
