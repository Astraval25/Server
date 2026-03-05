```
sudo su
cd /var/www/domains/villpo.astraval.com/source
git pull
cd ecommercebackend
./gradlew clean build -x test
sudo cp /var/www/domains/villpo.astraval.com/source/ecommercebackend/build/libs/ecommercebackend-0.0.1-SNAPSHOT.jar /var/www/villpo/villpo.jar

sudo chmod 755 /var/www/villpo/villpo.jar
sudo chown astraval:astraval /var/www/villpo/villpo.jar

```

```
sudo systemctl restart villpo
sudo journalctl -u villpo -f

```


# Frontend 

```
sudo su
cd /var/www/domains/villpo.astraval.com/source
git pull
cd ecommercefrontend/
```
```
npm ci
npm install 
npm run build
cp -r dist/* /var/www/villpo/frontend/
```

```
sudo chown -R www-data:www-data /var/www/villpo/frontend
sudo chmod -R 755 /var/www/villpo/frontend

```
