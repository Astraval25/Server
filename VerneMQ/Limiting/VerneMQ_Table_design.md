
<h1 align="center"> VerneMQ DB Desing</h1> 

## Table
```sql
DROP VIEW IF EXISTS public.vmq_auth_acl;
DROP TABLE IF EXISTS public.vmq_acl;
DROP TABLE IF EXISTS public.vmq_device;
```

### VMQ Table
```sql
CREATE TABLE public.vmq_device (
    id SERIAL PRIMARY KEY,
    mountpoint varchar(10) DEFAULT '' NOT NULL,
    client_id varchar(128) NOT NULL,
    username varchar(128) NOT NULL,
    "password" varchar(128),
    UNIQUE (mountpoint, client_id, username)
);
```

### ACL Table
```sql
CREATE TABLE public.vmq_acl (
    id SERIAL PRIMARY KEY,
    vmq_device_id INT NOT NULL,
    topic varchar(255) NOT NULL,
    "permission" varchar(10) NOT NULL,
    
    CONSTRAINT vmq_acl_permission_check CHECK (
        permission IN ('publish', 'subscribe', 'readwrite')
    ),
    
    CONSTRAINT vmq_acl_user_fkey 
        FOREIGN KEY (vmq_device_id) 
        REFERENCES public.vmq_device(id)
        ON DELETE CASCADE
);
```
## Index
```sql
CREATE INDEX idx_vmq_device_lookup 
ON vmq_device (mountpoint, client_id, username);

CREATE INDEX idx_vmq_acl_user 
ON vmq_acl (vmq_device_id);
```

## View
```sql
CREATE OR REPLACE VIEW public.vmq_auth_acl AS
SELECT 
    COALESCE(u.mountpoint, '') AS mountpoint,
    COALESCE(u.client_id, '') AS client_id,
    u.username,
    u.password,

    COALESCE((
        SELECT json_agg(json_build_object('pattern', a.topic))
        FROM vmq_acl a
        WHERE a.vmq_device_id = u.id
        AND a.permission IN ('publish', 'readwrite')
    ), '[]'::json) AS publish_acl,

    COALESCE((
        SELECT json_agg(json_build_object('pattern', a.topic))
        FROM vmq_acl a
        WHERE a.vmq_device_id = u.id
        AND a.permission IN ('subscribe', 'readwrite')
    ), '[]'::json) AS subscribe_acl

FROM vmq_device u;
```

### Sample Insert Query 
```sql
-- Insert users  password:1234567890
INSERT INTO vmq_device (mountpoint, client_id, username, password)
VALUES ('', 'device001', 'device001', '$2a$10$rMqG7Rl10ie4pSv1XAwW3uCrYNN1yVArMTTWgWQHcypjBOnDgieRG'); 
INSERT INTO vmq_device (mountpoint, client_id, username, password)
VALUES ('', 'device002', 'device002', '$2a$10$rMqG7Rl10ie4pSv1XAwW3uCrYNN1yVArMTTWgWQHcypjBOnDgieRG');
INSERT INTO vmq_device (mountpoint, client_id, username, password)
VALUES ('', 'app01', 'app01', '$2a$10$rMqG7Rl10ie4pSv1XAwW3uCrYNN1yVArMTTWgWQHcypjBOnDgieRG');

-- ACL using vmq_device_id
INSERT INTO public.vmq_acl (vmq_device_id, topic, permission)
SELECT id, 'd2', 'publish'
FROM vmq_device WHERE client_id='device001';

INSERT INTO public.vmq_acl (vmq_device_id, topic, permission)
SELECT id, 'd1', 'subscribe'
FROM vmq_device WHERE client_id='device001';

INSERT INTO public.vmq_acl (vmq_device_id, topic, permission)
SELECT id, 'd1', 'publish'
FROM vmq_device WHERE client_id='device002';

INSERT INTO public.vmq_acl (vmq_device_id, topic, permission)
SELECT id, 'd2', 'subscribe'
FROM vmq_device WHERE client_id='device002';

INSERT INTO public.vmq_acl (vmq_device_id, topic, permission)
SELECT id, 'd+/+', 'readwrite'
FROM vmq_device WHERE client_id='app01';
```


## Test
### Subscribe
```bash
mosquitto_sub -h astraval.com -p 1883 -i device002 -u device002 -P 1234567890 -t d2
```
```bash
mosquitto_pub -h astraval.com -p 1883 -i device001 -u device001 -P 1234567890 -t d2 -m on
mosquitto_pub -h astraval.com -p 1883 -i device001 -u device001 -P 1234567890 -t d2 -m off
```
