# CoreFlow Frontend Deployment Guide

## 1. Clone and Prepare Source
```bash
cd /var/www/domains/coreflow.astraval.com/
sudo rm -rf source
sudo git clone https://github.com/Astraval25/CoreFlowFrontend source
cd source
```
## 2. Install & build (no sudo)
```bash
sudo npm ci
sudo npm run build
```

## 3. Atomic deploy
```bash
sudo rm -rf ../public/*
sudo cp -r dist/* ../public/
```

## 4. Permissions
```bash
sudo chown -R www-data:www-data ../public
sudo find ../public -type d -exec chmod 755 {} \;
sudo find ../public -type f -exec chmod 644 {} \;
```
## 5. Cleanup
```bash
cd .. && rm -rf source
```
