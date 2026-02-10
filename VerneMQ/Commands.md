# VerneMQ Commands...

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
