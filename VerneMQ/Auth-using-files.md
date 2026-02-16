## Auth using files

## Enable Password Auth
```bash
sudo su
```
1. Edit `/opt/vernemq/etc/vernemq.conf`:
 ```
 allow_anonymous = off
 plugins.vmq_passwd = on
 vmq_passwd.password_file = /opt/vernemq/etc/vmq.passwd
 ```
verify
```bash
vmq-admin plugin show
```
[docs.vernemq](https://docs.vernemq.com/configuring-vernemq/plugins)

2. Create user:
 ```bash
 cd /opt/vernemq/bin
 ./vmq-passwd -c /opt/vernemq/etc/vmq.passwd client1
 ```

3. Restart VerneMQ (if running):
 ```bash
 /opt/vernemq/bin/vernemq restart
 ```

