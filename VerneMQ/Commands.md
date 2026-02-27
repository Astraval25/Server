# VerneMQ Commands...
For current user (bash):

- Edit `~/.bashrc` (or `~/.profile` if you use that) and add:
  - `export PATH=/opt/vernemq/bin:$PATH`
- Reload the file:
  - `source ~/.bashrc`
- Now run:
  - `vernemq restart`

For system-wide (all users):

- Create a file `/etc/profile.d/vernemq.sh` with:
  - `export PATH=/opt/vernemq/bin:$PATH`
- Log out and log back in, then:
  - `vernemq restart`

  
## Correct Commands
```bash
vernemq help    # Shows all valid commands
vernemq start   # Start as daemon (background)
vernemq stop    # Stop daemon  
vernemq restart # Restart daemon
vernemq ping    # Check if running
vernemq console # Interactive console
```
## Use Systemd (Recommended)
### Start as daemon (background)
```bash
sudo systemctl start vernemq
```
### Stop daemon  
```bash
sudo systemctl stop vernemq
```
### Restart daemon
```bash
sudo systemctl restart vernemq
```
### Check Status + logs
```bash
sudo systemctl status vernemq
```
### Auto-start on boot
```bash
sudo systemctl enable vernemq
```
