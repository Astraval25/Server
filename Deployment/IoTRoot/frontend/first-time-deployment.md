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
mkdir ../../public/
cp -r dist/* ../../public/
```
## 2. Set secure permissions
```
sudo chown -R www-data:www-data /var/www/domains/iotroot.astraval.com/public/
sudo chmod -R 755 /var/www/domains/iotroot.astraval.com/public/
```

## 3. Create Apache Virtual Host Config
```bash
sudo nano /etc/apache2/sites-available/iotroot.astraval.com.conf
```

**Paste this INDUSTRY STANDARD config:**
```apache2
<VirtualHost *:80>
    ServerName iotroot.astraval.com

    DocumentRoot /var/www/domains/iotroot.astraval.com/public

    <Directory /var/www/domains/iotroot.astraval.com/public>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted

        RewriteEngine On
        RewriteBase /

        # If a file or directory exists, serve it directly
        RewriteCond %{REQUEST_FILENAME} -f [OR]
        RewriteCond %{REQUEST_FILENAME} -d
        RewriteRule ^ - [L]

        # Otherwise serve index.html (SPA fallback)
        RewriteRule ^ index.html [L,QSA]
    </Directory>

    # Logs
    ErrorLog /var/www/domains/iotroot.astraval.com/logs/error.log
    CustomLog /var/www/domains/iotroot.astraval.com/logs/access.log combined
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
sudo chown -R www-data:www-data /var/www/domains/iotroot.astraval.com/
sudo chmod -R 755 /var/www/domains/iotroot.astraval.com/
sudo find /var/www/domains/iotroot.astraval.com/public/ -type f -exec chmod 644 {} \;
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
