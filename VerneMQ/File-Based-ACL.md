## File-Based ACL

### 1. **Add Multiple Users** (already working)
```bash
./vmq-passwd -c /opt/vernemq/etc/vmq.passwd user1
./vmq-passwd /opt/vernemq/etc/vmq.passwd user2  
./vmq-passwd /opt/vernemq/etc/vmq.passwd admin
```

### 2. **Assign Specific Topics** (`/opt/vernemq/etc/vmq.acl`)
```
# Client1: Only their sensors
user user1
topic readwrite user1/sensors/#

# Device001: Device telemetry only
user user2  
topic read user2/telemetry/+
topic write user2/status

# Admin: Full system access
user admin
topic readwrite $SYS/#
topic readwrite sensors/+/#
```

### 3. **Save → Wait 10s → Live!** ✅

```
client1 → Can ONLY use: client1/sensors/...
device001 → Can ONLY use: device001/telemetry/+ & device001/status
admin → Full access
```

## Test It
```bash
# Allowed
mosquitto_pub -h localhost -u user1 -P pass \
-t "user1/sensors/temp" -m "ok"

# Denied
mosquitto_pub -h localhost -u user1 -P pass \
-t "user2/telemetry/1" -m "test"
```

## Pattern Matching (Advanced)
```
# All clients get home access
pattern read %u/home/+
pattern write %u/home/%c

# Groups (all devices)
user device*
topic readwrite devices/%u/#
```
## **Wildcard Patterns Reference**
```
+          → Single level: sensors/+/temp
#          → Multi-level:  sensors/client1/#
%u         → Username:    %u/home/door1
%c         → ClientID:    home/%c/alarm
user*      → Username wildcard
```
