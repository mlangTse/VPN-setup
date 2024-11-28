
# Shadowsocks VPN Setup on AWS EC2

This document details the steps to set up a Shadowsocks VPN server on an AWS EC2 instance, including configuration, testing, and troubleshooting.

---

## 1. Prerequisites

- AWS EC2 instance (Linux-based, e.g., Amazon Linux or Ubuntu)
- Domain or public IP address of the EC2 instance
- Access to the EC2 instance via SSH
- Installed software: Python 3, pip, netstat

---

## 2. Shadowsocks Installation

### Install Python and Shadowsocks
```bash
sudo yum update -y        # Update system packages
sudo yum install python3 -y   # Install Python 3
sudo python3 -m pip install --upgrade pip
sudo python3 -m pip install shadowsocks
```

### Verify Installation
Check if the `ssserver` command is available:
```bash
which ssserver
```

---

## 3. Set Up a Virtual Environment (Optional)
```bash
python3 -m venv ~/shadowsocks_env/venv
source ~/shadowsocks_env/venv/bin/activate
pip install shadowsocks
```

---

## 4. Configure Shadowsocks

### Create Configuration File
Edit the configuration file at `/etc/shadowsocks.json`:
```bash
sudo vim /etc/shadowsocks.json
```

Add the following content:
```json
{
    "server": "0.0.0.0",
    "server_port": 8388,
    "password": "your_password",
    "timeout": 300,
    "method": "aes-256-gcm"
}
```

### Set File Permissions
```bash
sudo chmod 600 /etc/shadowsocks.json
sudo chown root:root /etc/shadowsocks.json
```

---

## 5. Set Up Systemd Service

### Create Service File
Edit `/etc/systemd/system/shadowsocks.service`:
```bash
sudo vim /etc/systemd/system/shadowsocks.service
```

Add the following:
```ini
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStart=/home/ec2-user/shadowsocks_env/venv/bin/ssserver -c /etc/shadowsocks.json
Restart=always
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

### Enable and Start the Service
```bash
sudo systemctl daemon-reload
sudo systemctl start shadowsocks
sudo systemctl enable shadowsocks
```

---

## 6. Test Shadowsocks

### Verify Service Status
```bash
sudo systemctl status shadowsocks
```

### Test Locally
```bash
curl -x socks5h://127.0.0.1:8388 http://ipinfo.io
```

---

## 7. Configure AWS Security Group

### Add Inbound Rules
- **Protocol**: TCP, UDP
- **Port**: 8388
- **Source**: 0.0.0.0/0

---

## 8. Troubleshooting

### Check Shadowsocks Logs
```bash
sudo journalctl -u shadowsocks.service -f
```

### Verify Port Usage
```bash
sudo netstat -tunlp | grep 8388
```

### Kill Conflicting Processes
```bash
sudo kill -9 <process_id>
```

### Modify DNS Settings
Update `/etc/resolv.conf`:
```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```

---

## 9. Connect Clients

### Example Client Configuration (e.g., Shadowrocket)
- **Server**: Public IP or Domain Name of your EC2 instance
- **Port**: 8388
- **Encryption**: aes-256-gcm
- **Password**: your_password

---

## 10. Additional Commands

### Stop Shadowsocks
```bash
sudo systemctl stop shadowsocks
```

### Restart Shadowsocks
```bash
sudo systemctl restart shadowsocks
```

### Test Server Speed
```bash
speedtest-cli
```

---

## 11. Common Errors and Fixes

- **Error: Address already in use**
  - Kill the conflicting process:
    ```bash
    sudo kill -9 <process_id>
    ```

- **Error: Port not reachable**
  - Check AWS Security Group and ensure the port is open.
  - Check system firewall rules.

---

This setup completes your VPN server configuration on AWS EC2 using Shadowsocks. Test thoroughly and enjoy secure, unrestricted access!
