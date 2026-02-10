# Fix VerneMQ defaults to NO MQTT listener when no listener.tcp.* is active
<img width="670" height="162" alt="error1" src="https://github.com/user-attachments/assets/43a47dc3-10df-4fce-94e1-0edb8f771080" />

## Enable MQTT Port 1883 (Simple Fix)

### 1. Edit Main Config
```bash
sudo nano /opt/vernemq/etc/vernemq.conf
```

### 2. **Uncomment/Add these 2 lines** (remove `##`):
```
listener.tcp.default = 0.0.0.0:1883
allow_anonymous = on
```

### 3. Stop, Regenerate Config, Start
```bash
vernemq stop
sudo pkill -f vernemq
cd /opt/vernemq
./bin/vernemq config generate
ulimit -n 65536
./bin/vernemq start
```

### 4. Verify MQTT Port
```bash
sudo netstat -tlnp | grep 1883
# Expected: tcp 0 0 0.0.0.0:1883 0.0.0.0:* LISTEN
```

### 5. Test MQTT
```bash
# Terminal 1:
mosquitto_sub -h localhost -t test/topic -v

# Terminal 2:  
mosquitto_pub -h localhost -t test/topic -m "Hello MQTT v5!"
```

## Ports After Fix
```
1883 TCP → MQTT (all clients)
8888 TCP → Admin API (curl localhost:8888/api/v5/nodes)
4369 TCP → Erlang clustering
```
## Firewall Check
```bash
sudo ufw status
# If active, allow MQTT:
sudo ufw allow 1883/tcp
sudo ufw allow 8888/tcp  # Admin API
```
##  Enable Global Admin API
**Confirmed!** Admin API is bound to **`127.0.0.1:8888`** (localhost only), which is why `curl 103.194.228.52:8888` fails.

## Quick Fix: Enable Global Admin API

### 1. Edit Config
```bash
sudo nano /opt/vernemq/etc/vernemq.conf
```

### 2. **Add this line** (exactly):
```
listener.http.default = 0.0.0.0:8888
# listener.http.default = 127.0.0.1:8888  # command this line by finding ctrl+shift+g
```

### 3. Restart VerneMQ
```bash
./bin/vernemq stop  
sudo pkill -f vernemq
./bin/vernemq config generate
ulimit -n 65536
./bin/vernemq start
```

### 4. Verify Global Binding
```bash
sudo netstat -tlnp | grep 8888
```

### 5. Test Worldwide
```bash
/opt/vernemq/bin/vmq-admin api-key create
curl -u "YOUR_API_KEY:" http://localhost:8888/health
```
Visit in your browser
```bash
http://SERVER_IP:8888/health
```


