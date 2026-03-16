
<h1 align="center"> VerneMQ DB Desing</h1> 

## Table
### VMQ User Table
```sql
-- Drop table
DROP TABLE IF EXISTS public.vmq_users;

CREATE TABLE public.vmq_users (
	mountpoint varchar(10) DEFAULT ''::character varying NOT NULL,
	client_id varchar(128) NOT NULL,
	username varchar(128) NOT NULL,
	"password" varchar(128) NULL,
	CONSTRAINT vmq_users_pkey PRIMARY KEY (mountpoint, client_id, username)
);
```

### ACL Table
```sql
-- Drop table
DROP TABLE IF EXISTS public.vmq_acl;

CREATE TABLE public.vmq_acl (
	id serial4 NOT NULL,
	mountpoint varchar(10) DEFAULT ''::character varying NOT NULL,
	client_id varchar(128) NOT NULL,
	username varchar(128) NOT NULL,
	topic varchar(255) NOT NULL,
	"permission" varchar(10) NOT NULL,
	CONSTRAINT vmq_acl_permission_check CHECK (((permission)::text = ANY ((ARRAY['publish'::character varying, 'subscribe'::character varying, 'readwrite'::character varying])::text[]))),
	CONSTRAINT vmq_acl_pkey PRIMARY KEY (id)
);

-- public.vmq_acl foreign keys
ALTER TABLE public.vmq_acl ADD CONSTRAINT vmq_acl_mountpoint_client_id_username_fkey FOREIGN KEY (mountpoint,client_id,username) REFERENCES public.vmq_users(mountpoint,client_id,username);
```
## View
```sql
-- public.vmq_auth_acl source

CREATE OR REPLACE VIEW public.vmq_auth_acl
AS SELECT COALESCE(mountpoint, ''::character varying) AS mountpoint,
    COALESCE(client_id, ''::character varying) AS client_id,
    username,
    password,
    COALESCE(( SELECT json_agg(json_build_object('pattern', vmq_acl.topic)) AS json_agg
           FROM vmq_acl
          WHERE u.mountpoint::text = vmq_acl.mountpoint::text AND u.client_id::text = vmq_acl.client_id::text AND u.username::text = vmq_acl.username::text AND (vmq_acl.permission::text = ANY (ARRAY['publish'::character varying, 'readwrite'::character varying]::text[]))), '[]'::json) AS publish_acl,
    COALESCE(( SELECT json_agg(json_build_object('pattern', vmq_acl.topic)) AS json_agg
           FROM vmq_acl
          WHERE u.mountpoint::text = vmq_acl.mountpoint::text AND u.client_id::text = vmq_acl.client_id::text AND u.username::text = vmq_acl.username::text AND (vmq_acl.permission::text = ANY (ARRAY['subscribe'::character varying, 'readwrite'::character varying]::text[]))), '[]'::json) AS subscribe_acl
   FROM vmq_users u;
```

### Sample Insert Query 
```sql
-- Insert users
INSERT INTO public.vmq_users (mountpoint, client_id, username, password)
VALUES 
('', 'device001', 'device001', 'pass123'),
('', 'device002', 'device002', 'pass123'),
('', 'app001', 'appuser', 'app123');


-- Insert ACL rules

-- device001 can publish sensor data
INSERT INTO public.vmq_acl (mountpoint, client_id, username, topic, permission)
VALUES 
('', 'device001', 'device001', 'devices/device001/data', 'publish');

-- device001 can subscribe commands
INSERT INTO public.vmq_acl (mountpoint, client_id, username, topic, permission)
VALUES 
('', 'device001', 'device001', 'devices/device001/cmd', 'subscribe');

-- device002 publish data
INSERT INTO public.vmq_acl (mountpoint, client_id, username, topic, permission)
VALUES 
('', 'device002', 'device002', 'devices/device002/data', 'publish');

-- device002 subscribe commands
INSERT INTO public.vmq_acl (mountpoint, client_id, username, topic, permission)
VALUES 
('', 'device002', 'device002', 'devices/device002/cmd', 'subscribe');

-- app can read/write device topics
INSERT INTO public.vmq_acl (mountpoint, client_id, username, topic, permission)
VALUES 
('', 'app001', 'appuser', 'devices/+/+', 'readwrite');
```
