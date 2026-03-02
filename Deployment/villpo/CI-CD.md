```
cd /var/www/domains/villpo.astraval.com/source
cd ecommercebackend
./gradlew clean build
sudo cp /var/www/domains/villpo.astraval.com/source/ecommercebackend/build/libs/ecommercebackend-0.0.1-SNAPSHOT.jar /var/www/villpo/villpo.jar

sudo chmod 755 /var/www/villpo/villpo.jar
sudo chown astraval:astraval /var/www/villpo/villpo.jar

```

```
sudo systemctl restart villpo
sudo journalctl -u villpo -f

```
