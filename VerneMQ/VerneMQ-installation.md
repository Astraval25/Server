# VerneMQ Installation Guide

## 1. System Update

```bash
sudo apt update
```

---

## 2. Install wxWidgets Dependency

```bash
wx-config --version
apt install libwxgtk3.2-dev
```

---

## 3. Install Build Dependencies

```bash
apt install pkg-config libwxgtk3.2-dev libgl1-mesa-dev libglu1-mesa-dev build-essential autoconf m4 libncurses-dev libpng-dev libssh-dev unixodbc-dev xsltproc fop libxml2-utils
```

---

## 4. Build & Install Erlang (27.3.3)

```bash
cd ~
./configure   --prefix=$HOME/opt/erlang-27.3.3   --without-javac

make -j$(nproc)
make install
```

---

## 5. Configure Erlang PATH

```bash
export PATH="$HOME/opt/erlang-27.3.3/bin:$PATH"
echo 'export PATH="$HOME/opt/erlang-27.3.3/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 6. Verify Erlang Installation

```bash
erl -version

erl -eval 'erlang:display(erlang:system_info(otp_release)), halt().'
```

---

## 7. Clone VerneMQ Source

```bash
cd ~

git clone https://github.com/vernemq/vernemq.git

cd ~/vernemq
git tag -l | grep 2.
git checkout 2.1.2
```

---

## 8. Build VerneMQ Release

```bash
make rel
```

---

## 9. Increase File Descriptor Limit

```bash
ulimit -n 65536
```

---

## 10. Start VerneMQ (Build Location)

```bash
cd /opt/vernemq/bin

./vernemq start

./vernemq ping
```

**Expected Output**

```
pong
```

---

## 11. Verify Release Files

```bash
ls _build/default/rel/vernemq/bin/
```

---

## 12. Move Release to /opt

```bash
sudo mkdir -p /opt/vernemq
sudo cp -r _build/default/rel/vernemq/* /opt/vernemq/
```

---

## 13. Set Permissions

```bash
sudo chown -R root:root /opt/vernemq
sudo chmod -R 755 /opt/vernemq
```

---

## 14. Create Systemd Service (Optional)

```bash
sudo tee /etc/systemd/system/vernemq.service <<EOF
[Unit]
Description=VerneMQ MQTT Broker
After=network.target

[Service]
Type=forking
ExecStart=/opt/vernemq/bin/vernemq start
ExecStop=/opt/vernemq/bin/vernemq stop
Restart=always
RestartSec=5
TimeoutStartSec=120
TimeoutStopSec=60
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
```

---


## 15. Enable & Reload Systemd

```bash
sudo systemctl daemon-reload
sudo systemctl enable vernemq
```

---

## 16. Update PATH (Optional)

```bash
echo 'export PATH="/opt/vernemq/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 17. Run VerneMQ

### Console Mode

```bash
vernemq console
```
exit command: `q().`
### Daemon Mode

```bash
vernemq daemon
```

### Systemd Service

```bash
sudo systemctl start vernemq
sudo systemctl status vernemq
```

---
**VerneMQ Installation Completed**


